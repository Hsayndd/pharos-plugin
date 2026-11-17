# How Pharos works

Pharos is a local engine that Claude Code talks to through MCP tools and hooks. This page explains what it builds, how it answers, and where its answers stop. It covers the free Core; Pharos Pro is a separate, paid edition built on the same engine.

## The pieces

| Piece | What it is |
|---|---|
| `pharos-core` | One binary: the indexer, the query and plan engine, and the MCP server Claude Code starts |
| `pharos-claude-code` | The hook adapter: reads Claude Code's hook events and answers in Claude Code's format |
| `.pharos/` in each repository | The index (`index.db`, SQLite), memory, a result cache and, if you turn it on, local telemetry |
| Skills | Seven `/pharos:*` commands, plain instructions to Claude that use the tools below |

Both binaries are written in Rust and built with no network code at all.

## 1. The index

Pharos lists every file Git tracks or would track, minus ignored files, and indexes each one at one of two tiers:

- **Grammar tier,** for TypeScript, JavaScript, Python, Rust, C, C++ and CUDA. A tree-sitter grammar parses the file. Pharos records its declarations (functions, methods, classes, types, constants) and the edges between them: calls, imports and includes, references to types, inheritance and overrides, and CUDA `<<<…>>>` and Triton `kernel[grid]` launches.
- **Heuristic tier,** for every other text file. Declarations are found by pattern. No calls and no uses are recorded.

No file is silently left out: `index.status` reports the count at each tier, and a file too large to read is listed as skipped.

**Resolving a call.** A call is linked to a declaration through the file's imports and qualified names first. When only the name matches, the link is kept but marked name-only, and reports list it apart. No language server is used.

**Deterministic.** The same repository state produces a byte-identical index. Nothing in it depends on time, randomness or hash ordering.

**Incremental.** After the first index, only files whose content changed are read again. A tool call re-indexes first when the index is more than 30 seconds old (configurable), so answers follow your edits.

## 2. Structural questions: `pharos.query`

One tool call answers one question, or several in a batch: `definition`, `references`, `callers`, `callees`, `implementors` or `changed_since`. Each answer gives the exact location, the declaration that contains it, and the line itself, so Claude can often answer without opening the file. Results are cached by content hash, so the same question on an unchanged repository costs nothing to compute again.

## 3. Context plans: `context.plan`

Given a task and a token budget, Pharos chooses the code most relevant to it:

1. **Retrieve.** Candidates come from two rankings: a lexical match of the task's words against names, paths and documentation, and the graph, which follows edges out from the strongest matches. The two are fused by reciprocal rank fusion.
2. **Pack.** Each candidate has a value and a token cost. Pharos chooses the set with the most value that fits the budget, a knapsack problem.
3. **Compact.** A declaration can be sent in full, or as its signature with the body left out, which costs far fewer tokens. Documentation comments are kept.

**The budget is a contract.** A plan never contains more tokens than the budget, as counted by a conservative estimator that errs on the high side. `/pharos:context` shows every item of a plan, why it was chosen and what it costs.

## 4. Memory

`memory.record` stores a decision, convention, open question or note, and requires the commit it applies to. `memory.recall` finds entries by text. Before an entry is returned, Pharos checks it against Git and marks it stale, with the reason, if its commit no longer resolves, is not an ancestor of the current commit, or the files it is anchored to have changed since.

## 5. Hooks

| When | What Pharos does |
|---|---|
| Before a shell command | Stops a short list of destructive patterns with a message |
| Before a read or search | After many raw reads in a session, suggests the one query that answers it. Suggests only; never blocks |
| When you send a prompt | Adds a short map of the repository to the context |
| After a Pharos tool call | Counts it for `/pharos:impact` |
| After an edit, at the end of a turn | Runs a format or check recipe, only if you configured and trusted one |

Hooks are silent unless they have something to say, and are kept fast: the read check has a 50 ms budget.

## 6. The tools Claude sees

Six MCP tools, served by two servers so the common ones load first:

| Server | Tools |
|---|---|
| `pharos`, always loaded | `pharos.query`, `context.plan` |
| `pharos-more`, loaded when needed | `index.status`, `memory.record`, `memory.recall`, `session.impact` |

Every input schema is strict: an argument a tool doesn't take is refused with a clear message, never silently ignored. A malformed request gets a JSON-RPC error, and the server keeps running.

## Limits

An answer lists the uses Pharos can see. These it cannot:

- **Heuristic-tier files:** no calls or uses at all.
- **JSX:** a React component rendered as an element, such as `<Price />`.
- **Re-exports:** a call through a file that only re-exports a name is linked by name only.
- **Uses by parameter name:** dependency injection, pytest fixtures.
- **Macros:** a call inside a Rust macro's arguments (`ready!(read_line(reader))`), or inside a C or C++ `#define` body. Items declared inside a Rust item macro such as `cfg_rt! { … }` are indexed.
- **Dynamic code:** `getattr`, computed names, function pointers in tables, reflection, code generated at build time.
- **Other repositories.**

So a short list is not proof that a change is safe. Keep testing as usual.

## Privacy

- **Local processing:** the binaries make no network call, and everything Pharos writes stays in the repository's `.pharos` folder and your home folder.
- **What reaches the model:** Pharos's answers are tool results. Claude Code sends them to the model like any other tool output, so code excerpts, paths and names that Claude asks for leave your machine that way.
- **Telemetry** is off by default. When on, it holds counts only, never code, and stays in `.pharos`.

## Source and licence

Pharos Core is Apache-2.0. This repository holds the built plugin: the binaries, their third-party notices, the hooks and the skills.
