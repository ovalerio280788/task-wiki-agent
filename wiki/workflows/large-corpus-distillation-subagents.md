---
title: Map-reduce distillation of a large message corpus with subagents
updated: 2026-06-29
tags: [tooling, capability, gotcha]
area: workflows
---

## Context

When you must analyze a corpus far larger than one context window (for example tens of thousands of
Slack messages to build a persona, voice clone, or behavioral profile), a single agent cannot read it
all. Use a map-reduce with subagents: many workers each distill one shard to a note on disk, then one
fresh reducer synthesizes the final document from the notes (never from the raw data).

## What works

- Shard the corpus into chunks of ~1500 items per worker. At ~2300 items per worker, distillation
  workers intermittently die with `[resource_exhausted]` (their own context, not an API quota);
  ~1500 is reliably safe and still rich.
- One worker per shard, fresh context, read-only. Each reads its whole shard and writes an
  evidence-grounded markdown note (with verbatim quotes) to its own `notes/note_XX.md`. Workers reply
  with a one-line confirmation only, so their bulky output never pollutes the parent.
- Run workers in waves of ~7. Launching ~16 at once raises the `resource_exhausted` failure rate;
  smaller waves finish cleanly. Re-launch any single empty/zero-line note that stalled.
- Reducer is a separate fresh subagent that reads ALL notes (not the raw corpus) and writes the final
  profile. Give it explicit structure and tell it to weight patterns by cross-note recurrence and to
  call out evolution over time (shard the corpus chronologically so this is possible).
- Delete any biased prior summary BEFORE running, so the reducer starts unbiased.

## Reusing prior work when the dataset grows

- Shard by exact stable id (for example Slack `ts`), not by position. If the corpus later grows
  (more data collected), recompute the set of ids already covered by good notes, take only the
  uncovered remainder, and shard just that delta into new `shard_rNN` files. Keep the good notes,
  re-distill only the new shards, then re-run the reducer over the union. This avoids redoing solid
  work after a dataset refresh.

## Gotchas

- `[resource_exhausted]` on a distillation worker means the shard was too big for its context. Re-run
  it as a smaller shard; do not just retry the same size.
- A note file can exist at 0 lines while its worker is still writing or after it silently died. Verify
  by line count, not file presence, before launching the reducer.
- Date-based shard boundaries are imprecise (many items share a day). Dedup/cover by exact id so the
  delta computation has no gaps or overlaps with already-distilled shards.

## Related

- [slack-mcp-bulk-export](../tooling/slack-mcp-bulk-export.md) — how to collect the corpus this method consumes.
