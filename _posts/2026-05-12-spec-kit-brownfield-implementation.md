---
layout: post
title: "Implementing the spec: where discipline holds and where it slips"
subtitle: "Right behaviour can ride on a wrong cited mechanism. Cross-artefact consistency is checked on apply, not on generation."
tags: [spec-kit, brownfield, ai-assisted-development, evaluation]
permalink: /blog/spec-kit-brownfield-implementation
---

[Post 1](/blog/spec-kit-brownfield-setup) and [Post 2](/blog/spec-kit-brownfield-self-audit) covered the Constitution-generation phase and the audit moves that corrected it.
Once the spec is stable, the same tool starts writing code, with the spec as the contract.

The implementation phase produced four episodes worth recording.
Two are self-correction events on actual code: one I caught by reading the call graph against a contracts file, and one the tool caught against my own working-notes decision to skip the audit.
The third, the `uv` pivot, sits outside the self-correction taxonomy; it is a tool-honesty episode.
The fourth is the `T009` reverse-engineering document: a sample of what the pipeline produces when its inputs are clean.

## Audit Move 4: the cascade-on-delete verification

A contracts file in the design phase asserted that the delete cascade preserved the behaviour of the legacy method `__delete_author_with_no_papers`.
The cited mechanism: that named method.

I pulled the actual code and traced the call graph.
The cascade happens in `delete_author_of_list`: after deleting the rows the user requested, the function issues a second DELETE against authors with no remaining join-table entries.
`__delete_author_with_no_papers` exists in the same module and is reachable from a different code path. But it is not invoked on the path the contract was talking about.
The outcome was correct; the cited mechanism was wrong.

I sent the audit.
The tool corrected the contract file, named `delete_author_of_list` as the actual mechanism, and added a trap-warning paragraph for future reviewers stating that the cascade is inline and that the similarly-named method should not be assumed to be on the path.
The tool extended the correction beyond what I had asked for.

The reason this failure mode matters: a reviewer who trusts the citation will look at `__delete_author_with_no_papers`, verify that the body of that method enforces the cascade behaviour correctly, and sign off.
The reviewer will have verified a method that is not on the code path under review.
The body of `delete_author_of_list` is never read.

I caught this in a personal codebase because I happened to read the call graph instead of the contract.
At ERP scale, that catch requires either machine-traceable contracts or a reviewer-discipline norm.
Obvious wrongness gets caught quickly, but documentation that is correct in outcome and wrong in cited mechanism passes inspection on a quick read.
The trap-warning is one of the few mechanisms that gives a reviewer a chance against this class of finding.

## Audit Move 6: the ConfigReader cross-artefact fix

While applying a Constitution amendment, the tool noticed that the original Stack & Constraints section of the Constitution referenced `ConfigReader`.
That class was scheduled to be deleted two phases later in task `T026`.

The tool extended the amendment to update the Stack & Constraints wording so the deleted class would no longer be referenced.
Then, it explained the extension verbatim:

> *"The Configuration line is updated in lockstep because ConfigReader is deleted in T026; leaving the original wording would make the constitution reference a class that no longer exists."*

The reasoning shape itself is striking.
The Constitution and the task list are separate documents.
`T026` had not run yet.
The tool inferred: "if I apply this amendment now, and `T026` runs later as scheduled, the Constitution will be internally inconsistent at that future point."
That is multi-step reasoning across the artefact set.

What also stands out is my working notes.
They contain an explicit decision to skip this audit.
I had flagged the `ConfigReader` reference as a likely future inconsistency.
But I decided not to spend quota on it during the amendment; I would catch it later when `T026` ran.
Yet, the tool caught it during the apply step before I had reviewed the amendment output.

If I had not been keeping notes, I would not have noticed that the tool did anything I would have skipped.
The fix would have been invisibly absorbed into the amendment diff.

## The fifth self-correction mode: cross-artefact consistency on apply

[Post 2](/blog/spec-kit-brownfield-self-audit) closed with a four-mode taxonomy and forward-referenced two more.
The `ConfigReader` episode is the example of the fifth mode.

| Mode | Trigger |
|---|---|
| User-direct | I point at a specific artefact and say it is wrong |
| Audit-class amplification | I point at one instance, the tool generalises across the class |
| Autonomous-constitution | The tool re-runs the Constitution check at a phase boundary and finds its own violation |
| Intra-artefact | The tool corrects itself mid-sentence within a single artefact during generation |
| **Cross-artefact consistency on apply** | The tool checks that an amendment does not leave the artefact set internally inconsistent against scheduled future tasks |

