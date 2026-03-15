---
layout: post
title: "When Nobody Reads the Code Anymore"
subtitle: "I built an AI review pipeline. It works. The team stopped reading the code."
tags: [code-review, ai, legacy-systems, knowledge-management]
permalink: /blog/knowledge-bottleneck
---

An automated code review pipeline for a legacy codebase. Local LLM, RAG over internal documentation, integrated into CI/CD. Push code, get review comments on the merge request. No manual step, no copy-paste, no context switching.

It catches real issues. Edge cases, style violations, missing error handling, subtle logic errors in translated legacy code. It scales review throughput exactly the way you'd hope. The backlog of unreviewed merge requests disappeared within a week.

Until I realized why our technical discussions were starting to raise my hackles: the absence of dissent.

## What Review Actually Did

Before the pipeline, I read every merge request myself. The codebase is a migration project — proprietary scripting language to Python 3 — and I'm the one driving it. Every translated script, every new library wrapper, every test: I read it before it merged. Slow, but I knew what was in there.

When I built the pipeline late last year, it initially ran only on my own work. I use a [multi-agent development workflow](/blog/multi-agent) where domain-specific review agents check every phase before I see it. The AI review pipeline was another layer on top. I still read the code, because the workflow forces me to — the agents review, but I make the final call at every gate.

The knowledge degradation didn't happen to me. It happened when the pipeline went live for the rest of the team.

## The Compounding Problem

Once the pipeline covered all merge requests, other developers stopped reading code they didn't write. Not immediately — in the first weeks, everyone still had their mental models from months of manual review. The AI caught things faster. Velocity went up. Nobody complained.

By February, the codebase had grown by thousands of lines that no human besides me had reviewed in detail. The AI reviewed all of it. The AI caught bugs in all of it. But then, I noticed something in discussions: not a bug or a confused question. I noticed the lack of questions. And nobody tore apart design choices. Because no one but me had a mental model of the new modules.

This compounds. Every month of AI-only review is a month where the team's understanding of their own system degrades. Not because they're lazy. Because the forcing function is gone.

## What the AI Can't Replace

The pipeline is good at catching bugs. Edge cases, missing error handling, type mismatches, style violations. RAG makes it better — injecting architecture docs and translation conventions into the review prompt produces surprisingly domain-aware feedback.

What it can't do is notice that three merge requests this week are pulling the architecture in contradictory directions. It evaluates each MR in isolation. It doesn't see that a pattern emerging in the database layer contradicts a decision made in the service layer two months ago. That kind of judgment requires holding the whole system in your head. The model has a context window that resets every time.

The deeper problem: an LLM doesn't build understanding. It doesn't call someone and demand to know why they did it that way. No knowledge is kept from Monday's review into Tuesday's. It doesn't notice its own knowledge degrading, because it never had knowledge to begin with.

## What I Actually Do Now

The pipeline stays. Removing it would be stupid. But the process changed.

The AI review is the first pass, not the only pass. It filters out noise: style issues, obvious bugs, mechanical problems. What's left is the stuff that requires judgment. A human reviewer reads that subset, and because the noise is gone, they can focus on architecture, intent, and whether the change makes sense in the context of everything else.

The rule: if the AI approved it, a human still reads it. Not re-review — read. Understand what it does and why. If the reviewer can't explain the change without opening the diff again, it doesn't merge yet.

Slower than full automation. Faster than no automation. It draws complaints from some devs. Others seem to play a game of convincing the author that the AI was wrong not to flag line X. Competitive instinct kicked in — "we're better than your AI." They are, on the things that matter. And now they're reading the code to prove it.

Organizational consequences hit as well. For me? I decided to spend one day a week with no AI assistance: write that library myself. If I find I can't do it due to lack of context — no more access until I can. Frankly, I only got off easy. "Explain why module X in package Z does Y." I passed the test due to my known [zero trust workflow](/blog/multi-agent). But better to be on my guard for the process as well.

That Oracle/PL/SQL to PostgreSQL migration devs wanted to speed up with AI? Dead in the water. "Do it yourselves — if it breaks you'll know where to look." A very harsh reaction, but maybe necessary to avoid a repeat.

Personally, I do not agree. Rubber-stamping reviews is as old as coding itself. The incentive here was the combination of lack of time, high workload and, understandably, lack of willingness to question the first signs of workload lift the modernization gave them. The mistake wasn't in giving them access to AI. It was in not freeing up time to explore, compare and question. As it so very often is.

As for me? My mistake was not to watch far more closely for any signs of this. Focusing less on my analysis of the next steps, the five "battles" of what, how, when, with whom and why this way, and more on how this system was used by the devs would have avoided this.

## The Uncomfortable Part

The pressure is always toward full automation. More AI, less human, faster throughput. Every metric improves except the one nobody measures: how well does the team understand what they're shipping?

There's no dashboard for "percentage of the codebase that at least one person can explain from memory." So when that number drops, nothing triggers an alert. It shows up later as slower debugging, worse architectural decisions, and an increasing reliance on the AI to explain the team's own system back to them.

I caught it after a few months on a small team. And only because I make a point to routinely have direct discussions with the devs to hit home that their domain knowledge is still highly appreciated despite the rapid modernization. In a larger organization, with more code, more people trusting the pipeline and no pointed "I want your opinion on this, because I value it" culture, it would take longer to notice and be harder to reverse.

If at all.

---

*I run AI-assisted review on a codebase older than most of the engineers who'll maintain it. The model catches bugs I'd miss. It doesn't learn anything from catching them. If your review pipeline doesn't force someone to understand the code, you're building a system nobody can maintain.*
