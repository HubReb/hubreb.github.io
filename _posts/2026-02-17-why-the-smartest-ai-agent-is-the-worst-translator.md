---
layout: post
title: Why the Smartest AI Agent Is the Worst Translator
subtitle: How Claude Opus filled my context window with brilliant thoughts and zero output
tags: [llm, legacy-migration, ai-tooling]
permalink: /blog/smartest-agent-worst-translator
---

Migrating 6,000+ scripts from a proprietary language into Python 3. The language is old enough to vote. No LLM has seen it in training. The scripts run a legacy ERP system, and the only definition of "correct" is: the database looks identical after the translated version runs.

My instinct said: use the best model available. Opus. Extended thinking. Maximum reasoning power. Surely the smartest model handles the unknown best.

It doesn't.

## What Opus Did

Opus understood everything. The source language, the target architecture, the spec I'd written. Genuinely impressive comprehension of a language it had never seen before.

And then it started *thinking*.

It explored edge cases the spec had already resolved. Considered architectural alternatives nobody asked for. Generated commentary about design tradeoffs that were irrelevant to the task. The context window filled up with high-quality reasoning that actively prevented it from finishing a straightforward translation.

"Just follow the spec and translate" didn't help. Opus would read the instruction, acknowledge it, and immediately start a new branch of exploration. Asking a reasoning model to stop reasoning is like asking a retriever not to fetch. It's in the training.

## What Sonnet Did

Same spec. Same source script. Same prompt.

Sonnet read the spec. Translated. Finished. Two hours, parity test passed.

No exploration. No architectural musings. It treated the spec as ground truth and the translation as mechanical work. Which is what it is.

## The Point

Legacy code translation is not a reasoning problem. It's a compliance problem. The source behavior *is* the specification. The target must replicate it exactly. Creativity is a bug.

I spent a day watching Opus think beautifully about a task that needed no thinking. Sonnet delivered in two hours what Opus couldn't finish.

The uncomfortable conclusion:

| Task | Model | Why |
| --- | --- | --- |
| Architecture decisions | Opus | Exploration is the point |
| Script selection, dependency analysis | Opus | Pattern recognition across a large codebase |
| Mechanical translation | Sonnet | Follow the spec, don't think about it |
| Parity validation | Neither | Deterministic comparison, no LLM needed |

80% of a migration pipeline is mechanical translation. The task where the most expensive model is the worst choice.

## The Workflow That Works

1. **Opus** selects the next script batch, identifies dependencies, writes the translation spec. This is where reasoning earns its keep.
2. **Sonnet** translates. No deviation, no exploration. Spec in, Python out.
3. **Parity test** compares database states. Pass or fail, no ambiguity.

Opus never translates. Sonnet never architects. Separation of concerns — applied to AI tooling.

## Why This Matters

If we use AI for legacy migration, we hit this wall. The smarter the model, the harder it is to make it do boring work. And migration is 80% boring work. The interesting 20% — understanding the codebase, identifying semantic traps, designing the target architecture — that's where the reasoning model belongs. For everything else, use the model that follows instructions without getting inspired.

## First Results

* First script: 2 hours, parity confirmed.
* Expected throughput: 30-60 minutes per script as the workflow stabilizes.
* Automation target: 60%+ without manual intervention. Below that, fine-tuning becomes the next step.

Early signs are good. The two-model split holds.

## Takeaway

Match the model to the task, not to the prestige. The hardest part of legacy migration isn't the translation — it's the analysis before it. Use the reasoning budget where reasoning matters. For the rest, use the model that does what it's told.

The smartest agent in the room is not always the one you want writing the code.

Same goes for humans, by the way. The smarter, the harder to get them to do boring work. And the higher the error rate.

---

*I migrate decades-old systems to modern stacks — one script, one table, one timestamp at a time. I build the AI tooling that makes it possible. If that sounds familiar, let's talk.*
