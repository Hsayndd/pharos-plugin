# Pharos for Claude Code

**A map of your code for Claude.** Pharos builds a symbol graph of your repository on your machine: its declarations and the calls, imports, includes, inheritance and overrides between them. Claude then answers "where is this defined?", "who calls this?" and "what implements this?" in one tool call, instead of searching and reading file after file. Pharos itself makes no network calls.

**Current release: Core 1.1.2, for Windows x64.** Free and open source (Apache-2.0).

[Guide](https://github.com/Hsayndd/pharos-plugin/blob/main/GUIDE.md) · [How it works](https://github.com/Hsayndd/pharos-plugin/blob/main/HOW_IT_WORKS.md) · [Release 1.1.2](https://github.com/Hsayndd/pharos-plugin/releases/tag/v1.1.2-windows) · [Report a problem](https://github.com/Hsayndd/pharos-plugin/issues/new/choose)

## Install

You need Claude Code 2.1.292 or later, and Git. No Node, no Python, no runtime.

**Windows x64 (Core 1.1.2).** Run these two lines in Claude Code:

```
/plugin marketplace add Hsayndd/pharos-plugin#windows
/plugin install pharos@pharos
```

**macOS and Linux (0.2.0-beta.2, an older build).** No 1.1.2 build exists for them yet; it is planned for 1.2.0. The beta parses TypeScript, JavaScript, Python and Rust, and lacks the fixes and features of later releases.

```
/plugin marketplace add Hsayndd/pharos-plugin
/plugin install pharos@pharos
```

On a slow connection the download can pass Claude Code's two-minute limit and fail with a misleading `Permission denied (publickey)`. Set `CLAUDE_CODE_PLUGIN_GIT_TIMEOUT_MS=600000`, restart Claude Code and install again. Details in the [guide](https://github.com/Hsayndd/pharos-plugin/blob/main/GUIDE.md#if-the-install-fails).

## Two minutes to a first answer

1. Open a repository in Claude Code.
2. Ask: *"Use index.status to index this repository."* Pharos writes its index to a `.pharos` folder there, which Git ignores.
3. Ask a structural question: *"Use pharos.query to list every caller of `parseConfig`."* The answer lists each caller with its file, line and the line itself.

More in the [guide](https://github.com/Hsayndd/pharos-plugin/blob/main/GUIDE.md#first-run): context plans within a token budget, memory tied to commits, and pause and resume.

## What was measured

Paired experiments with Claude Sonnet 5.5 in Claude Code, on frozen builds of this plugin: the same questions with Pharos on and off, each repository indexed beforehand. Each figure is the mean of paired per-task changes with its 95 % interval over tasks: a range for the average over tasks like these, not a guaranteed saving on yours.

| Where | Input tokens | Tool calls |
|---|---|---|
| nestjs, tokio and pytest (117,000 to 188,000 lines), 15 tasks | −14.6 % (−26.4 % to −0.1 %) | −30.4 % (−41.8 % to −17.1 %) |
| llama.cpp (877,000 lines), C, C++ and CUDA questions, 6 tasks | −46.3 % (−64.5 % to −28.6 %) | −52.3 % (−71.1 % to −31.9 %) |

- **Wall-clock:** −23.3 % (−37.1 % to −6.6 %) on the first set, with indexing done beforehand.
- **Not significant everywhere:** over all 10 tasks of the llama.cpp experiment, the token change was −21.5 % (−45.0 % to +2.5 %).
- **Tokens, not dollars:** input tokens include cached tokens, which are priced differently.
- **Where it costs more:** on a small repository, or a question one search answers, Pharos adds a fixed overhead to each turn and can cost more than it saves.
- **Answers:** in both experiments every answer named every expected symbol, with and without Pharos. That checks the expected names appear; it does not measure complete correctness.

## Languages and limits

- **Parsed in full:** TypeScript, JavaScript, Python, Rust, C, C++ and CUDA (macOS and Linux beta: the first four).
- **Indexed by pattern:** every other text file, Go, Java and C# among them. Declarations are found; calls are not.
- **Not followed:** a React component rendered as a JSX element; calls through re-exporting files (linked by name only); dependency injection and other uses by parameter name, such as pytest fixtures; calls inside a Rust macro's arguments or a C or C++ `#define`; dynamic calls, reflection and generated code.
- **No language server** is used. Every file Git doesn't ignore is indexed at one of two tiers, and `index.status` says which.
- **Unsigned binaries.** Check the release archive against its `SHA256SUMS` file. A checksum shows the file is the one we built, not who built it.

How the graph, the plans and the hooks work: [HOW_IT_WORKS.md](https://github.com/Hsayndd/pharos-plugin/blob/main/HOW_IT_WORKS.md).

## Privacy

- Pharos's binaries make no network call. The index, memory and settings stay in your repository's `.pharos` folder and your home folder.
- Pharos's answers are tool results: Claude Code sends them to the model like any other tool output. They hold code excerpts, paths and names from your repository, within the budget you set.
- Telemetry is off by default. When you turn it on, it stays in your repository's `.pharos` folder and holds counts only, never code.

## Feedback

Bugs, wrong answers and ideas: [open an issue](https://github.com/Hsayndd/pharos-plugin/issues/new/choose). A wrong or missing link is the most useful report we can get. Security problems: write privately to houssen.kanaan@gmail.com, not in a public issue.

## Licence

Pharos Core is Apache-2.0 (`LICENSE`). The binaries include open-source components listed in `engine/THIRD_PARTY_NOTICES.md`.

## Upgrade to Pharos Pro ($5 a month for the first 100 customers)

Pharos Pro is the paid edition, built on the same engine. It adds:

- **What a change can break** (`/pharos:blast-radius`): your uncommitted work or a branch, followed through its users to several depths, with what Pharos cannot see listed in the report. It also runs in CI on a persistent self-hosted runner.
- **Questions and context plans across repositories** (`/pharos:workspace`): register several local repositories, then ask across them, each answer cited by repository.
- **A dashboard** (`/pharos:dashboard`): your licence, index coverage, workspace and last session.

| | Per seat |
|---|---|
| **Launch offer, for our first 100 customers** | **$5 a month**, or $60 a year |
| Standard price, after that | $8 a month, or $96 a year |

- **Windows x64 only for now,** and it needs Node 24. macOS and Linux are planned.
- **One seat** is one named person on one registered machine. The licence is checked offline: no account, no activation server.
- **What Pro does not do:** change impact follows uses within one repository, and it has the blind spots listed above. No experiment has measured a saving from Pro's own features; the measured results above are Core's.

**To order,** write to [houssen.kanaan@gmail.com](mailto:houssen.kanaan@gmail.com?subject=Pharos%20Pro%20order) with your name or company, the number of seats, monthly or yearly billing, and your main languages. You receive an invoice, then the package and your licence by email.

Pharos is an independent product of baalbek.dev and is not affiliated with or endorsed by Anthropic. Claude and Claude Code are trademarks of Anthropic, PBC.
