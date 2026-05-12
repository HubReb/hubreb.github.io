---
layout: post
title: "Spec Kit on a brownfield codebase: setup and first impressions"
subtitle: "The interesting hallucinations show up in the Constitution, not the implementation."
tags: [spec-kit, brownfield, ai-assisted-development, evaluation]
permalink: /blog/spec-kit-brownfield-setup
---

In April and May I ran the same Spec Kit pipeline against two codebases with the same prompt. One was a small full-stack toy app I had refactored several times before the run, with intentional bugs left in place: sync sessions under async routes, dual ORM mapping, a hardcoded `"not implemented yet"` description, a header typo. Spec Kit's pipeline ran through it twice (Opus 4.6, then Sonnet 4.6 in a separate run) and reported all gates passing on completion. Tests claimed passing: 440 in run 1. Tests actually passing on the merged branch: 55, plus one error from a rate-limiter the pipeline added without adding a `conftest.py` to disable it during tests. External CI on each merged branch: 12 of 20 checks failing.

The second codebase was a three-year-old personal CLI of mine. PostgreSQL backend, Fernet-encrypted config file, around 2000 lines of Python. Real historical accidents, not curated bugs. A typo (`bibtext_id`) baked into the schema. Three procedural modules from before I bothered to write object-oriented code, sitting alongside the OO stack that replaced them. A placeholder test that always failed. Integration tests silently dependent on a live local database.

The first run mostly confirmed what's already documented elsewhere, including in Spec Kit's own README: pipeline self-validation is not a quality gate. The brownfield run was different. The failure mode shifts when the codebase is real, and the interesting hallucinations show up in the Constitution, not in the implementation.

This post is about the Constitution-generation step, because that's where the pipeline first touches the codebase and what it produces there shapes everything downstream. Later posts cover the audit moves I made during the run, the implementation phase, and where this all does and doesn't fit in real ERP modernization.

## What Spec Kit does first

Before any of the documented pipeline phases (`/specify`, `/plan`, `/tasks`, `/analyze`, `/implement`), the Claude Code instance hosting the pipeline reads the codebase. On the brownfield run, that pre-pipeline read produced a six-finding `CLAUDE.md` document: README-vs-reality mismatch on the run command path, two distinct test problems, the three-layer architecture identified with its driver-isolation boundary, the four-table schema with FK relationships reverse-engineered, three legacy procedural modules flagged as not wired into `run.py` and as still using the `bibtext_id` typo, and the constitution recognized as an unfilled template.

I had not asked for any of this, and I had not even issued a pipeline command. The model had read the code on its own and surfaced what it noticed.

For comparison: the curated-toy runs produced no equivalent. There was nothing for a pre-pipeline pass to find that the curated bugs hadn't already documented for me.

That set the expectation that the brownfield run would surface more, and surface it earlier.

## The Constitution

The Constitution is a four-principle document (code quality, testing, UX consistency, performance) that downstream pipeline phases must respect. Each principle has a predicate.

For the brownfield run, the Constitution had to be reverse-engineered from observed code patterns. Six mandates surfaced across the four principles. Four matched real architectural decisions I had made; two had no commit or code evidence behind them. A seventh conscious decision, clearly visible in the commit history, was missing from the Constitution entirely; I've added it to the table for comparison.

## The author-honesty table

I pulled the commit history for the relevant files and went through it line by line. Some of the patterns the tool surfaced are visible in commit messages as decisions I had documented at the time. Some are not. Here's the breakdown after the verification pass:

| Mandate | Origin (verified by commit) | Tool behaviour |
|---|---|---|
| Type hints everywhere | Conscious, Mar 4 2023 commit "add type hints"; Dec 25 2023 commit "add type hints for new methods" | Correctly extrapolated |
| OO architecture | Conscious, Mar 3 2023 commit "oop restructure" | Correctly extrapolated |
| Driver isolation (`psycopg2` only in `psycopg_db.py`) | Conscious, Mar 20 2023 commit "extract psycopg into separate class - ETC" | Correctly extrapolated and elevated to architectural mandate |
| Pylint-clean | Conscious, three separate "satisfy pylint" / "fix pylint" commits across 9 months in 2023 | Correctly extrapolated |
| Rollback-on-add-failure | Conscious, Apr 23 2023 commit "change adding of database entry to remove incomplete information from DB if add fails" | **Omitted**: not elevated to a Constitution principle |
| 1 second / 10 000 papers performance budget | Not conscious; never measured | **Fabricated** to "give the principle a testable shape" |
| Legacy modules frozen | Not decided | **Fabricated** policy |

