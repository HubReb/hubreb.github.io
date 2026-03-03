---
layout: post
title: "Local LLMs for Code Review: Why I Don't Send Proprietary Code to the Cloud"
subtitle: "llama.cpp, a proprietary codebase, and the compliance department that would murder me."
tags: [llm, code-review, local-inference, rag]
permalink: /blog/local-llm-code-review
---

I built an automated code review pipeline for a legacy codebase. The code is proprietary. The language is proprietary. The business logic is proprietary. Sending any of it to an external API was not an option.

So I ran the LLM locally. Here's what works, what doesn't, and why local inference is underrated for enterprise codebases.

## The Constraint

The codebase contains decades of business logic for an ERP system. Customer-specific configurations, pricing rules, workflow definitions — the kind of intellectual property that makes legal departments nervous. Uploading it to OpenAI, Anthropic, or any cloud API means it leaves the network. Compliance says no. And they're right.

This isn't paranoia. It's the reality for most enterprise codebases. The companies that most need AI-assisted tooling are the ones that can least afford to send their code outside.

## The Setup

**llama.cpp** for inference. One machine with a 32GB VRAM GPU that I had to fight for. ("We need this for the code review pipeline." "Why don't you set up a private GitHub repo and actions?!?" "Because compliance would murder us." "...approved.") llama.cpp runs quantized models on that single card. It's not fast. It doesn't need to be. Code review isn't real-time.

**Model selection** runs two tiers: a 70B-class dense reasoning model (Q4_K_M) for merge request reviews, and a 200B+ MoE model (Q4_K_M) for deeper nightly reviews. Both quantized to fit the single card. The specific models change as better ones come out. The principle stays: fast and good enough for MR reviews, slow and thorough for nightly batch runs.

**The pipeline** integrates into CI/CD. Push code, trigger review, get comments. No manual step, no copy-paste into a chat interface, no context switching. The review is there when you open the merge request.

## What You Actually Get

Here's a real review output for a translated legacy script. The model reviews Python code that was translated from a proprietary language, checking for bugs, API compatibility, and edge cases:

> **Potential Bugs:**
> In `_handle_lock()`, when `lock_state > 0`, `retry_count` is decremented. If `retry_count` is 0, this could make it negative, potentially causing unintended behavior in the retry logic.
>
> **API Compatibility:**
> The function signature and return values match the expected API contract. The use of process dictionaries aligns with the legacy conventions.
>
> **Performance:**
> The exponential sleep backoff is efficient and prevents busy waiting. The loop includes proper stagnation handling to avoid indefinite blocking.
>
> **Edge Cases:**
> The code handles the case where `lock` is None by falling back to the default lock state. However, the negative `retry_count` scenario in `_handle_lock()` could be problematic.

That negative `retry_count` catch is real. A human reviewer might have missed it in a 200-line translation. The model flagged it on the first pass.

This isn't "add a docstring" level feedback. This is domain-aware review of translated legacy code, catching a subtle logic error in retry behavior.

## Why RAG Makes or Breaks It

Here's what I learned the hard way: a local model reviewing proprietary code without context produces noise. It doesn't know your conventions, your architecture, your API contracts. It flags things that are correct and misses things that are wrong.

RAG (Retrieval-Augmented Generation) changed the quality from "interesting experiment" to "production pipeline."

The setup: FAISS for semantic search, BM25 for exact term matching, hybrid retrieval with reciprocal rank fusion. Documentation gets chunked, indexed, and injected into the review prompt. When the model reviews a file that uses a custom database write function, it gets the documentation for that function in context. When it reviews a migration script, it gets the translation conventions.

The difference is night and day. Without RAG, the model reviews Python code. With RAG, it reviews Python code *in the context of this specific codebase and its conventions*. The negative `retry_count` catch above? That only works because the model knows the API contract from the injected documentation.

Hybrid retrieval matters because proprietary codebases have identifiers that semantic search can't handle. Internal constants, custom function names, domain-specific class names: BM25 catches these by exact match where dense retrieval would miss them. Neither retrieval method alone is sufficient. Together, they cover both semantic similarity and exact identifiers.

