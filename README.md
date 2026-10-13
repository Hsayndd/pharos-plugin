# Pharos for Claude Code (beta)

Pharos gives Claude a map of your repository: a symbol graph built on your machine, one-call structural questions (where is it defined, who calls it, what implements it), and context plans packed into a token budget you choose. Nothing leaves your machine.

**Status: beta.**
- The binaries are not code-signed yet.
- Token savings have not been measured yet.
- TypeScript, JavaScript, Python and Rust are parsed in full; other files are indexed more coarsely.

## Install

You need Claude Code 2.1.292 or later. Nothing else: no Node, no Python, no runtime.

In Claude Code, run the line for your system:

| System | Command |
|---|---|
| macOS or Linux | `/plugin install pharos --marketplace Hsayndd/pharos-plugin` |
| Windows | `/plugin install pharos --marketplace Hsayndd/pharos-plugin#windows` |

Choose a scope when asked: "Install for you" makes Pharos available in every project.

### If the install times out

Claude Code gives the download two minutes. Pharos is about 6 MB on Windows and 24 MB on macOS and Linux, so a slow connection can run out of time. The install then fails with "Git clone timed out", or with `Permission denied (publickey)`.

Allow ten minutes instead, restart Claude Code, and run the install again:

| System | Command |
|---|---|
| macOS or Linux | `export CLAUDE_CODE_PLUGIN_GIT_TIMEOUT_MS=600000` in the terminal you start Claude Code from |
| Windows | `setx CLAUDE_CODE_PLUGIN_GIT_TIMEOUT_MS 600000`, then restart Claude Code or VS Code |

## First run

1. Open a repository in Claude Code.
2. Ask: *"Use index.status to index this repository."*
3. Pharos indexes it into a `.pharos` folder there, which Git ignores. Later questions keep the index up to date on their own.

Then try:
- *"Use pharos.query to list every caller of `parseConfig`."*
- `/pharos:context fix the cache invalidation bug, budget 6000`: shows what a plan would send, and why.
- `/pharos:plan add rate limiting to the API, budget 12000`: a plan grounded in the code, with no edits.
- `/pharos:remember decision: public API errors use ErrorEnvelope`: a note tied to the current commit.

## Privacy

Pharos makes no network calls. Telemetry is off by default; when you turn it on, it stays in your repository's `.pharos` folder and holds counts only, never code.

## Feedback

Report problems and ideas on [GitHub Issues](https://github.com/Hsayndd/pharos-plugin/issues).

## Licence

Pharos Core is Apache-2.0 (`LICENSE`). The binaries include open-source components listed in `engine/THIRD_PARTY_NOTICES.md`.
