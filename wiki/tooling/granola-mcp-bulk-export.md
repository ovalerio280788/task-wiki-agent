---
title: Granola MCP bulk transcript export
updated: 2026-09-29
tags: [tooling, capability, gotcha, data]
area: tooling
---

## Context

Pulling a complete history of Granola meeting transcripts (for a persona corpus, voice clone, or
meeting analysis) through the `plugin-granola-granola` MCP. The useful tools are `list_meetings`
(index) and `get_meeting_transcript` (verbatim text per meeting id). `get_account_info` confirms
which account is connected.

## What works

- Build the index with `list_meetings`. The live schema accepts only `this_week`,
  `last_week`, or `last_30_days`. It returns id, title, date, and participants per meeting.
  For the content of a named meeting, follow with `get_meetings` or `query_granola_meetings`.
- The index response is capped at roughly 138 meetings (about 44 KB), and `list_meetings` cannot
  page earlier than `last_30_days`. For a specific older discussion, use `query_granola_meetings`.
- Transcript speaker labels are fixed: `Me:` is always the connected account owner, `Them:` is
  every other speaker (not individualized by name). Filter on `Me:` for the owner's own words.
- For bulk pull, split meeting ids into batch files and hand each batch to a subagent that loops
  `get_meeting_transcript` and writes one file per id. Large tool outputs are spilled to an
  agent-tools file, so instruct workers to `cp` that path rather than retype (preserves verbatim
  fidelity and avoids context bloat).

## Gotchas

- `get_meeting_transcript` is hard rate-capped server side at roughly one transcript per two
  minutes. This is per-account, not per-caller: adding more concurrent subagents does NOT speed it
  up and tends to thrash (same lesson as Slack search). Plan a slow bulk pull (about 30 per hour)
  and proceed to distillation on a partial-but-representative sample rather than blocking for hours.
- No local bypass. The desktop cache `~/Library/Application Support/Granola/granola.db` is encrypted
  (`sqlite3` reports "file is not a database"), and `cache-v6.json.enc` is encrypted too. The MCP is
  the only practical extraction path.
- `list_meetings` rejects `time_range: "custom"` (and `custom_start` / `custom_end`) even
  though older notes describe that window. Do not retry the same call. Use `last_30_days`,
  then `query_granola_meetings` for a specific discussion.
- For the mechanical fetch-to-disk step a fast model (composer) is cheaper than a thinking model,
  but the throughput ceiling is the MCP rate cap, not the model.

## Related

- [large-corpus-distillation-subagents](../workflows/large-corpus-distillation-subagents.md) — shard, distill, reduce the transcripts once collected.
- [slack-mcp-bulk-export](slack-mcp-bulk-export.md) — same shared-rate-limit thrash pattern on a different MCP.
