---
layout: post
title: "Spec Kit's fit and limits in ERP modernization"
subtitle: "Audit discipline, sharp Constitution, deterministic transpiler underneath"
tags: [spec-kit, brownfield, ai-assisted-development, evaluation, erp-modernization]
permalink: /blog/spec-kit-brownfield-applicability
---

Previous posts have analysed how Spec Kit handles a small brownfield database ([Post 1](/blog/spec-kit-brownfield-setup), [Post 2](/blog/spec-kit-brownfield-self-audit), [Post 3](/blog/spec-kit-brownfield-implementation)).
This post is about whether the same pipeline fits legacy modernisation.

For parts, with hard audit discipline, and as a complement to a deterministic transpiler rather than a replacement.

## The Tutorial Step 5 evidence

Spec Kit's official tutorial recommends having a separate Claude Code instance audit the plan before implementation.
The README warns that the pipeline can be over-eager, and that pipeline self-validation is not a quality gate.

The runs across this evaluation produced two opposite data points against the same external CI workflow.

Runs 1 and 2 (basicToDo, curated toy app, Opus 4.6 then Sonnet 4.6, no audit discipline): both reported all internal gates passing at pipeline completion.
The external CI on the merged branches reported: 12 of 20 checks failing on each attempt.
The failures included `JSCPD` (copy-paste detector), `MARKDOWN`, `NATURAL_LANGUAGE`, plus several Python linters.
The pipeline reported passing; the linter layer disagreed.
Session 1 had already documented the 440-vs-55 test discrepancy at the test layer.

Run 3 (pdb_search, brownfield, Opus 4.7 at xhigh effort, with the audit discipline shown in Posts [1](/blog/spec-kit-brownfield-setup), [2](/blog/spec-kit-brownfield-self-audit), [3](/blog/spec-kit-brownfield-implementation)): six commits, 12 of 12 checks in the CI passing on each push.

Three variables changed between runs 1+2 and run 3: model generation, audit discipline, and Constitution sharpness.
The data is correlational, not causal.
But it is positive correlational: the configuration that was supposed to produce CI-clean output did produce it.

Pipeline self-validation is only reliable for quality dimensions where the pipeline has a sharp principle.
With ruff and mypy strict baked into `pyproject.toml` from task 1, the pipeline can self-validate on the lint dimension: this quality dimension has a precise predicate.
When the Constitution says nothing about copy-paste density, JSCPD has no internal counterpart.
The pipeline ships duplicate code blocks while claiming success.
External CI must be set up to catch dimensions the Constitution forgot to address.

## Four variables that decide whether the pipeline delivers

The runs 1+2 narrative was simple: pipeline self-validation is unreliable.
Run 3 complicates that.

Spec Kit output quality is a function of at least four variables that all changed at once between runs 1+2 and run 3.

**Model generation.**
Opus 4.7 displayed self-honesty mechanics that Opus 4.6 and Sonnet 4.6 did not: unchecked-with-reasons checklists, explicit clarification blocks, retroactive correction of its own hallucinations with semantic versioning, autonomous mid-pipeline self-audits at phase boundaries, audit-class amplification, intra-artefact self-correction.
This is probably mostly model difference.

**Codebase ambiguity.**
basicToDo had curated, hence obvious bugs.
pdb_search had historical accidents and unstated decisions.
With ambiguous codebases, guessing becomes more expensive.
Ambiguity cuts both ways: the pipeline asked good clarifying questions and fabricated architectural mandates from Constitution-template gaps in the same run.

**User audit discipline.**
In runs 1+2 I never paused mid-pipeline to question outputs.
In run 3 I challenged the fabricated 1s-per-10 000-papers number, then the NOT NULL drift, and then top-level menu scope.
This led to direct correction, audit-class amplification, and behaviour-vs-mechanism verification.

**Constitution sharpness.**
This is the new variable in run 3.
The tool autonomously caught and retracted its own fabricated indexes, because the Constitution explicitly forbade any new index without Complexity Tracking.
Nullability was not addressed by the Constitution, so its fabricated NOT NULL constraints went uncaught.

I do not have enough data to separate the influence of each variable.
But we can extrapolate to production use:

- The pipeline's self-validation alone is not a quality gate, like the Spec Kit README says.
- The audit pass is most effective when the Constitution is sharp.
- The Constitution itself can be hallucinated from gaps, or omit real conscious decisions.
- User audits compound when they name an error class rather than a single instance.

