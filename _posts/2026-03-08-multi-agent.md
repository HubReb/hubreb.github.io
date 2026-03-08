---
layout: post
title: "One Agent Per Domain, Zero Trust"
subtitle: "Why one should never use a single AI agent"
tags: [ai, multi-agent, legacy-migration]
---

Migrating a 30+ year old system from a proprietary scripting language to Python 3: thousands of scripts. Hundreds of libraries. Business logic that nobody documented because "everyone knows how it works." Except the people who knew have left.

I use frontier models for the development workflow, not as autocomplete. As agents with defined roles, domain context, and review authority over each other. The human stays in the loop — but only after the agents have argued it out first.

## The Problem With One Model

A single model doing everything — planning, implementing, reviewing — has no adversary. It approves its own assumptions, misses its own blind spots, generates code that looks correct, passes syntax checks, and implements the wrong business logic. We have all seen this enough times to stop trusting it.

The failure mode is *plausible* code. Code that a reviewer glances at and approves because the structure looks right. The bug hides in a convention the model doesn't know, a pattern it hasn't seen, a domain rule it was never told about.

One model can't catch its own mistakes. So give it colleagues.

## The Agents

Each part of the codebase needs its own expert agent. Not one generalist that "knows everything." Dedicated specialists, each loaded with the documentation and source for their domain.

For this system, that means: one agent for the Python 3 bridge architecture, one for the C runtime, one for the proprietary scripting language, and more as the migration touches other parts of the system.

Each agent reviews from its own perspective. Bridge agent catches pattern violations. C runtime agent catches assumptions about memory layout or API behavior. And the scripting language agent catches gaps between what the spec says and what the code actually does.

That last role matters most because it catches the biggest problem: **unbacked assumptions.** The frontier model makes assumptions constantly — in the planning phase, in the spec, in the implementation. "This function returns a list"... does it? "This error code means failure"... in every context? "This field is always populated"... are you sure?

Without a domain expert agent that can check claims against the actual source, assumptions survive into implementation. The scripting language agent doesn't just verify behavior. It forces the model to show its work.

The pattern scales. New subsystem? New agent. The cost is context curation, not framework configuration.

## The Workflow

Three phases, same review structure in each.

In **planning**, a frontier model (largest tier) takes the task, breaks it into steps, identifies dependencies and risks. Domain agents review the plan for architecture fit and unbacked assumptions. Loop until consensus, then a human reviews. Disagreement means feedback and another cycle.

In **specification**, the same model writes the technical spec: interfaces, types, test cases, edge cases. Same review loop.

In **implementation**, a smaller, faster model codes against the spec with TDD. Same review loop.

Every phase ends with a human — no exception. The agents reduce the noise. They catch pattern violations, architecture breaks, and spec-vs-reality mismatches before a human ever sees the code. Human review focuses on what AI still can't judge: is this the right thing to build?

So far: 17 libraries and 175 application scripts migrated this way.

## In Practice

Bridge agent catches layering violations: "You're calling the transport layer directly from the API. Go through the backend." C runtime agent catches dangerous assumptions: "You're assuming this function returns NULL on error. It doesn't. It returns -1 and sets a global." Scripting language agent catches the important ones: "The original code short-circuits on this error code. Your implementation raises an exception. That changes behavior for dozens of scripts that depend on silent failure."

What they don't catch: business logic that isn't in the code, institutional knowledge in someone's head, security implications. And recurring quality issues — DRY violations, SOLID principle breaks, tests that should be parametrized but aren't. The models know these principles. They still don't apply them consistently without explicit pressure. Agents catch the majority of what a senior reviewer would catch. What remains is why the human engineer must be in the loop.

## Why Not a Framework?

AgentOS, LangGraph, CrewAI, Google ADK. They optimize for orchestration: how agents communicate, how state flows, how you handle failure.

My bottleneck is knowledge, not orchestration. Does the agent understand this specific system well enough to review code for it? That's prompt engineering, document curation, and context window management. No framework solves that.

The workflow itself is simple: sequential phases, review loops, human gate. That's a for-loop, not a framework. The complexity lives in what each agent knows, not how they're wired together.

---

*Real teams don't have one expert for the whole codebase. Neither should your AI.*
