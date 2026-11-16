# Pharos for Claude Code

Pharos gives Claude a map of your repository: a symbol graph built on your machine, one-call structural questions (where is it defined, who calls it, what implements it), and context plans packed into a token budget you choose. Pharos itself makes no network call; what Claude asks it for, Claude receives as tool results ([Privacy](#privacy)).

**Version 1.1.2.**
- **Measured.** These come from paired experiments with Claude Sonnet 5.5 on frozen builds of this plugin (pilot 3 from `ea75861`, pilot 4 from `2a166b7`): the same questions with Pharos on and off. Each figure is the mean of paired per-task changes, with its 95 % interval over tasks: a range for the average, not a guaranteed saving.
  - **Medium TypeScript, Rust and Python repositories** (117,000 to 188,000 lines):
    - input tokens −14.6 % (−26.4 % to −0.1 %);
    - tool calls −30.4 % (−41.8 % to −17.1 %);
    - wall-clock −23.3 % (−37.1 % to −6.6 %), with indexing done beforehand.
  - **llama.cpp** (877,000 lines), on structural C, C++ and CUDA questions, a pre-registered secondary slice of 6 tasks:
    - input tokens −46.3 % (−64.5 % to −28.6 %);
    - tool calls −52.3 % (−71.1 % to −31.9 %).

    Over all 10 tasks of that experiment, the token change was not statistically significant (−21.5 %, −45.0 % to +2.5 %).
  - **Tokens, not dollars.** Input tokens include cached tokens, which are priced differently, so these are not cost savings.
  - **Answers.** In both experiments, every answer named every expected symbol, with and without Pharos. That checks that the expected names appear; it does not measure complete correctness.
  - **Where it costs more.** In a small repository, or for a question one search answers, Pharos can cost more than it saves: it adds a fixed overhead to each turn.
- **Languages.** TypeScript, JavaScript, Python, Rust, C, C++ and CUDA are parsed in full, including CUDA `<<<…>>>` and Triton `kernel[grid]` launches. Every other file, Go, Java and C# among them, is indexed by pattern only: its declarations are found, its calls are not. No language server is used. A React component rendered as JSX (`<Price />`) has no recorded use.
- **Unsigned binaries.** The binaries are not code-signed. If macOS blocks the engine, clear the quarantine attribute from the Pharos plugin folder only, once: `xattr -dr com.apple.quarantine ~/.claude/plugins/cache/pharos`.

## Install

You need Claude Code 2.1.292 or later. Nothing else: no Node, no Python, no runtime.

In Claude Code, run the line for your system:

| System | Command |
|---|---|
| macOS or Linux | `/plugin install pharos --marketplace Hsayndd/pharos-plugin` |
| Windows | `/plugin install pharos --marketplace Hsayndd/pharos-plugin#windows` |

Choose a scope when asked: "Install for you" makes Pharos available in every project.

### If the install times out

Claude Code gives the download two minutes. Pharos is about 6 MB on Windows and 24 MB on macOS and Linux, so a slow connection can run out of time. The install then fails with "Git clone timed out".

Allow ten minutes instead, restart Claude Code, and run the install again:

| System | Command |
|---|---|
| macOS or Linux | `export CLAUDE_CODE_PLUGIN_GIT_TIMEOUT_MS=600000` in the terminal you start Claude Code from |
| Windows | `setx CLAUDE_CODE_PLUGIN_GIT_TIMEOUT_MS 600000`, then restart Claude Code or VS Code |

### If Git cannot reach GitHub

`Permission denied (publickey)`, `Authentication failed` or `Repository not found` is not a timeout: Git could not authenticate to GitHub, or could not reach it. Check that `git ls-remote https://github.com/Hsayndd/pharos-plugin.git` works in a terminal, fix your Git credentials or proxy, then install again.

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

- **Local processing.** Pharos's binaries make no network call. Indexes, memory and settings stay in your repository's `.pharos` folder and your home folder.
- **What reaches Claude.** Pharos's answers are tool results: Claude Code sends them to the model like any other tool output. They hold code excerpts, file paths and symbol names from your repository, within the budget you set.
- **Telemetry** is off by default; when you turn it on, it stays in your repository's `.pharos` folder and holds counts only, never code.

## Feedback

Report problems and ideas on [GitHub Issues](https://github.com/Hsayndd/pharos-plugin/issues).

## Licence

Pharos Core is Apache-2.0 (`LICENSE`). The binaries include open-source components listed in `engine/THIRD_PARTY_NOTICES.md`.
