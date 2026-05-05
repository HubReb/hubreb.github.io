---
layout: post
title: "Auditing as you go: what corrections actually look like"
subtitle: "Vague principles produce compliance theater. Sharp principles produce mechanical self-correction."
tags: [spec-kit, brownfield, ai-assisted-development, evaluation]
permalink: /blog/spec-kit-brownfield-self-audit
---

*"Claude Code might be over-eager and add components that you did not ask for."* is the warning spec kit includes in its own README.

In the previous post, three categories of Constitution-extraction error were shown: hallucination from gap, hallucination from over-tightening and omission. The first two were catchable through self-audit at phase boundaries whereas the third was not.

Spec kit was run on the actual brownfield code base. 
I made four audit moves. 
Three were user-driven: 
I noticed something, drafted an audit and sent it. 
One was tool-autonomous: the tool retracted its own hallucination during a Constitution Re-Check before my audit was sent. 
One additional finding, *audit-class amplification*, came from the tool during one of those user-driven audits: 
I had pointed at one instance of an error class, and the tool searched its own output for other instances of the same class.

Let us look at this small empirical record of what self-audit looks like when Constitution principles are sharp in detail.

## Audit Move 1: The fabricated performance budget

The first hallucination from Post 1's table was the one-second-per-ten-thousand-papers performance budget:
The Constitution stated it as a quantitative principle. 
Yet, the codebase contained nothing to justify the number. 
I had never measured performance beyond 'runs without pause and doesn't annoy me' - which is what most of my private projects use as benchmark.

I sent the audit during the spec phase: rewrite the corresponding success criterion (SC-006) as non-regression-against-baseline rather than against a fabricated absolute, amend the Constitution principle in parallel, and treat the fabricated number as the issue rather than the principle's existence.

The tool's response, verbatim:

> *"You're right that the 1s/10k numbers were ungrounded; I inserted them to give the principle a testable shape but never grounded them in measurement. Treating this as constitution v1.1.0 (MINOR — principle predicate is redefined, but it's a relaxation, not a removal)."*

Five edits landed in parallel: spec FR-017, spec SC-006, the Constitution bumped from v1.0.0 to v1.1.0 with a new Sync Impact Report entry, the `CLAUDE.md` summary, and the requirements checklist. 
The new principle rationale, verbatim:

> *"The criterion is framed as 'non-regression vs. the current baseline on a personal-library-sized dataset' rather than as fabricated absolute numbers, because the current implementation has not been benchmarked and any specific bound would be a guess. Refactors are evaluated against measured baseline, not against a fictional target."*

Two things to note: 
The tool described its own past behaviour mechanically: 
*"I inserted them to give the principle a testable shape but never grounded them in measurement"* is the fabrication pattern from Post 1: the Constitution-template required a quantitative slot, but the code base didn't supply one. Hence, the model produced a plausible default. 
The propagation across all five artefacts landed on the first response, with the version bump correctly classified as MINOR (relaxation rather than removal).

It cannot be determined whether the tool would have caught the fabrication unprompted. The next audit gives stronger evidence on that question.

## Audit Move 2: NOT NULL drift

Two schema discrepancies surfaced in the design phase. 
The first: the modernised schema declared NOT NULL on most columns. Yet, the legacy `create_tables()` had bare INT/VARCHAR with no constraint. 
The second: `bib.bibtex` was declared NOT NULL alongside an otherwise-correct UNIQUE translation. But the original had no NOT NULL there either.

Both fall into Post 1's over-tightening category: a pattern read correctly (uniqueness, non-empty content) was inflated into stricter policy than the code actually enforced. 
I drafted the audit listing the two issues and sent it.

The tool's response, verbatim:

> *"Good catch — and there's a third schema-tightening issue I introduced that you didn't mention but is in the same class: I added FK constraints on authors_papers(author_id, paper_id) that don't exist in the original (the original create_tables() declares them as bare INT with no FK). Going to fix all three together since they're the same kind of mistake."*

