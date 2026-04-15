---
layout: post
title: "Quantifying a Legacy Codebase You Can't Rewrite"
subtitle: "227 writes. Three semantic traps."
date: 2026-04-15
tags: [legacy-modernization, c, database, strangler-fig, migration]
permalink: /blog/quantifying-legacy/
---

I inherited a 40-year-old codebase.

The last team that tried a Big Bang rewrite failed.

My job was to migrate away from it. Incrementally, without breaking production, and then hand the plan to whoever comes after me. Before designing the new system, I needed to measure the old one.

## Why Measurement Comes First

Architecture diagrams are opinions. When you propose a Strangler Fig migration, the first question from management is always some variant of "Can't we just write it all new?"

If your answer is a box-and-arrow diagram with lots and lots of "see how x influences y and why we have to be careful"? Congratulations, you have lost the room.

If your answer is "We have around 230 write calls across four modules, distributed across two API generations, with a clean choke point, and no idea of the rest," you have a conversation.

That is the real reason. I did not understand this codebase. Nobody does. The people who wrote it are gone. The documentation, if it ever existed, is gone. What remains is the source and analysis. Interpret with caution. It does not lie, but it will not tell you anything unless you ask. You have to know the right questions.

## The Right Questions

For a database layer migration, the questions that matter are:

### 1. How many access paths exist?

How many distinct API surfaces are in use? Do not attach meaning to the number of files. In this codebase, the answer was three generations:

| Generation | Era | Role |
|---|---|---|
| Gen 1 | Late 1980s | Universal dispatcher with flag argument, used by ~23,000 scripts |
| Gen 2 | Mid 1990s | Thin wrappers over Gen 3, mostly internal |
| Gen 3 | 2000s | Current API, basis implementation |

Gen 3 is the foundation. Gen 2 wraps it. Gen 1 takes a separate path through a universal dispatcher. Any migration plan that ignores Gen 1 ignores roughly 400 call sites in the business logic alone.

### 2. What is the read/write ratio in the business logic?

This question determines your migration sequence. If the business logic is mostly read-only against the database, you can swap the database layer underneath with lower risk. If it writes heavily, you need to solve consistency between old and new paths before you can swap anything.

Answering it required understanding that the Gen 1 dispatcher is not one operation. It is every operation, selected by a flag:

```c
db_dispatch(handle, DB_WRITE, ...);    // write
db_dispatch(handle, DB_DELETE, ...);   // delete
db_dispatch(handle, DB_STEP, ...);     // read/iterate
db_dispatch(handle, DB_LOCK, ...);     // lock
db_dispatch(handle, DB_SEEK, ...);     // seek
```

A naive grep for function names misses this entirely. You need to extract the second argument:

```bash
grep -rh --include='*.c' -oE 'db_dispatch\([^,]+, *[A-Z_]+' \
  src/warehouse/ src/app/ src/finance/ src/core/ \
  | sed 's/db_dispatch([^,]*, *//' \
  | sort | uniq -c | sort -rn
```

Result:

| Operation | Calls | Type |
|---|---|---|
| DB_WRITE | ~195 | Write |
| DB_STEP | ~90 | Read |
| DB_DELETE | ~16 | Delete |
| DB_SEEK | ~25 | Read |
| DB_LOCK | ~14 | Lock |
| DB_READ | ~20 | Read |
| DB_INIT | ~10 | Read |
| Others | ~10 | Read |

Combined with the Gen 3 API read calls (~400) and a handful of Gen 3 deletes:

- **Total write calls: ~230**
- **Total read calls: ~550**

Around 230 writes to the database across four business logic modules. That number drives every decision downstream.

### 3. Where are the writes concentrated?

230 writes across four directories is not the same as 230 writes spread evenly. If one module accounts for 80% of writes, you migrate that module first. Distribution matters for sequencing.

### 4. What are the semantic traps?

Counting calls tells you the volume. It cannot tell you the danger. For that, you need to read the code, but you need to know where to read. Quantification narrows thousands of files to the handful that matter.

In this codebase, three semantic traps emerged from the analysis:

**NULL equals Zero.** Every numeric zero is written as database NULL. Every database NULL is read as zero. The system cannot distinguish between "value is zero" and "value is absent." Thousands of scripts depend on this behavior. Any replacement that treats NULL and zero differently will silently corrupt data.

**Optimistic locking via a Change ID field.** Before every update, the code reads a CID field, compares it to the value it read earlier, and increments it on write. If the new database layer does not replicate this exactly, concurrent writes from the old C path will silently overwrite changes from the new path. No error, no log, just data loss.

**Application-level triggers.** A small number of tables have triggers implemented in the scripting language, not in the database. They fire only through the C layer's universal entry point. If the new database layer writes to these tables directly, the triggers do not fire.

None of these show up in an architecture diagram. They only show up in the code, once you know where to look.

## The Uncomfortable Number

The write count in the business logic is ~230.

Even after migrating the scripting layer to a new database service (the first three phases of the Strangler Fig), the C business logic still writes to the database around 230 times through the old path. Two parallel write paths. For the entire duration of the business logic migration. For years, not months.

CID optimistic locking is the safety net during this period. It was designed for concurrent access between multiple C processes. A new database service is, technically, just another process. It works, but only if the new service implements CID identically.

That number turns "we will migrate the database layer" from a sentence into a project plan:

- How many call sites need individual migration in the business logic phase
- How long the dual-write period will last
- What the consistency mechanism must guarantee
- Why Big Bang is off the table (you cannot migrate 230 write sites atomically without tests, and there are no tests)

## What the Table Interface Layer Tells You

Below the business logic sits a table interface layer: roughly 300 files, around 80,000 lines of C. Sounds enormous. Quantification tells a different story.

95% of these files are mechanical: they map field names to struct pointers. This mapping already exists as runtime metadata in a binary descriptor format. A generic service that reads the metadata replaces the vast majority of these files without individual migration.

About 16 files contain actual logic: credit limit calculations, pricing queries, validation rules. Those need individual attention. 16 is a different conversation than 300.

Staring at 80,000 lines, I estimated months of work. After quantification, I was looking at 16 files and a metadata parser.

## The Tooling Question

When multiple languages access the same database (C, Python, Java), the schema lifecycle tool cannot be coupled to any one of them. ORM-driven migration tools are out. Schema is the source of truth, not the model. Migration files are SQL, not Python or Java. Anything that generates migrations from model diffs creates a coupling that breaks the moment a second language touches the database.

Someone who has only worked in single-stack environments will not see this. They will recommend what they have always used, and it will sound reasonable. The 230 C write calls that will coexist with the new service for years make the argument concrete: you cannot tie your schema lifecycle to one stack when three stacks write to the same tables.

## What You Leave Behind

Quantifying a codebase you cannot rewrite is not about aiming to rewrite it. It is about handing the next person a map instead of a blank page.

Around 230 write sites that need to migrate. Three semantic traps that will corrupt data if missed. Hundreds of files that can be replaced generically, a handful that cannot. A consistency mechanism that keeps the system safe during the transition, documented down to the field level.

How you get there depends on the team, the timeline, the budget, and organizational factors that no technical analysis can predict. But without the map, you are guessing. In a 40-year-old codebase with no tests and no documentation, guessing is how the last Big Bang rewrite failed.

Measure first.

---

*Working on legacy migration requires facts only. Once interpretation replaces factual analysis, you fail sooner or later.*
