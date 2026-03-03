---
layout: page
title: Projects
subtitle: AI-Assisted Legacy Modernization
---

## Automated Code Review Pipeline

A CI/CD-integrated code review system running local LLMs on a single 32GB VRAM GPU. Reviews every merge request and runs deeper nightly batch reviews — all on-premises, no code leaves the network.

**Stack:** llama.cpp, FastAPI, Redis, FAISS + BM25 (hybrid RAG), Docker, GitLab CI/CD, Prometheus

**Key decisions:**
- Two-tier model setup: 70B-class dense model for MR reviews, 200B+ MoE for nightly deep reviews
- Hybrid retrieval (semantic + keyword) for domain-aware context injection
- Model hot-swap via Docker API to run different models on the same GPU
- Q4_K_M quantization as the sweet spot for fitting serious models on consumer-grade hardware

**Blog post:** [Local LLMs for Code Review](/blog/local-llm-code-review)

---

## AI-Assisted Legacy Code Translation

A two-model pipeline translating 6,000+ scripts from a proprietary language (zero internet presence, no training data) to Python 3. Functional parity verified by database state comparison.

**Stack:** Claude Opus (analysis + spec writing), Claude Sonnet (mechanical translation), Python 3, Oracle, PostgreSQL

**Key decisions:**
- Spec-driven translation instead of fine-tuning — the model doesn't learn the language, it follows a map
- Reasoning model for architecture, execution model for translation — separation of concerns applied to AI tooling
- Parity testing via database state snapshots as the only definition of "correct"
- Reverse engineering bottom-up before translating a single line

**Blog posts:** [Why the Smartest AI Agent Is the Worst Translator](/blog/smartest-agent-worst-translator) · [Teaching an LLM a Language It Has Never Seen](/blog/teaching-llm-unseen-language) · [Parity Testing](/blog/parity-testing)

---

## AI-Assisted Y2038 Migration Strategy

Analysis and migration strategy for Y2038 (Unix timestamp overflow) across a legacy system. AI-assisted scope reduction turned a 6-12 month estimate into 2-4 months by identifying which code paths actually touch timestamp-sensitive operations.

**Approach:**
- Automated static analysis to identify affected code paths
- Classification of timestamp usage patterns (display-only vs. calculation vs. storage)
- Prioritization by business impact rather than code location
- Migration strategy that separates mechanical fixes from semantic changes

**Blog post:** Coming soon

---

## What's Next

The translation pipeline is in early production. The bet: spec quality improves faster than script complexity increases. If that holds, the pipeline scales to the full 6,000+ script codebase. If not, the next step is fine-tuning on the growing corpus of verified translations.

I'll document the journey here.
