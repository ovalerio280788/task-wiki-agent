---
title: Bulk-exporting Slack messages via the Slack MCP
updated: 2026-07-01
tags: [tooling, capability, gotcha]
area: tooling
---

## Context

When collecting many Slack messages through the Slack MCP server (for example exporting a person's
authored messages across channels), a naive paginate-everything approach exhausts the agent context
window and the run dies with `resource_exhausted`. The fix is a streaming, low-context protocol.

## What works

- Get the acting user's own `user_id` from the tool descriptor text (the search tool descriptions
  state "Current logged in user's user_id is ..."), so identity needs no extra discovery call.
- Author filter modifier in the search `query`: `from:<@USER_ID>` (angle brackets are literal for IDs).
- Exclude a DM counterpart by appending a negated DM filter: `-in:<@OTHER_USER_ID>`.
- Shrink every response: pass `response_format="concise"` and `include_context=false`. Search returns
  at most 20 results per page; paginate with the returned `cursor`.
- Stream to disk: after each page, extract only the fields you need and append them to a file
  immediately, then discard the raw response from working memory. Never accumulate raw API responses
  across pages. Cap the number of pages.

## Paging past the 20-page-per-query cap

- Each search `query` is hard-capped at 20 pages. Page 21 fails with `page_limit_exceeded` (a tool
  limit, unrelated to context exhaustion). A month with heavy activity will exceed this.
- The `after:` modifier only accepts `YYYY-MM-DD` dates. A raw numeric Unix timestamp silently
  returns "No results found", so it cannot be used to resume mid-stream.
- To resume past the cap, narrow the date window instead: re-query a smaller span (for example a
  single day, `after:DAY-1 before:DAY+1`), page through it, then continue with `after:LAST_DAY`.
  Slack `after:`/`before:` are day-exclusive bounds.
- Narrower windows re-surface already-saved messages at the boundary. Dedup by message `ts`: skip
  any record whose `ts` is at or before the last one already written.
- Because the bounds are day-exclusive, a window written as `after:MONTH-01 before:...` silently drops
  day 1 of the month. To capture a full month start, begin at the previous day (`after:lastDayPrevMonth`)
  or run a dedicated first-day window; rely on the `ts` dedup pass to absorb the overlap.
- Same gotcha between adjacent weekly windows: if window N ends `before:D` and window N+1 starts
  `after:D` (same date `D`), calendar day `D` is dropped by both. Make windows overlap (window N+1
  starts `after:D-1`), or backfill each shared boundary day with a dedicated `on:D` query. The cleanest
  fix when you already collected with shared dates is a per-day `on:YYYY-MM-DD` pass for each boundary.
- When a window caps mid-day (page 21 fails but the day is not finished), re-query that single day
  with `sort_dir="desc"` and append only the records newer than the last `ts` you saved, stopping
  once you reach it. This grabs just the uncovered tail instead of re-paging the whole day ascending.
- Use `response_format="detailed"` (not concise) when you need real epoch `ts` and accurate dates.

## Writing results to disk

- Prefer durable per-window shard files over one long append loop. Extract each page into a private
  staging file, then do one final merge plus a dedup-by-`ts` pass. Shards protect against context
  loss, slow shells, and mid-run retries.
- Build JSONL with `jq -nRc` reading fixed line groups from a quoted heredoc. `jq` handles JSON
  escaping safely, and the quoted heredoc stops shell expansion inside message text.
- Avoid piping page text through `python3` here. In this workspace, interpreter startup and stdin
  reads can hang long enough to stall the run. If a Python parser is unavoidable, run it foreground
  with `python3 -S` and keep the script stdlib-only.
- The MCP response only exists in agent context. If you need to re-emit raw search results for
  parsing, save the markdown as plain text and split on `### Result N of N`; do not `json.load` the
  whole tool payload because `results` contains literal newlines.
- If parsing markdown, parse text non-greedily up to the trailing separator (`Text:\s*\n(.*?)\n---`);
  a greedy match leaks `---` separators into message text.
- Skip non-target author records that slip through `from:`, skip empty-text messages, and map
  self-DMs to your own name instead of `group`.
- If several sibling agents collect slices in one directory, namespace every helper, batch, and
  staging file with a unique prefix. Only copy or move onto the canonical slice path as the final
  step, then confirm the last expected record is present before the final dedup.

## Gotchas

- `resource_exhausted` here usually means context exhaustion, not an API rate/quota limit. Reducing
  per-response size and streaming to disk fixes it; retrying the same heavy approach does not.
- The concise format returns no raw epoch `ts`, and the date strings it surfaces can carry shifted or
  non-real years. Treat absolute dates from concise search as unreliable; relative ordering
  (newest first with `sort=timestamp`) is still correct. If you need true dates, fetch them another way.
- Heavy single-purpose collection is best delegated to a subagent with an explicit low-context,
  bounded-pages brief, so a blow-up does not take down the parent.
- The Slack search rate limit is shared across the whole token, so parallel collector subagents all
  draw from one budget. Running ~8 at once thrashes it: progress goes glacial and agents die with
  "Connection failed repeatedly" (a 429 storm, not a code bug). ~4-6 concurrent is the sweet spot, and
  survivors visibly speed up as peers finish. Long-running per-window agents also drop connections
  mid-run and lose nothing only because each writes to its own staging file. Plan for deaths: give each
  agent a narrow window, let it stream to a unique file, and reconcile at the end with a single
  dedup-by-`ts` merge across every staging/slice file rather than chasing each failure live.
- Size the brief to roughly one dense month per subagent. A single busy month can be ~1100 authored
  messages spanning ~55-60 search pages, which on its own consumes a subagent's whole context. Do not
  hand one subagent a multi-month window expecting completion; split by month (or finer) and have each
  subagent flush a durable per-slice file before it runs out. Writing cleaned JSONL records straight to
  disk per page (rather than re-emitting raw + parsing) roughly halves the output-token cost per page.

## Related

- [github prs via gh cli](github-pr-gh-cli-cloud-agents.md) — same pattern of bulk-pulling personal
  activity, but via CLI instead of MCP.
