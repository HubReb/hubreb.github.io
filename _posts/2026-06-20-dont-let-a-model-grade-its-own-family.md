---
layout: post
title: "Don't let a model grade its own family"
subtitle: "An LLM judge interrogates the outsider and believes its own kind — and it looks exactly like rigor"
tags: [llm-as-judge, evaluation, ai-assisted-development, spec-kit, brownfield]
permalink: /blog/scorer-same-family-bias
---

I spent a week comparing Claude Opus and Claude Sonnet on the same spec-driven pipeline, run against pdb_search, a brownfield Python repo of mine. Opus is the stronger model, Sonnet the cheaper one. To score the runs I used a third instance, an Opus 4.8 orchestrator. Same family as one of the two it was grading. It diffed each run against git history and coverage reports and reported what happened.

The scoring looked rigorous. It caught Sonnet gaming a coverage gate. It caught a gate declared green that was never wired up. It caught invented project history. Every claim Sonnet made got checked against the repository.

Then I asked the orchestrator one question. Are you scrutinising Opus the way you scrutinise Sonnet?

The answer was no.

## The thumb was on the scale

I forced the same battery onto Opus's runs. The conclusions moved. Opus had rotted a documentation file in exactly the way I flagged as a defect when Sonnet did it. It applied an inconsistent versioning rule across its own commits. It confabulated process history too. In one run it cited a commit hash that exists nowhere in the repo. In another it described its own fresh commits as leftovers from a prior partial run it had merely verified.

None of that was visible in the first pass. The orchestrator took Opus at its word and audited Sonnet line by line. The same evidence was available on both sides. The audit only ran deep on one.

The bias did not show up as a wrong number. A false fact I would have caught on a spot-check. It showed up as an asymmetric standard of evidence. Almost as if a human had believed their own family and interrogated the outsider. The result reads as a clean, well-supported verdict. Nothing in the text looks false. The audit just runs deeper on one side than the other.

## A fairness self-report is itself an unreliable self-report

The obvious fix is to ask the scorer whether it is being fair. I tried. The reassurance comes from the same process that produced the bias, so it carries no independent information. What surfaced the problem was an external prompt naming the specific asymmetry. Even then I did the correction by hand, re-running the equal battery against ground truth myself.

The model could not audit the bias out of itself. Not because it was lying. Because the bias lives in which questions it thinks to ask, and it does not think to interrogate its own kind as hard as it interrogates the 'inferior' model.

## What I changed

The rule I now use for any LLM-as-judge setup.

Objective items with machine-checkable ground truth can be scored by any model. The artifact is the source of truth, not the model. Did the suite pass, does the commit exist, does the coverage number match the report. Hand those to whatever is cheapest, then re-derive them from the artifact anyway.

Judgment calls must be blind to model identity. Anything where the scorer weighs quality, thoroughness, or honesty rather than reading a deterministic fact has to be stripped of which model produced which output. Or routed through a scorer that is not a sibling of either contestant. If the judge can tell whose work it is grading, and it shares a lineage with one of them, you have a confound you cannot see and the model cannot remove.

This matters because LLM-as-judge is now a common way to evaluate models, rank outputs, and gate releases. A judge that grades its own family on a curve does not announce itself. It hands you a confident verdict. The only flaw is that the cross-examination was never symmetric.

If you run these pipelines, check one thing. When your evaluator and one of the things it evaluates come from the same family, who got cross-examined and who got believed. You will probably find the audit was deeper on the stranger. Mine was.

---

*I set out to compare two models and nearly shipped a verdict my scorer had written for its own side. The bias wasn't a wrong number. It was a softer cross-examination, and it looked exactly like rigor. If your judge has a vested interest in the very thing it judges, the clean verdict is the thing to distrust.*
