---
layout: post
title: "The Cards Are Always Face Up"
subtitle: "Spec-driven development promised to move all decisions up front and leave only implementation. The failure mode looks familiar, minus the part where you notice."
tags: [ai-assisted-development, architecture, spec-driven-development]
permalink: /blog/cards-always-face-up
---

Some weeks ago, I ran a spec-kit pipeline on a modernisation slice. The spec described what I needed: modernising a three-year-old personal CLI of mine, an offline paper-database searcher with a PostgreSQL backend, around 2000 lines of Python. The pipeline produced Alembic.

I was never asked to choose. The pipeline simply decided: we use Alembic. It assumed I had an ORM, that migrations would live in Python alongside the application code, that the schema-evolution model would be generated from model diffs. None of it held. The code was raw psycopg2 with no ORM and no migrations at all; the schema came from a `create_tables()` call issuing `CREATE TABLE IF NOT EXISTS` at runtime.

Spec-kit produced Alembic as if migrations meant Alembic. The decision happened below the layer of the spec.

## The defaults problem

Spec-driven development builds on one idea: you think on the spec layer, make architectural choices explicit in the spec. Implementation is execution against it, done by an LLM.

In practice, that idea doesn't hold. 'Continuous architecture' exists for a reason. We tell spec-kit "Modernise this application." It produces Alembic. The architectural decision is now wearing the hat of an implementation following from the requirement.

These are not really defaults in the traditional sense, where a default is what you get if you don't specify. They are hidden architectural decisions presented as inevitable consequences of the spec.

My input, verbatim in the [spec file](https://github.com/HubReb/pdb_search/blob/main/specs/001-modernize-stack/spec.md): "Reverse engineer this code base. Modernize it to main stream frameworks." Out of that the pipeline wrote FR-004 — database access MUST go through a mainstream ORM, replacing hand-written SQL — and FR-005, the same for a migration tool. Then the [research document](https://github.com/HubReb/pdb_search/blob/main/specs/001-modernize-stack/research.md) rejects raw SQL because it would not satisfy FR-004, and picks Alembic citing FR-005. The constraint checks ran, and they ran correctly, against requirements the same hand had written two steps earlier.

'Mainstream frameworks' is my wording. 'MUST use an ORM' is not. Everything between those two sentences is the architectural decision, already made, wearing a requirement's clothes. And a MUST in the spec is no longer a face-up card you can argue against. Arguing against it is a spec violation.

The obvious answer is to specify the decision. But the decision you would have to write down is not visible when you write the spec. The work has not surfaced it yet. That is what makes it a gap rather than an omission.

You can, of course, attempt to foresee every architectural decision before the work has surfaced them. Or write a detailed prompt from the very beginning. Or watch like a hawk, interfere and tell spec-kit to use something else.

Now, continuous architecture has long told us of the antidote: decide architecture as late as possible. Once you spec, you have foreseen all decisions in advance.

Except that now, instead of a dev pinging you with "what should be done", the LLM gets to decide for you, because "that is how it is done in the training data, and so it is done here as well".

We have successfully elevated the decision layer. The design went down with it.

## Too small to spec

The pattern doesn't stop at the decisions you would think to write down. It runs through the ones you wouldn't.

Python 3 is the clearest case, because the same model in the same context changes its mind across releases. One iteration prefers Mixins over composition, tight coupling finally making its comeback. Another decides configuration objects are superfluous nonsense, a method with 20+ arguments being a paragon of simplicity.

Each of those is a real architectural decision that depends on context: MRO complexity, runtime configurability, test isolation. Each preferred option is reasonable in some context. No pattern is universally right, and the careful consideration of trade-offs is gone.

The preference drifted while the codebase and the spec stayed where they were. A decision that follows from the spec does not drift when the model is swapped underneath it, so this one was never following from the spec.

The LLM produces the majority option: the most plausible one according to its training data. The context that would justify a different option doesn't surface, and neither does the context that produces the majority option. Context and preferences via CLAUDE.md can help. But context is limited, and even careful context engineering cannot always avoid the most probable according to training data.

The majority conventions within the training data cannot mirror the codebase: enterprise engineering best practices meets ML best practices meets Django best practices.

## Planning poker, inverted

There's a reason planning poker is used for estimation. Once a number is in the room, every subsequent estimate anchors to it. Hence, the cards stay face down until everyone has committed.

Spec-kit is the structural opposite of planning poker: only face-up cards. The question shifts from "what is the right migration tool for this context?" to "why not Alembic?". We now face a different question, higher cognitive load due to first-mover advantage, a worse anchor point, and fairness bias toward what's already been suggested.

This isn't fixable by audit logging or transparency. FR-004 was written down, reasoned, and checked against. The decision is made within the pipeline. You audit instead of choosing.

Rejecting the PR is a surface answer. The decisions are made, the ticket has moved to 'in review', the reviewer is under pressure, and 'back to square one' is politically expensive.

Asking an LLM directly has the same failure mode in milder form. You ask "which migration tool should I use for Python?" You get Alembic, possibly with caveats. The card is face up. Again, you're now arguing against an anchor rather than evaluating cold.

## What I cannot claim

I cannot say Alembic was the wrong call.

It runs. For a single-stack personal CLI, SQLAlchemy with Alembic on top is a defensible modernisation, and I might well have picked it myself.

The claim is narrower: it was not chosen. Whether the majority option fits is a separate question from whether anyone decided. FR-004 exists. R1 checks against it and rejects raw SQL. At no point in that chain did a constraint and an option get weighed against each other.

That is also why reading the output does not catch it. Wrong output shows up sooner or later. An unmade decision that lands right leaves nothing to find, and the next one gets made the same way.

## Deciding late

Again, continuous architecture preaches to decide at the last possible moment: the alternative is deciding before the work has surfaced what the decision is about.

The new part is that a tool will fill that gap and you might not notice it did.

Spec-driven is perhaps one of the best ways to work with the LLM agents, yet every spec you write is a card you turn over. And this time there is no annoying dev to hit the brakes. The tool decides on the gaps.

---

*No choice is made. The majority in the data arrives wearing a decision's clothes.*