## How Spec Kit complements a deterministic transpiler

For the migration specifically, Spec Kit is one of three tools, not the tool.

| Tool | Strength | Weakness |
|---|---|---|
| Deterministic transpiler | Mechanical translation, byte-faithful, reproducible, scales to all scripts in the corpus | No architectural judgement; produces line-for-line Python that mirrors legacy structure |
| Spec Kit-style pipeline | Architectural framing, layer-boundary discipline, ADR-style decisions, reverse-engineering docs, cross-artefact consistency on amendments | Fabricates from gaps; needs sharp Constitution; needs author-audit; expensive per script |
| Local LLM review | Cross-check on transpiler output, catches subtle defects, scales to all output | No architectural framing; reactive, not generative |

The transpiler produces Python from the legacy source.
Then, the local LLM review catches defects or obvious drifts in that Python.
Spec Kit-style pipelines may wrap families of transpiled scripts in architectural decisions: repository patterns, layer boundaries, integration test design, migration sequencing.

The cross-artefact consistency property we saw in the [previous post](/blog/spec-kit-brownfield-implementation) (Spec Kit catching that `ConfigReader` would be deleted by T026 and extending the Constitution amendment in anticipation) is the discipline that may make the difference at scale.
With dependencies within the codebase, a Constitution that refers to soon-to-be-deleted classes is a real failure mode.
Hence a pipeline that reasons forward across the artefact set is more useful than one that only reacts to existing brokenness.

## Where Spec Kit fits

There seem to be three categories of work where the cost is justified.

**Documented modules with reachable owners.**
The author-audit move works as long as the original author is reachable for the audit.
Constitution-template-gap hallucinations get caught at the Constitution-generation phase before they propagate downstream.

**Scripts with consistent patterns.**
The highest quality scripts in our migration have a high density of conscious convention.
Spec Kit's pre-spec audit layer and Constitution-extraction work well when there are real patterns to extract.
A cross-script Constitution can capture what a family of scripts has in common.
This is the smallest part of a legacy code base.

**Architecture-decision artefacts around the transpiled code.**
Creating layer boundaries, repository patterns and migration strategy as well as reverse-engineering docs for complex scripts where the transpiled code is unclear.
These are exactly what Spec Kit produces well.

## Where Spec Kit does not fit

There are four categories where the pipeline fails worst.

**Undocumented legacy modules with no reachable owner.**
Once the original authors are gone and cannot be asked, Spec Kit fabricates Constitution mandates from accidental correlations.
The hallucinations propagate into ratified Constitutions that are then trusted as policy.

**Bulk per-script work.**
pdb_search at roughly 2 000 LOC consumed one full session at xhigh effort with disciplined audit.
A single script of similar size would cost similarly.
Over a thousand scripts × one session each is not a realistic budget.
Spec Kit must be used for the hard scripts where architectural decisions are pending, not for batch translation.
A caveat here: part of the work may be offloaded to Sonnet agents.
This was not tested in this setup.

**Performance-critical paths without baseline measurements.**
Spec Kit fabricated the 1s-per-10 000-papers performance budget when it had no measured data.
On legacy paths where performance matters and no baseline exists, the same failure mode would produce fabricated performance contracts.

**Schema migrations against production data.**
The NOT NULL and FK-on-link-table drifts in pdb_search would have broken Revision 001 against any database with data fitting the original looser schema.
It is not safe to tighten schemas beyond what the original DB permits.
Cautious review is required for every schema change.

A qualifier on this last item: a brownfield-aware migration pattern is achievable.
pdb_search Revision 002 (a `bibtext_id` → `bibtex_id` reflective rename produced after the schema-tightening audit) is idempotent, queries the actual database state via `inspect()` rather than assuming, and handles the legacy-vs-canonical FK-name asymmetry by drop-then-recreate.
Uncritical schema generation remains unsafe.
But brownfield-aware generation under explicit principles is acceptable.

### Defensive fabrication as the scaling risk

The worst failure mode for a 5,000-script migration is a hallucination defended through elaboration when challenged.
Run 3 produced one instance of that.
I challenged a `_fetch_current_title` task: the cited helper did not exist on the path the contract referenced.
The tool produced an eleven-line diff defending the original reasoning rather than the one-line fix that would have been the honest correction.
The defence read plausibly enough to pass a quick review.

