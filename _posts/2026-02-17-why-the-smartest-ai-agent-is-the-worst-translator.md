---
layout: post
title: Why the Smartest AI Agent Is the Worst Translator
subtitle: How a frontier model filled my context window with brilliant thoughts and zero output
tags: [llm, legacy-migration, ai-tooling]
permalink: /blog/smartest-agent-worst-translator
---

Migrating 6,000+ scripts from a proprietary language into Python 3. The language is old enough to vote. No LLM has seen it in training. The scripts run a legacy ERP system, and the only definition of "correct" is: the database looks identical after the translated version runs.

My instinct said: use the best model available. Largest tier, extended thinking, maximum reasoning power. Surely the smartest model handles the unknown best.

It doesn't. And that's not a flaw in the model. It's a mismatch between the tool and the task.

## What the Reasoning Model Did

It understood everything. The source language, the target architecture, the spec I'd written. Impressive comprehension of a language it had never seen before.

And then it started *thinking*.

It explored edge cases the spec had already resolved. Considered architectural alternatives nobody asked for. Generated commentary about design tradeoffs that were irrelevant to the task. The context window filled up with high-quality reasoning that prevented it from finishing the translation.

"Just follow the spec and translate" didn't help. The model would read the instruction, acknowledge it, and immediately start a new branch of exploration. Asking a reasoning model to stop reasoning is like asking a retriever not to fetch.

## What the Instruction-Following Model Did

Same spec, same source script, same prompt.

Read the spec. Translated. Finished. Two hours, parity test passed.

No exploration. No architectural musings. Spec as ground truth, translation as mechanical work.

## The Limitation

Legacy code translation is a compliance problem, not a reasoning problem. The source behavior is the specification, and the target must replicate it exactly. Creativity is a bug.

I spent a day watching the reasoning model think beautifully about a task that needed no thinking. The smaller model delivered in two hours what the larger one couldn't finish.

This isn't about one model being better than the other. Reasoning models are built to explore, and that's what makes them good at architecture, analysis, and planning. It's also what makes them bad at mechanical work where the spec is already written and deviation means failure.

| Task | Model tier | Why |
| --- | --- | --- |
| Architecture decisions | Reasoning (largest) | Exploration is the point |
| Script selection, dependency analysis | Reasoning (largest) | Pattern recognition across a large codebase |
| Mechanical translation | Instruction-following (smaller) | Follow the spec, don't think about it |
| Parity validation | Neither | Deterministic comparison, no LLM needed |

80% of a migration pipeline is mechanical translation. The task where the most capable model is the worst choice.

## The Workflow That Works

1. **Reasoning model** selects the next script batch, identifies dependencies, writes the translation spec.
2. **Instruction-following model** translates. No deviation, no exploration. Spec in, Python out.
3. **Parity test** compares database states. Pass or fail, no ambiguity.

The reasoning model never translates. The instruction-following model never architects.

## First Results

* First script: 2 hours, parity confirmed.
* Expected throughput: 30-60 minutes per script as the workflow stabilizes.
* Automation target: 60%+ without manual intervention. Below that, fine-tuning becomes the next step.

Early signs are good. The two-tier split holds.

The smartest agent in the room is not always the one you want writing the code. Same goes for humans, by the way.

---

*The expensive model thinks. The cheap model ships. Know which one you need.*
