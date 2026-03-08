---
layout: post
title: Teaching an LLM a Language It Has Never Seen
subtitle: No fine-tuning, no training data, no examples on the internet. Just a spec and a prayer.
tags: [llm, legacy-migration, prompt-engineering]
permalink: /blog/teaching-llm-unseen-language
---

The proprietary scripting language in this migration has zero presence on the internet. No Stack Overflow questions. No GitHub repos. No documentation outside the company. Some claim no documentation inside the company either. It's been running production workloads for over two decades, and the only people who know it are the people who wrote it.

When I pointed Claude at the first script, I expected nonsense. What I got was surprisingly coherent nonsense — close enough to be dangerous, wrong enough to be useless.

Here's how to get from "almost right" to "parity test passed" without fine-tuning a single weight.

## The Problem With Zero Training Data

LLMs are pattern matchers. They've seen Python, Java, C, COBOL, even some obscure languages that had a brief moment on GitHub. They generalize syntax patterns remarkably well. Hand them a language they've never seen, and they'll try to map it to the closest thing they know.

That's the trap. The proprietary language *looks* like something the model knows. Some constructs resemble Python. Others look like C. The model confidently translates based on surface similarity. The semantics get thrown out the window.

A keyword that looks like a loop might be a cursor operation. A function call that looks synchronous might trigger an asynchronous database write. The syntax is familiar. The behavior is not.

## Why Fine-Tuning Wasn't the Answer

The obvious solution: fine-tune the model on the proprietary language. Get a few hundred examples, train a LoRA adapter, and let the model learn.

Three problems:

1. **Volume.** You need thousands of correct translations to fine-tune. If you had them, you don't need this pipeline. Quite the circle.
2. **Validation.** Fine-tuned output still needs parity testing. If you are testing anyway, the fine-tuning just shifts where the errors happen, it doesn't eliminate them.
3. **Brittleness.** A fine-tuned model learns patterns from training examples. The scripts being translated vary wildly in complexity and style — 30 years of different developers, different conventions, different eras.

Fine-tuning is the fallback if the spec-driven approach fails. Not the starting point.

## The Spec-Driven Approach

Instead of teaching the model the language, teach it the *rules*.

The translation specification is a document that describes:

* **Construct mapping.** "When you see X in the source, produce Y in Python." Not syntax, but semantics. What does the construct *do*, not what it *looks like*.
* **Implicit behaviors.** The things the source language does silently — automatic commits, implicit type conversions, hidden state mutations. These are the landmines.
* **Library mapping.** The proprietary standard library functions and their Python equivalents. Where no equivalent exists, what to build.
* **Forbidden patterns.** Things the model will try because they're Pythonic but wrong in this context. Explicit "do not" instructions.

The spec doesn't teach the model to *read* the source language. It teaches the model to *translate* it — which is a different skill. You don't need to understand Latin grammar to use a translation table. And you don't need polished Pythonic to start running: rumbling and "hurts my eyes" style is fine for a beginning. Correctness first, elegance later.

## What Works

**Reverse engineering before translation.** Take the hit at the beginning. Before translating a single line, have the model explore the source codebase bottom-up. Let it build the data structures itself. Find common patterns. Convert them into bigger constructs. Save the documentation it produces. Provide the file locations as reference for future translations.

This front-loaded investment pays for itself immediately. The model doesn't just translate — it builds a map of the territory. Every subsequent translation is faster because the map already exists.

**Construct-level mapping beats line-level translation.** The model performs significantly better when the spec describes semantic blocks ("this pattern is a database cursor loop, translate it to this Python pattern") rather than individual syntax elements. LLMs think in patterns, not in tokens.

**Explicit negative examples.** "Do NOT use a Python for-loop here, even though it looks like one" is more effective than "use a while-loop with explicit cursor fetch." The model's instinct is to normalize to familiar patterns. Actively block the wrong ones.

**One script at a time.** Context window management is critical. The spec, the source script, and the target output have to fit. No batch translation, no multi-file context. One script, one translation, one parity test.

## What Doesn't Work

**Asking the model to "learn" the language from examples.** Providing 10 translated scripts as few-shot examples sounds smart. In practice, the model over-indexes on surface patterns from the examples and misses the semantic rules. Worse, it starts hallucinating constructs that appeared in the examples but not in the current script.

**Relying on the model's "understanding."** Opus can have a remarkably intelligent conversation about the source language's design philosophy. It can explain *why* a construct works the way it does. And then it translates it wrong. Understanding is not the same as compliance.

**Natural language specs.** "This function writes to the database" is not specific enough. "This function calls cx\_write with parameters (table, key\_fields, data\_fields), which performs an UPSERT using the key\_fields as WHERE clause" — that's a spec.

## The Iteration Loop

The workflow isn't write-spec-once-and-go. It's:

1. Write initial spec based on language documentation and source analysis.
2. Translate a script.
3. Run parity test.
4. Parity fails.
5. Analyze *why* — which construct was mistranslated?
6. Add the correction to the spec.
7. Translate next script with updated spec.

The spec is a living document. Every failed parity test makes it better. After enough iterations, the spec captures the language's implicit behaviors more completely than any documentation ever did.

Which is ironic: the best documentation for a proprietary language might end up being an LLM translation spec.

## Results

The first script took multiple parity test failures before passing. Each failure added a rule to the spec. By the time the spec was stable enough for the second script, the translation was cleaner.

This is the bet: the spec improves faster than the scripts get harder. If that holds, the pipeline scales. If not, it's fine-tuning time.

Early signs say it holds.

## Takeaway

Nobody navigates capital cities by learning each street name. We use Google Maps.

Same principle. The model doesn't need to learn the language. It needs a good enough map — a spec that tells it where each construct leads and where the dead ends are. The model provides the Python fluency. The spec provides the source language knowledge. Together, they do what neither can do alone.

---

*I migrate decades-old systems to modern stacks — one script, one table, one timestamp at a time. I build translation specs for languages that don't exist on the internet. If your codebase predates Google, we should talk.*
