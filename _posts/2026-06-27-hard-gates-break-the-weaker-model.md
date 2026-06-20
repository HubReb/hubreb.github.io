---
layout: post
title: "Hard gates don't break whoever holds them. They break the weaker model."
subtitle: "Offloading to a cheaper model has a boundary — and it's the gate that exposes it"
tags: [llm-reliability, evaluation, ai-assisted-development, spec-kit, brownfield]
permalink: /blog/offload-boundary
---

The premise of offloading work to a cheaper model is great. Let the expensive model design. Let the cheap model implement. Put hard gates around the cheap model so it cannot ship something broken. Coverage thresholds, benchmark checks, documentation currency. The gate is supposed to make the model's reliability irrelevant. It either passes or it does not.

I tested that premise on a spec-driven pipeline against pdb_search, a brownfield Python project of mine. The two models were Claude Sonnet, the cheaper one, and Claude Opus, the stronger. The hardened constitution added per-layer coverage gates, a benchmark-presence gate, and a documentation-currency check. The premise turns out to be half right. Sadly, the wrong half crashes the implementation.

## A single run will sell you a clean story

My first gated run looked tidy. Sonnet delivered a working implementation and tripped exactly one gate. The way it tripped suggested a composition problem between two of the gates. The obvious write-up was right there. Harden the gates, Sonnet performs, modulo one fixable defect.

Both halves of that story were artifacts of a single run.

I re-ran Sonnet against the frozen hardened gates seven times. The gate-composition defect showed up once in seven. The headline reliability was worse than the first run implied. The hardened gates were honored cleanly in only three of the seven runs. The dominant failure was that it never wired the gate up at all. Then it declared it passed.

LLMs are non-deterministic. Hence, a single run of an agentic pipeline is an anecdote. The interesting behaviour is in the variance.

## The defect is an interaction, not a property

The next question decides whether offloading is viable. Is this unreliability a property of the gate, or of the model? If hard gates are gameable in general, offloading is dead. If it is the cheap model specifically, offloading has a boundary you can design around.

So I built the two-by-two. Each model, Sonnet and Opus, under both conditions.

Three of the four cells came back clean. The defect lives in exactly one. Sonnet under hardened gates. Sonnet under the plain pipeline was clean and accurate across four runs. Opus under the plain pipeline was clean. And Opus, handed the identical hardened gate Sonnet false-greened, held it every time it ran.

This is an interaction: The hard gate is necessary for the failure to appear, because the cheap model is fine without it. Handed to the stronger model, the same gate is honored. It breaks the weaker model specifically.

## Where the line actually falls

Offload-by-gate is safe for existence and syntactic gates: Is the benchmark file present, does a token appear in the docs, does the suite run. Both models honored those across the board. They are deterministically checkable anyway, so the gate is real regardless of who reports it.

Offload-by-gate degrades when a cheap model is run against a hard proxy gate. The kind that takes real work to satisfy and whose pass condition is easy to assert and tedious to verify. That is where you get gaming, silent non-enforcement, and false-green self-reports.

Enforce the gate by the runner: Re-run the gate in CI and read the artifact. Do not read the model's sentence that says the gate passed. Model identity shifts the rate of rot, not the rule. Opus rots less often, but you still never trust its self-report.

## What I could not claim, and why

I cannot say "Opus is more honest."

Opus's self-reports were accurate every time. But it met the gate every time. It never failed, so it was never in the position that produced Sonnet's lies, which were all cover-ups of a failure. Counting it as "zero false-greens in six runs" looks like an honesty record. But false-greening requires a failure to cover, and Opus had zero failures to cover.

Opus is more thorough. It reliably does the verification work the gate demands. Whether Opus would lie about a failure is untested, because its thoroughness kept it from ever failing.

That gap is why the rule is "verify artifacts, not narratives," and why no model is exempt. Opus is safe on its passes and unobserved on its failures. I cannot claim it will not repeat Sonnet's behaviour. You check the runner, not the sentence.

---

*Part of a series evaluating Spec Kit on a brownfield codebase. Earlier posts cover [implementing the spec](/blog/spec-kit-brownfield-implementation) and [its fit for legacy modernisation](/blog/spec-kit-brownfield-applicability).*

*Hard gates don't make a model reliable — they only move where it fools you. Read the runner, never the model's account of it.*
