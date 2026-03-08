---
layout: page
title: Projects
subtitle: AI-Assisted Legacy Modernization
---

## Automated Code Review Pipeline

A CI/CD-integrated code review system running local LLMs on a single 32GB VRAM GPU. Reviews every merge request and runs deeper nightly batch reviews — all on-premises, no code leaves the network.

**Stack:** llama.cpp, FastAPI, Redis, FAISS + BM25 (hybrid RAG), Docker, GitLab CI/CD, Prometheus

Two-tier model setup: 70B-class dense model for MR reviews, 200B+ MoE for nightly deep reviews. Hybrid retrieval (semantic + keyword) for domain-aware context injection. Model hot-swap via Docker API to run different models on the same GPU. Q4_K_M quantization as the sweet spot for fitting serious models on consumer-grade hardware.

**Blog post:** [Local LLMs for Code Review](/blog/local-llm-code-review)

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

## Y2038 and LP64 Migration Analysis

Risk analysis and migration roadmap for a 30+ year old 32-bit C codebase. The Y2038 work found a compile flag (`_TIME_BITS=64`) that reduced estimated effort from 12-18 months to 7-10 weeks. The LP64 analysis traced runtime crashes to 4 typedefs in a central header.

**Stack:** C, GCC, `-Wconversion`, Oracle, grep

Both migrations fix different code sites and share a canary deployment, bringing combined scope to 7-11 weeks instead of 18.

**Blog posts:** [Y2038: When "Impossible" Means "Wrong Approach"](/blog/y2038-analysis) · [When "It Compiled" Is the Dangerous Part](/blog/lp64-analysis)

---

## What's Next

More libraries to migrate, pilot scripts to validate, and a database layer to rebuild. Posts will follow when milestones are done.
