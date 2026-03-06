---
layout: post
title: "When You Can't Embed, Bridge"
subtitle: "How a PTY and a marker protocol replaced tens of thousands of lines of C++ bindings."
tags: [python, c, legacy-systems, ipc, migration]
---

A 30+ year old ERP system. The core is C. The scripting layer is a proprietary language: thousands of business scripts, a custom interpreter embedded in the C process via tens of thousands of lines of C++ bindings.

The migration target: Python 3. The obvious path: port the bindings. `PyString_FromString` → `PyUnicode_FromString`. `PyInt_AsLong` → `PyLong_AsLong`. `Py_InitModule` → `PyModule_Create`. Across all of them. No tests. No resources.

I didn't rewrite the bindings. I went around them.

## Why the Bindings Are the Wrong Target

The old interpreter is *embedded*. It runs inside the C process, shares its address space, calls C functions through direct pointer access. Every binding assumes Python 2 memory layout and API conventions. One wrong refcount, one missed API change, and you get silent memory corruption in production.

But the scripts don't need any of that. They don't do pointer arithmetic. They don't manage memory. What they actually do: open a table handle, read records, run queries, insert, update, delete. Begin a transaction, commit or rollback. Check error state. Database operations and control flow.

All of that can cross a process boundary. The bindings solve a problem the scripts don't have.

So: Python 3 as a separate process. The old interpreter stays inside the C runtime with its working bindings. Python 3 talks to it through a pseudo-terminal, using a marker-based protocol I defined for structured communication.

Three layers: an API layer that matches the old scripting interface, a backend that translates each call into proprietary scripting code and parses responses, and a PTY transport that handles the subprocess. The C++ bindings stay untouched.

## Everything That Went Wrong

**The PTY fights you.** A PTY fakes a terminal. Terminals echo input. Every command I sent came back mixed into the response. The legacy interpreter also prints initialization output on startup, which sat in the buffer and contaminated the first command's response. Intermittently. Two weeks of debugging for a one-line fix: drain the buffer after startup, before sending anything.

**Bidirectional streaming doesn't work.** The constant temptation: send commands while still reading responses. Pipeline operations. Every attempt made things worse. A PTY is a single channel. Interleaving reads and writes creates race conditions that are impossible to reproduce. The answer was boring: send, wait, read, repeat. Sequential. Slow. Reliable.

**Subprocesses die.** Not a one-time fix. An ongoing reality. The legacy interpreter crashes, hangs, or silently stops responding. The transport layer needs heartbeat detection, process recovery, and clean restart. This isn't solved and shelved. It's maintained.

**Not everything crosses a process boundary.** The legacy UI framework owns the terminal and fought the bridge for control. Three quarters of all scripts don't touch UI and work fine. The rest are deferred.

The old interpreter also keeps cursor state between calls: select opens a cursor, the next select continues where it left off. Across a process boundary, no shared state. Replicating it would turn the bridge into a chatty interface the PTY was never designed for. Instead: build a repository and gateway layer on the Python 3 side. Some legacy patterns need a Rosetta Stone, not a mirror.

## The Marker Protocol

The legacy interpreter doesn't speak JSON. It prints unstructured text for human eyes. So I built a protocol on top: the backend generates scripting code that wraps every operation's output in markers.

```
<<RC>>0
<<COLS>>('name', 'price', 'stock')
<<ROW>>(b'Widget\x00', b'29.99\x00', b'142\x00')
<<END>>
```

Return code, column metadata, row data, end-of-response. Error cases get separate markers propagating the C runtime's own error codes and messages. The parser reads byte by byte until end marker or timeout. Text-based, not binary. When something breaks at 2 AM, you can read the raw PTY output. Debuggability over efficiency.

## What This Buys You

**Isolation.** Python 3 crashes don't affect the C runtime. Segfaults don't corrupt Python state. The bridge itself is testable with mocked backends and pytest. The system went from zero tests to thousands.

**Incremental migration.** Old scripts stay on the embedded interpreter. New scripts run on Python 3 via the bridge. Both coexist. Core libraries migrate manually with full test coverage. The remaining hundreds get mechanical translation. Scripts migrate in bulk with output-diff validation.

**The bindings become irrelevant.** Not today. The bridge still needs the legacy interpreter. But once all scripts run on Python 3, the embedded interpreter and all its C++ glue can be removed entirely. The bridge is the path to a clean Python 3 ↔ C interface, built later, on our terms, with tests.

## What This Demonstrates

**Don't rewrite the glue. Route around it.** Working C++ bindings are a liability if you touch them and an asset if you don't. The bridge treats the old interpreter as an execution engine, not an integration target.

**Process boundaries are an architecture tool.** The overhead is real (milliseconds per call instead of nanoseconds). For the typical business script, it's invisible. For tight loops, a repository and gateway layer on the Python 3 side bypasses the bridge entirely.

**Every interprocess bridge looks simple on the whiteboard.** Then the PTY echoes your input. The startup buffer poisons your first command. The UI framework fights you for the terminal. Subprocesses die and you need heartbeat recovery. Legacy cursor patterns don't survive process boundaries and need a translation layer instead of replication. The architecture was the easy part. The edge cases were the work.

---

*I migrate decades-old systems to modern stacks. The rewrite everyone wanted would have taken months and risked production. The bridge nobody believed in is running. If your team is split between "rewrite it properly" and "find another way," I've been on the other side of that argument. Let's talk.*
