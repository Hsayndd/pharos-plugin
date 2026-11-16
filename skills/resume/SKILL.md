---
name: resume
description: Resume Pharos in this repository after /pharos:pause. Its hooks and tools work again and the next tool call brings the index up to date. Use when the user invokes /pharos:resume.
disable-model-invocation: true
disallowed-tools: Read, Grep, Glob, Edit, Write, NotebookEdit, WebSearch, WebFetch, Agent, Task
---

# Resume Pharos

Run exactly one command with the Bash tool, from the project directory:

    "${CLAUDE_PLUGIN_ROOT}/engine/pharos-core.exe" resume .

Then repeat its one-line answer to the user and add nothing else.