Then, the corrections propagated across three artefacts:

- Schema: verbatim with the legacy contract: no NOT NULL on the affected columns and no new FKs. The only FK defined was the single existing FK `papers.bibtex_id → bib.bibtex_id`
- `data-model.md` entity tables: "Required, non-empty" claims had been removed from schema-constraint columns and moved into a separate "Application-level validation" subsection
- SQLAlchemy mapping: nullable columns were now typed `Mapped[str | None]` / `Mapped[int | None]`. The ORM navigation had been rewritten to use `relationship(primaryjoin=..., secondaryjoin=...)`. Thus, the ORM's `MetaData` matches the schema 1:1 and no FK DDL is emitted at metadata-create time.

The standard SQLAlchemy form is `mapped_column(ForeignKey(...))`, which emits the FK DDL the audit asked to remove. 
Keeping ORM-side navigation while suppressing DDL emission requires the explicit `primaryjoin`/`secondaryjoin` form, which the tool selected on its own.

The FK constraint on `authors_papers` was not in my audit; the tool found it.
The next section covers that finding.

## Audit-class amplification

The FK constraint on `authors_papers` was not in my audit. 
The tool reported it in response to my audit on the two NOT NULL issues.
The generalisation visible in the verbatim quote: from the two instances of schema-tightening drift I had named to other instances of the same class. 
Its own phrase for the class — "the same kind of mistake" — is in the quote.
It found one new instance, and bundled it with the corrections I had asked for.

The legacy `create_tables()` had bare INT for `authors_papers.author_id` and `paper_id`; the modernised schema had added FK constraints. Removing them restored the legacy contract that 001-modernize-stack was constrained to preserve.

The wording of an audit shapes how much the tool corrects: a narrowly-scoped audit gets the specific case fixed, a class-named audit may get the whole class fixed.

## The autonomous retraction

This episode happened during the design phase, before any audit on indexes had been sent. 
I had drafted such an audit but not yet submitted it. 
The tool, on its own initiative, re-ran the post-design Constitution Check and announced:

> *"Re-evaluating Constitution Check post-design — I see one issue. I added two indexes (papers.title, authors_id.author) in database-schema.md 'to back the search hot path', but Principle IV (v1.2.0) explicitly says any new index beyond the existing primary keys requires Complexity Tracking justification. Adding them speculatively against a non-regression budget is exactly the rationalisation the principle warns against. Removing them."*

The two indexes had been added during the design phase with the rationale *"to back the search hot path"*: a phrase the tool had inserted itself. There was no measurement and no benchmark behind it. 
This is the same fabrication pattern as Audit Move 1: a quantitative-shaped claim invented to justify a structural addition.

By the time the post-design Constitution Re-Check ran, Principle IV had been amended in Audit Move 1 from a fabricated absolute target to a non-regression rule with explicit Complexity Tracking. 
The amended principle was sharp enough to apply mechanically: any added index has to either fall out of an existing primary key or carry a Complexity Tracking justification. 
The two speculative indexes had neither. 
The tool ran the principle against its own design output, found the violation, and removed the indexes.

This is the only audit move in the run that I did not have to make. 
The tool's own scheduled self-check found the violation.

## What predicts self-audit effectiveness

Looking back across the four episodes, one variable determines whether the tool corrects an error, on its own or in response to an audit.
It is the sharpness of the Constitution principle that governs the area where the error occurred.

In Audit Move 1, Principle IV had stated a fabricated absolute (1s/10k) as its predicate.
That predicate was vague: it could not be applied to any specific design decision without further interpretation. 
Because the codebase had no measurement to compare against, the fabrication remained until I audited it directly.
The principle could not be applied mechanically.
Hence, the principle could not catch it.