The first four modes operate locally: within one artefact at a time, whether triggered by user audit, phase-boundary check, or mid-generation revision.
Mode 5 crosses both artefact boundaries and task scheduling.

What makes Mode 5 possible is the artefact structure that the pipeline imposes:
Each artefact has a defined role and defined relationships to the others.
Without that structure there is no Constitution and no scheduled task list, hence no consistency relation to maintain.
Mode 5 can only exist within the pipeline's artefact set.

There may be a sixth mode: cross-session learning.
Whether structural lessons from one session carry to a later session, while syntactic reflexes do not, is the subject of a separate evaluation.

## The uv pivot, mid-implementation

At roughly 49% of the session quota, I noticed that Poetry was not installed on my machine.
Yet the spec assumed it was.
The tool's `R9` entry in `research.md` had explicitly considered switching to `uv` during the research phase and had deferred it on the grounds that *"Poetry is already installed."*
I had also revised my position on `uv` since the research phase: it was now my preferred tool, not a tied alternative.

I sent a pivot request asking the tool to revisit `R9` honestly rather than retrofit a justification.

The revision landed:

> *"Both premises were re-examined before T002 and found to be false."*

The R9 entry was rewritten with the corrected premises and a re-derived recommendation.
The Constitution version bumped from v1.2.0 to v1.3.0, documented as two cumulative MINOR groups (the original amendment block plus the `uv` switch).
While bumping the version, the tool noticed a fourth version-string drift elsewhere in the document set and corrected it without prompting.

The tool did not defend the original `R9` reasoning when its premises were challenged.
Instead of producing a longer justification for Poetry or splitting the difference, it stated "both premises were false" and rewrote.
When a measured outcome contradicts a fabricated mandate, the tool does not defend the mandate.

The converse failure mode is defensive fabrication: the tool elaborates a defence rather than retracting.
It also appeared in this run, in a different context.
Both will be covered in detail later on.

## The architecture document as a vendable artefact

`T009` produced a 400-line reverse-engineering document of the legacy code.
Eight quirks were documented explicitly, including a rollback-SQL bug in the original code that the tool spotted while reading it.

Three caveats stop me from calling this an unqualified success.

The document was generated from static analysis only.
It missed three runtime bugs that surfaced during later phases: a search-crash bug in `helpers.py`, a menu-number normalisation bug in `UserInteraction.update`, and a logger-disable bug in `migrations/env.py`.
Each was discovered when the relevant code path was actually exercised: the first two during integration test work, the third during T045.
A reverse-engineering document generated by a Spec-Kit-style pipeline must be marked "static analysis only. Runtime behaviour was not verified" until the documented paths are exercised against tests.

The document is internally well-structured but inherits the quirk-detection ceiling of static analysis: whatever the tool could not see in the source, it did not document.
Treating the documentation as complete would be the same category of error as treating the Constitution-generation output as a complete extraction.

With those caveats, the document is quite like the Constitution: it accelerates the work of a reviewer who reads the codebase fresh.
But it must be treated as a starting hypothesis rather than as ground truth.

## Closing

The implementation phase highlighted the pros and cons of artefact structure in a way the spec phase had not.

The cascade-on-delete verification was caught only because I traced the call graph.
The contract had right behaviour with the wrong cited mechanism.
That gets harder to catch as the codebase grows.

The `ConfigReader` cross-artefact fix happened without prompting and against my explicit decision to skip the work.
It is the sharpest example of a pipeline-specific capability: consistency turns into a reasoning problem the tool can solve.

I record the `uv` pivot because the tool revised honestly rather than defending the original conclusion.
The `T009` architecture document is useful, but static-analysis-only documentation has known blind spots that only running code reveals.

Whether any of this scales to a 5,000-script ERP system is the question [Post 4](/blog/spec-kit-brownfield-applicability) takes on.
The four-variables framework that organises the answer comes from looking across the entire eval rather than at any single phase.

---

*Post 3 of 4 in a series evaluating Spec Kit on a brownfield codebase. Series notes: pdb_search ([github.com/HubReb/pdb_search](https://github.com/HubReb/pdb_search)) and basicToDo ([github.com/HubReb/basicToDo](https://github.com/HubReb/basicToDo)), Claude Code with Opus 4.7 at xhigh effort, [Spec Kit v0.7.5](https://github.com/github/spec-kit/tree/v0.7.5), eval conducted April–May 2026.*

*One engineer's eval on two personal codebases over five sessions, with one tool that's still evolving.*

*Next: [where Spec Kit fits in ERP modernization — applicability and limits](/blog/spec-kit-brownfield-applicability).*
