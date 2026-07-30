---
layout: page
title: Projects
subtitle: AI-Assisted Legacy Modernization
---

## Automated Code Review Pipeline

A CI/CD-integrated code review system running local LLMs on a single 32GB VRAM GPU. Reviews every merge request and runs deeper nightly batch reviews — all on-premises, no code leaves the network.

**Stack:** llama.cpp, FastAPI, Redis, FAISS + BM25 (hybrid RAG), Docker, GitLab CI/CD, Prometheus

Two-tier model setup: 70B-class dense model for MR reviews, 200B+ MoE for nightly deep reviews. Hybrid retrieval (semantic + keyword) for domain-aware context injection. Model hot-swap via Docker API to run different models on the same GPU. Q4_K_M quantization as the sweet spot for fitting serious models on consumer-grade hardware.

**Blog posts:** [Local LLMs for Code Review](/blog/local-llm-code-review) · [When Nobody Reads the Code Anymore](/blog/knowledge-bottleneck)

---

## Python 3 Migration Bridge

PTY-based interprocess bridge connecting Python 3 to a legacy C runtime without touching tens of thousands of lines of C++ bindings. Instead of porting the embedded interpreter's bindings to the Python 3 C API, a marker protocol over pseudo-terminal replaces the entire integration approach.

**Stack:** Python 3, PTY subprocess management, custom marker protocol, pytest

Process isolation instead of binding migration. Sequential send/wait/read to avoid PTY race conditions. Text-based markers for debuggability. Repository and gateway layer for patterns that don't survive process boundaries.

**Blog post:** [When You Can't Embed, Bridge](/blog/ipc-bridge)

---

## Multi-Agent Development Workflow

Development workflow for the Python 3 migration using frontier models with domain-specific review agents. Each part of the codebase has its own expert agent loaded with architecture docs and source. A reasoning model plans and specifies, an instruction-following model implements with TDD, and the domain agents review at every phase.

**Stack:** Frontier model APIs, custom agents with RAG-augmented domain context, pytest

The workflow itself is a sequential loop. The complexity is in what each agent knows, not how they're wired together.

**Blog post:** [One Agent Per Domain, Zero Trust](/blog/multi-agent)

---

## Translation Pipeline & Verification

Spec-driven translation of a proprietary scripting language to Python 3, without fine-tuning or training data. The model gets a translation spec instead of language knowledge. Verification through database state comparison: run the original, run the translation, diff.

Key findings: reasoning models explore when they should comply — mechanical translation needs the instruction-following tier. Parity testing against the database is the only verification that doesn't depend on understanding a system nobody fully understands.

**Blog posts:** [Why the Smartest AI Agent Is the Worst Translator](/blog/smartest-agent-worst-translator) · [Teaching an LLM a Language It Has Never Seen](/blog/teaching-llm-unseen-language) · [Parity Testing: The Only Definition of Correct](/blog/parity-testing) · [Conservative by Design: Transpiling Without Guessing](/blog/transpiler)

---

## Y2038 and LP64 Migration Analysis

Risk analysis and migration roadmap for a 30+ year old 32-bit C codebase. The Y2038 work found a compile flag (`_TIME_BITS=64`) that reduced estimated effort from 12-18 months to 7-10 weeks. The LP64 analysis traced runtime crashes to 4 typedefs in a central header.

**Stack:** C, GCC, `-Wconversion`, Oracle, grep

Both migrations fix different code sites and share a canary deployment, bringing combined scope to 7-11 weeks instead of 18.

**Blog posts:** [Y2038: When "Impossible" Means "Wrong Approach"](/blog/y2038-analysis) · [When "It Compiled" Is the Dangerous Part](/blog/lp64-analysis) · [Quantifying a Legacy Codebase You Can't Rewrite](/blog/quantifying-legacy)

---

## Spec Kit Evaluation on Brownfield Codebases

Four-post series evaluating GitHub's Spec Kit pipeline on two codebases: a curated full-stack toy app with planted bugs, and a three-year-old personal CLI with real history. Both runs were Constitution-driven. The brownfield case exposed extraction failures the toy case couldn't. Constitution-generation fabricates principles to fill template slots the codebase doesn't speak to. It also omits conscious decisions visible in commit history.

**Stack (toy):** FastAPI, SQLAlchemy 2, aiosqlite, Uvicorn, pytest-asyncio, mypy (strict), pylint, black, slowapi  
**Stack (CLI):** SQLAlchemy 2, Alembic, psycopg 3, PostgreSQL, Typer, Rich, cryptography, pybtex, ruff, mypy (strict), pytest-postgresql, uv, hatch  
**Eval:** Spec Kit v0.7.5, Claude Code (Opus 4.7), git

Three error categories surfaced: hallucination from gap, hallucination from over-tightening, omission. The first two are mechanically catchable through sharper principles plus self-audit at phase boundaries. Omission requires the original author with commit history open, which doesn't scale to legacy codebases whose authors are gone.

**Blog posts:** [Spec Kit on a Brownfield Codebase](/blog/spec-kit-brownfield-setup) · [Auditing as You Go: What Corrections Actually Look Like](/blog/spec-kit-brownfield-self-audit) · [Implementing the Spec: Where Discipline Holds and Where It Slips](/blog/spec-kit-brownfield-implementation) · [Spec Kit's Fit and Limits in ERP Modernization](/blog/spec-kit-brownfield-applicability) · [Don't Let a Model Grade Its Own Family](/blog/scorer-same-family-bias) · [Hard Gates Break the Weaker Model](/blog/offload-boundary) · [The Cards Are Always Face Up](/blog/cards-always-face-up)

---

## Reflections on Engineering Culture

What LLM-augmented engineering does to the institutions around the code: comprehension can't form fast enough, skill no longer predicts performance, and the institutional language has no name for the resulting asymmetries. Cross-cutting reflection rather than project documentation. Read or skip independently of the project posts.

**Blog posts:** [When Nobody Reads the Code Anymore](/blog/knowledge-bottleneck){% assign feedback_post = site.posts | where: "url", "/blog/feedback-that-changes-behaviour" | first %}{% if feedback_post %} · [Feedback That Changes Behaviour Is Feedback You Can Check](/blog/feedback-that-changes-behaviour){% endif %}{% assign grade_post = site.posts | where: "url", "/blog/grade-your-evidence" | first %}{% if grade_post %} · [Grade Your Evidence, Not Your Confidence](/blog/grade-your-evidence){% endif %}

---

## What's Next

More libraries to migrate, pilot scripts to validate, and a database layer to rebuild. Posts will follow when milestones are done.