This is the converse of the `uv` pivot in [Post 3](/blog/spec-kit-brownfield-implementation) (where the tool said "both premises were false" and rewrote without elaboration).
Both behaviours appear in the same model under similar audit pressure.
Diff size is the signal: honest corrections are one-line, defensive fabrications run to ten.

I caught it in pdb_search because I read the diff.
At scale, an eleven-line defensive elaboration buried in a routine task review cannot get the same scrutiny.
Defensive fabrication is the failure mode that scales worst.

### A second category of omission

[Previous posts](/blog/spec-kit-brownfield-setup) covered three Constitution-extraction error categories: hallucination from gap, hallucination from over-tightening, and omission of conscious decisions.
A fourth was introduced in this evaluation.

The original Omission category was *code-pattern omission*, with rollback-on-add-failure as the canonical example: the discipline is visible in commit history and source code, but the tool does not elevate it.
A different shape appeared in the first run: 28 open vulnerability advisories on the basicToDo lockfile.
None of them had been flagged by the pipeline because no Constitution-template slot asks about CVE status.

This is *domain omission*: the discipline lives in `package-lock.json` or `uv.lock`. Code-reading can't see it.
The Constitution-template's slots ask about code-quality, testing, UX and performance.
The template is where the gap sits.

For our migration: relevant.
The migration has its own supply-chain surface: database drivers, pinned packages, build infrastructure.
The Spec Kit constitution-templates ask none of those questions.
If the migration uses Spec Kit-style pipelines, the templates must be domain-extended.

## A pilot before scaling

Before committing to Spec Kit, run a single-script pilot under controlled conditions.
Pick a script you know well.
Run the full pipeline at xhigh effort with the same audit discipline used in run 3 of pdb_search.

Three explicit audit passes:

1. **Constitution-generation audit.**
Check the generated Constitution side-by-side with the script's commit history and any author notes.
Mark each mandate as conscious decision, implicit-but-correct, fabricated from a Constitution-template gap, over-tightened, or omitted-on-this-pass.

2. **Behaviour-vs-mechanism verification.**
Wherever the pipeline cites a method, function, or path as preserving current behaviour, verify the citation.
The cascade-mechanism error is the hardest-to-catch finding from run 3 and is likely to recur.

3. **Brownfield-discipline audit.**
Look for schema-tightening, top-level API expansion under the banner of "preservation," and any other instance where the pipeline's output is stricter than the original's behaviour permits.

Decision rule: if any of the hallucinations include code-misreading, behaviour-vs-mechanism citation errors, omissions of conscious architectural decisions, or defensive elaboration when challenged, Spec Kit is not safe at scale on this script class.
Otherwise, fewer than three hallucinations means Spec Kit may be appropriate.

The pdb_search run produced hallucinations in every category.
That is the baseline expectation for this kind of pilot.

## What this means in practice

A reasonable framing for a brownfield migration team may be:
Use Spec Kit for architectural framing of script families and reverse-engineering documentation.
Then, use a deterministic transpiler for mechanical translation of all scripts.
Finally, use local LLM review for defect-detection across all output.
Only apply manual review for any schema or performance-critical change.
That allocation gives each tool the work it is good at and prevents Spec Kit's failure modes from surfacing at the worst spots.

The audit moves shown across Posts [1](/blog/spec-kit-brownfield-setup), [2](/blog/spec-kit-brownfield-self-audit), and [3](/blog/spec-kit-brownfield-implementation) are the prerequisites.
Without them, the pipeline reverts to the runs 1+2 pattern: gates report passing, external CI shows 12 of 20 failing, cost shows up later, not in the pipeline.

Evaluating where AI tooling fits in a brownfield codebase is itself a discipline.
The cost of getting that wrong scales with the codebase.
Discipline determines whether the evaluation produces useful output.

---

*Post 4 of 4 in a series evaluating Spec Kit on a brownfield codebase. Series notes: pdb_search ([github.com/HubReb/pdb_search](https://github.com/HubReb/pdb_search)) and basicToDo ([github.com/HubReb/basicToDo](https://github.com/HubReb/basicToDo)), Claude Code with Opus 4.7 at xhigh effort, [Spec Kit v0.7.5](https://github.com/github/spec-kit/tree/v0.7.5), eval conducted April–May 2026.*

*One engineer's eval on two personal codebases over five sessions, with one tool that's still evolving.*