## What Local LLMs Are Good At

**Pattern detection across a codebase.** "This function does the same thing as that function in a different file." Local models catch duplication and inconsistency well, because you can feed them large chunks of the codebase without worrying about token costs or data leakage.

**Style consistency.** Does this new code follow the conventions established in the rest of the codebase? Local models can ingest the existing code as context and flag deviations. No fine-tuning needed, just enough context.

**Documentation gaps.** "This function modifies database state but has no docstring explaining which tables it touches." The model identifies what's missing, not what's wrong. Less controversial than "your code is bad," more useful than a linter.

**Semantic traps.** In a proprietary language with implicit behaviors, there are patterns that look correct but aren't. "You're using construct X, which silently commits. Did you mean to commit here?" This is where local LLMs outperform traditional static analysis: they can reason about intent, not just syntax.

## What Local LLMs Are Bad At

**Deep architectural review.** "Should this service exist?" is still beyond what a local model can answer meaningfully, even at 70B. Architectural decisions need the kind of reasoning and context that cloud-tier models handle better. But that's not what the pipeline is for.

**Correctness guarantees.** The model flags potential issues. It doesn't prove correctness. Parity testing does that. The code review catches the obvious mistakes early, the parity test catches everything else later.

**Novel patterns.** If the codebase introduces a pattern the model hasn't seen in training, it might flag it as wrong simply because it's unfamiliar. False positives from novelty are the most annoying failure mode.

## The Economics

Cloud API cost for reviewing 6,000+ scripts: significant. Token costs add up when each review needs the source script, the translation spec, the translated output, and relevant library context.

Local inference cost: one machine, running overnight. The electricity bill is negligible. The model is free. The setup time was a few days.

For ongoing use, local inference is essentially free per review. The cost is front-loaded in setup, not variable per use. For a migration project that runs months to years, that difference matters.

## The Tradeoff

Local models are worse than cloud models. This is not a debate. GPT-5.2, Claude Opus, Gemini 3.1 Pro all produce better reviews than quantized models running on a single GPU.

But "better review that violates compliance" isn't an option. "Adequate review that stays on-premises" is. And "adequate" is surprisingly good when the alternative is "no automated review at all," which is what most legacy codebases have.

The pipeline catches 60-70% of the issues that a senior engineer would catch in manual review. It catches them instantly, on every commit, without getting tired or distracted. The remaining 30-40% still need human eyes. But the human now reviews pre-screened code instead of raw output, which is a fundamentally different task.

## Production Lessons

**Quantization matters.** Q4_K_M is the sweet spot for fitting serious models on a 32GB card. The 70B dense fits comfortably. A 200B+ MoE with CPU offload squeezes in. Q2 is too lossy for code review. Q8 won't fit the models you actually want to run.

**Prompt engineering beats model upgrades.** A well-structured review prompt with specific instructions ("check for implicit commits," "flag unhandled cursor states," "verify type consistency") outperforms a larger model with a generic "review this code" prompt. The domain knowledge is in the prompt, not the weights.

**RAG is not optional.** Without domain context, local models produce generic reviews. With RAG, they produce domain-aware reviews that catch real bugs. The investment in indexing your documentation pays for itself on the first review that catches a semantic error.

**Batch, don't stream.** Code review doesn't need token-by-token output. Batch processing overnight means you can run larger models at acceptable speed. Morning coffee, fresh reviews. The workflow fits.

## Takeaway

If you're working with proprietary code and you're not using local LLMs for review, you're leaving value on the table. Not because local models are great. They're adequate. But adequate and compliant beats excellent and forbidden.

The compliance department sleeps well. The code gets reviewed. Everybody wins.

---

*I migrate decades-old systems to modern stacks — one script, one table, one timestamp at a time. I run 70B+ models on a single GPU to review code that can't leave the building. If your compliance department says no to cloud APIs, I've been there. Let's talk.*