After the amendment, the same Principle IV stated a non-regression rule plus explicit Complexity Tracking.
That predicate was sharp: for any proposed index, the tool could ask "does this fall out of an existing primary key, or does it carry a Complexity Tracking justification?" and answer mechanically. 
The autonomous retraction in the previous section was that mechanical application running on the tool's own design output.

In Audit Move 2, the principle being violated (schema preservation against the legacy contract) was sharp in form but had not been encoded as a Constitution principle.
The tool had inferred a vaguer "modernise the schema" goal and over-tightened.
Catching it required me to point at specific instances. 
Then, the tool generalised across the class once the error type had been named.

There is a pattern across all four episodes: vague principles produce compliance theater.
The tool reports the principle as satisfied without being able to check it. 
Sharp, verifiable principles produce mechanical self-correction.
The tool can then run the predicate against its own output and find violations. 
The autonomous retraction in Audit Move 1's wake was not a different kind of capability from the user-driven audits. 
It was the same checking machinery, applied to a principle that had become verifiable enough to drive it.

That matters for the omission category from Post 1.
Omission is the failure to elevate a pattern to a Constitution principle.
If the pattern never becomes a principle, no sharpening can be done to it later, and no self-audit can find it.
Sharpness mitigates the first two error categories.
It cannot reach the third.

## Five modes of self-correction

The four episodes above are instances of broader patterns. 
The full run produced five distinct modes of self-correction, plus a sixth that requires data from later sessions.

| Mode | Trigger |
|---|---|
| User-direct | I point at a specific artefact and say it is wrong (Audit Move 1, NOT NULL portion of Audit Move 2) |
| Audit-class amplification | I point at one instance, the tool generalises and searches for others (FK constraints in Audit Move 2) |
| Autonomous-constitution | The tool re-runs the constitution check at a phase boundary and finds its own violation (the autonomous retraction) |
| Intra-artefact | The tool corrects itself mid-sentence within a single artefact during generation (not covered above; visible in tasks.md) |
| Cross-artefact anticipation | The tool checks that an amendment does not reference code that later tasks will delete (not covered above; visible in a Constitution amendment that removed a reference to code scheduled for deletion two phases later) |

The first three are the ones the four episodes above illustrated. The fourth and fifth will be covered in a separate post.

There may be a sixth mode: cross-session learning.
An audit correction in one session may prevent the same error pattern from recurring in a later session, be it structural or syntactic.
That behaviour was not verifiable from the data of this run alone.

## Closing

This post started with the README warning that Claude Code might be over-eager and add components that you did not ask for.
It describes the failure mode, not the mitigation.

All four episodes show one mitigation that worked. 
When a Constitution principle is verifiable, the tool can run it against its own output and correct itself.
This happens either in response to an audit or autonomously at a phase boundary. 
When a Constitution principle is not verifiable, audits remain entirely user-driven and the tool defaults to compliance theater. 
The sharpness of the principle predicts which of these modes applies.

Three open questions remain. 
Intra-artefact correction and cross-artefact anticipation as additional modes appeared in this run but in places that took longer to read; the next post works through them.
As for cross-session learning: Verifying this requires data across multiple sessions and is the subject of a separate evaluation. 
Finally, the omission category from Post 1 remains beyond the reach of any sharpness-based mitigation: A pattern that never becomes a principle cannot be sharpened later.

---

*Post 2 of 4 in a series evaluating Spec Kit on a brownfield codebase. Series notes: pdb_search ([github.com/HubReb/pdb_search](https://github.com/HubReb/pdb_search)) and basicToDo ([github.com/HubReb/basicToDo](https://github.com/HubReb/basicToDo)), Claude Code with Opus 4.7 at xhigh effort, [Spec Kit v0.7.5](https://github.com/github/spec-kit/tree/v0.7.5), eval conducted April–May 2026.*

*One engineer's eval on two personal codebases over five sessions, with one tool that's still evolving.*

*Next: Self-correction beyond the principle — intra-artefact and cross-artefact patterns.*