The four "correctly extrapolated" rows have commit evidence. The driver-isolation extraction is the most notable of these: the pipeline correctly identified that I was treating `psycopg_db.py` as a hard boundary and elevated that to an architectural mandate, with the supporting commit visible in the history.

The two fabricated rows have no commit evidence and no code evidence either. The 1-second-per-10,000-papers performance budget I never measured. There is no commit anywhere in the repository that reasons about performance. The number was constructed because the Constitution-template's performance-principle slot needed a quantitative answer and the code didn't supply one. The legacy-modules-frozen policy I never decided. The procedural modules sit alongside the OO stack because I didn't migrate them, not because I ratified their existence as frozen.

The omitted row is the most interesting. April 23, 2023, commit message verbatim: *"change adding of database entry to remove incomplete information from DB if add fails."* Rollback-on-add-failure. Data-integrity discipline as direct as a commit message gets. The pipeline read both the commit and the code implementing it, but the Constitution doesn't mention rollback anywhere.

The dates on these commits split across the boundary of when I started a new job in 2023 and applied actual engineering discipline. Two of the patterns I'm credited with reflect what I learned during that employment; the rest reflect earlier patterns from when I was still a student. The pipeline can extract both vintages of intent from the same repository, without distinguishing them. For the purposes of this post that's a side observation. But it points to a finding worth flagging: the Constitution treats a codebase as a single coherent set of decisions, when in fact most real codebases are layered. Constitution mandates extracted from a layered repository implicitly average across the layers.

## Three categories of extraction error

| Category | Mechanism | Example |
|---|---|---|
| Hallucination from gap | Tool fills a Constitution-template slot the code doesn't speak to, with a plausible default | 1-second performance budget; legacy-modules-frozen policy |
| Hallucination from over-tightening | Tool reads a code pattern correctly but inflates it into stricter policy than the code actually enforces | (surfaced later in the eval; covered in the next post) |
| Omission | Tool sees a pattern but doesn't elevate it to a Constitution principle | Rollback-on-add-failure |

The first two are catchable. If a Constitution principle is sharp enough (something with a mechanically-checkable predicate), the pipeline can re-run its own checks at phase boundaries and find its own hallucinations. Vague principles can't catch fabrications; sharp principles can. There's a working example of this from later in the eval that I'll cover in the next post.

The third category is the one without a clean mitigation. Omission is invisible to the pipeline's self-audit mechanisms. The pipeline cannot know what conscious decision it failed to elevate. Only the author who wrote the original code, with the commit history open, can recognize the gap. For a personal codebase that's me with an afternoon and a `git log`. For a 5,000-script ERP system whose original authors left the company a decade ago, there is no reachable author to perform the equivalent check.

## What I take from this

The pipeline reads code well. Even subtle architectural patterns get correctly identified, with commit-message evidence in some cases. The failure mode is structural. When the Constitution-template asks for a number and the code doesn't supply one, the model produces a plausible default. Patterns that don't fit a template slot go uncaptured.

For curated greenfield work, this matters less. The Constitution-template's slots and the codebase's architectural axes are designed under the same assumptions. For a real codebase whose architectural decisions evolved organically, the slots and the axes diverge. Where they diverge, the Constitution drifts away from what the codebase actually decided.

Sharper Constitution principles plus a self-audit pass at phase boundaries mitigate the gap and over-tightening categories. Omission has no equivalent mitigation. Catching it requires an author-audit against the commit history, which neither the template nor the model can perform.

That distinction is what shapes how the rest of the eval went. Some of the corrections I made directly where I noticed them. The pipeline made others on its own when its principles were sharp enough to catch its own mistakes. One it caught anticipatorily, extending an amendment to cover code that a later task would delete, without my asking.

---

*Post 1 of 4 in a series evaluating Spec Kit on a brownfield codebase. Series notes: pdb_search ([github.com/HubReb/pdb_search](https://github.com/HubReb/pdb_search)) and basicToDo ([github.com/HubReb/basicToDo](https://github.com/HubReb/basicToDo)), Claude Code with Opus 4.7 at xhigh effort, [Spec Kit v0.7.5](https://github.com/github/spec-kit/tree/v0.7.5), eval conducted April–May 2026.*

*One engineer's eval on two personal codebases over five sessions, with one tool that's still evolving.*

*Next post: [auditing as you go](/blog/spec-kit-brownfield-self-audit).*
