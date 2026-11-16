---
name: context
description: Explain a Pharos MCP context plan — files, symbols, tier, view, token cost and reasons. For local preview before disclosure to Claude, use the standalone CLI path.
disable-model-invocation: true
argument-hint: <task description>
---
Call `context.plan` with `task = $ARGUMENTS`, a configured budget supplied by the host (or ask the user if absent) and `include_diff = true`. Print a table (path · symbols · tier · view · tokens · why), the total against the budget, the items dropped and why, and the tier mix. The MCP response is already visible to Claude; do not promise preview before disclosure. Direct users who require that boundary to the local CLI preview and preview-only mode in ADR-0008. Report unavailable tools honestly. Do not implement the task from this invocation.
