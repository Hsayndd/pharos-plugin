---
name: pause
description: Pause Pharos in this repository without uninstalling it. Its hooks pass through and its tools do no work until /pharos:resume; the index stays on disk. Use when the user invokes /pharos:pause.
disable-model-invocation: true
disallowed-tools: Read, Grep, Glob, Edit, Write, NotebookEdit, WebSearch, WebFetch, Agent, Task
---

# Pause Pharos

Run exactly one command with the Bash tool, from the project directory:

    "${CLAUDE_PLUGIN_ROOT}/engine/pharos-core.exe" pause .

Then repeat its one-line answer to the user and add nothing else. Pausing only writes the marker file `.pharos/paused`; the index, memory and caches stay on disk, and `/pharos:resume` turns Pharos back on.
