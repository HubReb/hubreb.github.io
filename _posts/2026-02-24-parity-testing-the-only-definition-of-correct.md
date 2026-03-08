---
layout: post
title: "Parity Testing: The Only Definition of Correct"
subtitle: Unit tests can't save you. Code review can't save you. Only the database knows the truth.
tags: [testing, legacy-migration, verification]
permalink: /blog/parity-testing
---

When you translate a script from a proprietary language to Python 3, how do you know it's correct?

Not "looks correct." Not "passes code review." Not "a senior engineer approved it." Actually correct. Functionally identical to the original. Every database write, every state change, every side effect, identical.

The answer is embarrassingly simple, and it took me too long to arrive at it.

## The Problem With Traditional Verification

**Unit tests** require understanding the expected behavior well enough to write assertions. For a legacy codebase with 30 years of implicit behaviors, undocumented side effects, and business logic baked into database triggers, you don't have that understanding. Writing unit tests for a translation means writing unit tests for a system you're still reverse-engineering.

And here's the part nobody talks about: the developers who wrote those scripts are gone. Retired, moved on, left the company years ago, or dead. Nobody in the building can tell you what that script is supposed to do. The documentation, if it ever existed, is either lost or wrong. You're translating code that nobody alive fully understands.

**Code review** means a human reads the translated Python and decides "yes, this does the same thing." For trivial scripts, maybe. For anything involving cursor operations, implicit commits, type coercions, and state mutations across multiple tables, no human can reliably verify that by reading code. Especially not when the reviewer doesn't know what the original was supposed to do in the first place.

**Integration tests** get closer but still require someone to define what "correct" means for each script. That's the entire problem. The original system *is* the definition of correct. Any test that doesn't compare against it is an opinion.

## The Parity Test

The approach is:

1. Run the original script against a test database. Snapshot the database state after.
2. Reset the database.
3. Run the translated Python script against the same database. Snapshot the database state after.
4. Compare the two snapshots.

Identical state = correct translation. Different state = bug. No ambiguity.

That's it. No test framework, no assertion library, no expected-value definitions. The original script *is* the test. The database *is* the oracle.

## Why This Works for Legacy Migration

Legacy systems are defined by their behavior, not by their documentation. Nobody wrote a spec for what the script is supposed to do. The script *does* what it does, and the business runs on that behavior. Any translation that changes that behavior is wrong, even if the change is "better."

This is counterintuitive for engineers who think in terms of improvement. The Python version should be cleaner, more efficient, more maintainable. And it will be. But first, it has to be *identical*. Refactoring comes after parity, not during.

We have all seen translations that "improved" the original by handling edge cases more gracefully. They failed the parity test. The original's edge case behavior was the expected behavior. The improvement was a bug.

## The Setup

**Database snapshots** need to capture everything the script touches: tables, sequences, metadata. Not a full database dump (too slow). Targeted snapshots of the tables the script writes to.

Which means you need to know which tables the script writes to. For a proprietary language where database operations might be wrapped in abstraction layers, buried in library calls, or triggered implicitly, that's non-trivial analysis. The parity test is simple. The setup is not.

**Isolation** is critical. Both runs hit the same database in the same state. No concurrent processes, no background jobs, no triggers firing from external events. The only difference between run A and run B is the language the script is written in.

**Determinism** matters. If the script generates timestamps, random IDs, or sequence values, the outputs will differ even if the logic is identical. These need to be controlled: fixed timestamps, seeded randoms, pre-set sequences. Any source of non-determinism is a false negative waiting to happen.

## What the Parity Test Catches

**Semantic translation errors.** A construct that looks like a loop but behaves like a cursor. The output is different, the parity test catches it.

**Implicit behaviors.** The original language auto-commits after certain operations. Python doesn't. Missing commit = different state.

**Type coercions.** The original silently converts strings to numbers in comparisons. Python doesn't. Different comparison result, different branch taken, different writes. The classic: the original treats 0 and NULL as equivalent. Python distinguishes between `0`, `None`, and `False`. One missed coercion, one wrong branch, one corrupted record.

**Order of operations.** The original processes records in insertion order. The Python version uses a dict, which until 3.7 didn't guarantee order. Different processing order, different final state if operations aren't commutative.

All of these are invisible in code review. All of them are obvious in a state comparison.

## What It Doesn't Catch

**Performance.** A functionally identical script that takes 10x longer passes the parity test. Performance is a separate concern.

**Non-database side effects.** File writes, API calls, log entries. The parity test only sees the database. Other side effects need separate verification.

**Partial correctness.** If the original script has a bug, the translation must reproduce the bug. The parity test doesn't distinguish between features and bugs. That's a feature.

## The Uncomfortable Truth

Parity testing means accepting that the legacy system is correct by definition. Not correct as in "well-designed." Correct as in "this is what the business runs on, and changing it is a business decision, not an engineering decision."

Every engineer wants to fix the bugs they find during migration. Don't. Translate first. Prove parity. Then open a ticket for the bug and fix it in the Python version with a proper change request. Mixing migration and improvement is how legacy migrations fail.

## Takeaway

If you're migrating legacy code with AI assistance, parity testing is not optional. It's the only verification that doesn't depend on human understanding of a system that humans have been failing to understand for decades.

The database doesn't care how elegant the Python is. It cares whether the state is identical. Start there.

---

*I migrate decades-old systems to modern stacks — one script, one table, one timestamp at a time. I validate translations by asking the only witness that doesn't lie: the database. More on AI-assisted migration at [hubreb.github.io](https://hubreb.github.io/).*
