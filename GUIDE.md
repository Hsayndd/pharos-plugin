# Pharos Core guide

How to install the free Pharos plugin for Claude Code, check it, and use it. This guide is for **Core 1.1.2 on Windows x64**. Where the macOS and Linux beta (0.2.0-beta.2) differs, it says so.

- [Install](#install)
- [Check the install](#check-the-install)
- [First run](#first-run)
- [Asking structural questions](#asking-structural-questions)
- [Context plans](#context-plans)
- [Memory](#memory)
- [Skills](#skills)
- [Hooks](#hooks)
- [Settings](#settings)
- [From a terminal](#from-a-terminal)
- [If the install fails](#if-the-install-fails)
- [Update, pause, uninstall](#update-pause-uninstall)

## Install

**Requirements:** Claude Code 2.1.292 or later, and Git. Nothing else.

**Windows x64,** in Claude Code:

```
/plugin marketplace add Hsayndd/pharos-plugin#windows
/plugin install pharos@pharos
```

**macOS and Linux** (the 0.2.0-beta.2 build):

```
/plugin marketplace add Hsayndd/pharos-plugin
/plugin install pharos@pharos
```

Choose a scope when asked: "Install for you" makes Pharos available in every project. Restart Claude Code when the install finishes.

**macOS only:** the binaries are not code-signed. If macOS blocks the engine, clear the quarantine attribute from the Pharos plugin folder only, once:

```
xattr -dr com.apple.quarantine ~/.claude/plugins/cache/pharos
```

**Offline install (Windows):** download `pharos-plugin-1.1.2-18891f388e21-windows.tar.gz` and `SHA256SUMS-windows` from the [1.1.2 release](https://github.com/Hsayndd/pharos-plugin/releases/tag/v1.1.2-windows). Check the archive in PowerShell:

```
(Get-FileHash .\pharos-plugin-1.1.2-18891f388e21-windows.tar.gz -Algorithm SHA256).Hash
```

It must equal `4b3e440d1416df37acc823877a1e03e6f72b89985725d957b7a61a0deedd503c`, the value in `SHA256SUMS-windows`, whatever the letter case. Then extract it and add the extracted `pharos` folder as a local marketplace: `/plugin marketplace add C:\path\to\pharos`, then `/plugin install pharos@pharos`.

## Check the install

1. In Claude Code, run `/plugin` and open the installed plugins: `pharos@pharos` shows version **1.1.2** (beta: 0.2.0-beta.2), enabled.
2. In a terminal, the engine prints its version and the commit it was built from:

   ```
   & "$env:USERPROFILE\.claude\plugins\cache\pharos\pharos\1.1.2\engine\pharos-core.exe" --version
   ```

   It prints `pharos-core 1.1.2 (18891f388e219c5e42b91f7ccf11f7e3866eec49)`.
3. In Claude Code, run `/mcp`: the plugin's two Pharos servers, `pharos` and `pharos-more`, are connected.

## First run

1. Open a repository in Claude Code.
2. Ask: *"Use index.status to index this repository."*

   Pharos writes its index to `.pharos/` in the repository and adds it to Git's ignore list. The answer lists how many files were indexed, by language and by tier. On a large repository the first index takes from seconds to a few minutes; later ones read only what changed.
3. Ask as you normally would. Claude calls Pharos when a question is structural, and you can also name the tool, as below.

**Read the tiers.** Files at the `grammar` tier have their calls and uses recorded. Files at the `heuristic` tier have their declarations only. If the code you care about is mostly heuristic, Pharos will help little there.

## Asking structural questions

`pharos.query` answers six kinds of question, each in one call, with the file, line and the line itself:

| Ask | Kind |
|---|---|
| *"Where is `parseConfig` defined?"* | `definition` |
| *"Use pharos.query to list every caller of `parseConfig`."* | `callers` |
| *"What does `handleRequest` call?"* | `callees` |
| *"Where is the type `UserId` used?"* | `references` |
| *"What implements the `Storage` interface?"* | `implementors` |
| *"What changed since `main`?"* | `changed_since` |

- **Several at once:** *"In one pharos.query batch, list the callers of `save` and the implementors of `Store`."*
- **Two declarations with the same name:** name the file: *"callers of `format` in `src/money.ts`"*.
- **A long list** is cut at a limit and says so; ask for a higher limit when you need all of it.

An answer lists the uses Pharos can see. It is not proof that nothing else uses the code: see the limits in [HOW_IT_WORKS.md](HOW_IT_WORKS.md#limits).

## Context plans

`context.plan` returns the code most relevant to a task, packed into a token budget you choose. It never returns more than the budget.

- *"Use context.plan for: add retries to the HTTP client, budget 6000."*
- `/pharos:context fix the cache invalidation bug, budget 6000` explains a plan: each item, why it was chosen, its view (full or signatures) and its cost.
- `/pharos:plan add rate limiting to the API, budget 12000` writes an implementation plan grounded in the code, with no edits.

## Memory

Pharos keeps decisions, conventions and open questions per repository, each tied to the commit it applies to.

- `/pharos:remember decision: public API errors use ErrorEnvelope` records one at the current commit.
- `/pharos:remember error envelope` recalls the matching entries.
- An entry whose commit no longer matches the code is marked stale when recalled.

Memory stays on your machine, in the repository's `.pharos` folder.

## Skills

| Skill | What it does |
|---|---|
| `/pharos:context <task>, budget <n>` | Explains a context plan item by item |
| `/pharos:plan <task>, budget <n>` | A plan grounded in the code, with no edits |
| `/pharos:remember …` | Records or recalls memory |
| `/pharos:impact` | What Pharos did in this session: tool calls, cache hits, suggestions made and followed |
| `/pharos:eval` | Checks evaluation readiness from this session's data |
| `/pharos:pause` | Pharos does no work in this repository until resumed; the index stays (1.1.2 only) |
| `/pharos:resume` | Pharos works again (1.1.2 only) |

## Hooks

Pharos adds a few hooks to Claude Code. They are silent unless they have something to say.

- **Before a shell command:** a short list of destructive patterns (`sudo`, a forced `git push`, `rm` of a root path, `curl … | sh`) is stopped with a message. It is a safety net, not a sandbox: your own permission rules still apply.
- **Before reads and searches:** after many raw reads in a session, Pharos suggests the one query that answers the question. It only suggests; it never blocks a read.
- **When you send a prompt:** a short map of the repository is added to the context.
- **After Pharos tool calls:** counts for `/pharos:impact`.
- **After edits and at the end of a turn:** a format or check recipe runs only if you configured one and trusted it.

## Settings

Optional, in `.claude/pharos.toml` in the repository. Everything has a default.

```toml
[rank]
budget = 24000         # default context.plan budget, in tokens

[query]
limit = 50             # answers per question before the list is cut

[hooks]
read_budget = 25       # raw reads before Pharos suggests a query

[index]
refresh_seconds = 30   # how old the index may get before a tool call re-indexes

[telemetry]
enabled = false        # local counts only, never code
```

## From a terminal

The engine also runs by itself, for scripts and other agents. On Windows it is `pharos-core.exe` in the plugin's `engine` folder ([Check the install](#check-the-install) shows the path).

```
pharos-core index .
pharos-core status .
pharos-core query callers parseConfig --repo .
pharos-core query definition parseConfig --repo . --path src/config.ts
pharos-core plan --repo . --task "add retries to the HTTP client" --budget 4000 --format text
```

`query` prints JSON: each answer has a location, the containing symbol and an excerpt.

## If the install fails

- **"Git clone timed out", or `Permission denied (publickey)` after a long wait:** Claude Code gives the download two minutes. Allow ten instead, restart Claude Code, and install again:

  | System | Command |
  |---|---|
  | Windows | `setx CLAUDE_CODE_PLUGIN_GIT_TIMEOUT_MS 600000`, then restart Claude Code or VS Code |
  | macOS or Linux | `export CLAUDE_CODE_PLUGIN_GIT_TIMEOUT_MS=600000` in the terminal you start Claude Code from |

- **`Permission denied (publickey)`, `Authentication failed` or `Repository not found` at once:** Git could not reach or authenticate to GitHub. Check that `git ls-remote https://github.com/Hsayndd/pharos-plugin.git` works in a terminal, fix your Git credentials or proxy, then install again.
- **The `pharos` server is not connected** (`/mcp`): restart Claude Code. If it stays disconnected, [open an issue](https://github.com/Hsayndd/pharos-plugin/issues/new/choose) with the version and the server's log line.

## Update, pause, uninstall

- **Update:** `/plugin marketplace update pharos`, then restart Claude Code. The index is rebuilt automatically when a release changes its format; memory is kept.
- **Pause in one repository:** `/pharos:pause`. The hooks pass through and the tools do no work; `/pharos:resume` turns it back on.
- **Uninstall:** `/plugin uninstall pharos@pharos`. To remove its data from a repository, delete that repository's `.pharos` folder.
