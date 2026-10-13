---
name: impact
description: Show what Pharos saved in this session — tokens (paired estimate), tool calls, cache hits, read-budget suggestions followed, and the tier mix of what was sent — and why.
disable-model-invocation: true
---
Call the MCP tool `session.impact` with no arguments and present the result as a short table:

| Metric | Value |
|---|---|
| Estimated input tokens saved | `tokens_saved_est` (with the calibration interval) |
| Tool calls this session | `tool_calls.total` (of which `tool_calls.pharos_query`, `tool_calls.raw_reads`) |
| Result-cache hits / prefetch hits | `cache_hits`, `prefetch_hits` |
| Read-budget suggestions made / followed | `suggestions.made` / `suggestions.followed` |
| Tier mix of context sent | grammar / lsp / heuristic percentages |

Then list the three plan items that saved the most tokens through compaction (`per_turn[].top_savings`) and, if `suggestions.made > suggestions.followed + 3`, add one line recommending a higher `read_budget` in `.claude/pharos.toml`. Do not estimate anything yourself: every number comes from the tool result, and if the tool reports `calibration: "uncalibrated"`, say that the savings are unmeasured until `just eval` has run on this repository.
