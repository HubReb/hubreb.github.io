---
layout: post
title: "Conservative by Design: Transpiling Thousands of Legacy Scripts Without Guessing"
subtitle: "A transpiler that flags what it can't do instead of guessing. Revolutionary, apparently."
tags: [transpiler, legacy-systems, python, migration]
permalink: /blog/transpiler
---

I need to migrate several thousand scripts from a proprietary language into Python 3. The language is a Python 2.2 dialect with custom builtins bolted on over two decades. No documentation worth the name. No LLM has seen it in training. The scripts run a legacy ERP system, and the only definition of "correct" is: the database looks identical after the translated version runs.

In [part one](/blog/teaching-llm-unseen-language), I described the spec-driven LLM approach. It works. You build an iterative spec, feed it to the model, translate one script at a time, parity-test each output. Quality is good. Around 150 scripts migrated and parity-tested that way. But every script needs a full context window, every translation has token cost and variance. At this scale, the LLM-only path would take the better part of a year.

Those migrations weren't wasted. Every parity failure had improved the spec. After 150+ iterations, the spec had become a 760-line semantic reference — a Rosetta Stone mapping every construct to its Python 3 equivalent. Nine sections, 17 documented edge cases, pattern frequencies across the entire corpus.

That document *is* a transpiler spec. So I built one.

## Why Six Phases

Pre-clean → fissix (Py2→3) → Scaffold → Import resolution → API map → ruff format

The ordering isn't arbitrary. Each phase depends on the output of the previous one being clean.

Pre-clean fixes syntax that would crash fissix. fissix runs on flat source before scaffold wraps everything in `def main()`. Scaffold strips headers before import resolution inserts new imports. Imports run before API mapping so that `rt.STATUS_CODE → STATUS_CODE` rewrites are visible to import detection. ruff runs last because everything before it changes the source.

The alternative was a single-pass AST transform — walk the tree once, apply all rewrites together. Faster, but it can't use fissix. fissix is CST-based, which means it preserves comments. CST and AST can't round-trip losslessly. I'd have to rewrite the ~50 standard Python 2→3 fixers that fissix gives me for free. For a one-person project, that's not a trade-off, that's a trap.

Multi-pass is slower. Each phase is independently testable. I'll take that deal every time.

## String Exceptions

Hundreds of scripts use `raise "Tag"` / `except "Tag":` as structured goto. This is legal Python 2.2. It's a SyntaxError in Python 3.

The scripts use string labels like `"CheckOK"` or `"BadInput"` to jump out of nested logic. Multiple labels can coexist in the same try block. This matters.

My approach: for each paired label in a try block, generate a class. `class _CheckOKError(Exception): pass`. Rewrite `raise "CheckOK"` to `raise _CheckOKError()`. Rewrite `except "CheckOK":` to `except _CheckOKError:`. Two-arg raises like `raise "X", value` store the value in the exception.

The simpler alternative: a single generic exception with the string as message. `raise Exception("CheckOK")` / `except Exception as e: if e.args[0] == "CheckOK"`. Works until two labels coexist in one block. `except Exception` catches both. Now `"ValidateOK"` and `"BadInput"` are indistinguishable. Silent wrong behavior. Back to generated classes.

Unpaired labels and ambiguous cases get `# TODO` comments. They don't get guesses.

## The rt.exec() Problem

`rt.exec()` has two completely different meanings. It either loads a library into the caller's namespace, or it invokes a script as a subprocess. Same function, different semantics, distinguished by how the arguments look.

The heuristic: string literal as first argument + no positional args = library load → convert to import. Anything with positional args or variable names = script invocation → keep as `rt.exec()`.

The alternative: build a complete lookup table classifying every target as library vs. script. More accurate, but that's ~200 ambiguous names to classify manually. The heuristic covers 95%+ without false positives. The ~200 ambiguous names default to `rt.exec()`, which is safe — worst case is a runtime call instead of an import. Not a crash.

## Wildcard Imports

Several libraries inject dozens of names into the caller's namespace: one exports 40+ global constants, another a class hierarchy with helpers, a third a set of status codes. In the original language, `rt.exec("globals_lib")` just dumps everything into your scope. The Python 3 equivalent is `from bridge.libs.globals_lib import *`.

The cleaner alternative: scan each script for which names it actually uses, emit only those. Requires maintaining a complete export manifest in the transpiler. Any addition or rename in the library forces a transpiler update. Wildcard is less readable but survives change. Linters complain. `# noqa` handles that.

The rest export one or two names each. Those get explicit named imports. No blanket rule where it isn't needed.

## The Central Decision

Every choice above follows from one principle: a silently wrong script is worse than a script with a TODO comment.

Assume all integer division should become `//`? Works until it breaks float arithmetic. Wrong string exception mapping? Silently skips an error handler. Wrong `rt.exec()` classification? Import fails at runtime or, worse, succeeds with wrong behavior.

TODOs are annoying. You can grep for them. A silent regression passes `py_compile`, might pass parity tests, and then corrupts production data weeks later. I know which one I prefer.

5% of scripts — nested loops, PL/SQL calls, stateful logic — always go through manual review. No transpiler handles those correctly, and pretending otherwise would be the kind of confidence that breaks systems.

## Where It Stands

The ~150 LLM-migrated scripts from part one still stand. On top of that, the transpiler has converted and parity-tested another 500+. That's the proof of concept. The vast majority of the corpus remains. Same patterns, same pipeline — but blocked on several unmigrated core libraries that most scripts depend on.

95% of scripts fall into mechanical patterns. The pipeline handles those. The remaining 5% get flagged for manual work. The architecture scales. The bottleneck isn't the transpiler — it's the libraries. Once those are migrated, the pipeline eats through the backlog. Until then, it waits.

The pipeline doesn't do anything clever. It does exactly what the spec says, flags what it can't resolve, and never guesses. That's the entire design philosophy.

---

*This is part three of a series on AI-assisted legacy modernization. [Part one](/blog/teaching-llm-unseen-language) covers teaching an LLM a proprietary language. [Part two](/blog/parity-testing) covers parity testing.*
