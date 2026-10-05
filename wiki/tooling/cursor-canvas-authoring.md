---
title: Cursor canvas authoring
updated: 2026-09-19
tags: [tooling, gotcha, capability]
area: tooling
---

## Context

Turning an analytical artifact (sprint packet, audit, metrics breakdown) into a Cursor canvas — a
single `.canvas.tsx` React file the IDE compiles and opens beside the chat.

## What works

- Read the canvas skill first, then the SDK declarations in
  `~/.cursor/skills-cursor/canvas/sdk/*.d.ts` for exact exports and prop shapes. Referencing a
  non-existent export is the most common runtime failure.
- Write the file directly to `~/.cursor/projects/<workspace-slug>/canvases/<name>.canvas.tsx`.
  Nothing outside that exact directory is detected. The directory is pre-provisioned; do not
  `mkdir` or check for it.
- The write/edit tool result ends with a `Canvas TypeScript check` line. That is the authoritative
  diagnostics signal — iterate until it says no errors.
- Import only from `cursor/canvas`, embed all data inline, default-export the component.
- Converting a markdown report: keep the numbers and links, drop the markdown scaffolding.
  Multi-line table cells and repeated one-task-per-row hacks are unnecessary. Use real columns
  (`Slice | Task | Owner | Status`) and put the group label on the first row of each group.
- Sprint kickoff: write the dated markdown packet first, then the canvas from that file using the
  matching **Canvas structure** in [qa-platform-sprint-kickoff](../workflows/qa-platform-sprint-kickoff.md)
  step 6 (kickoff vs week two are different layouts). Filename
  `qa-platform-sprint-<kickoff|week-two>-YYYY-MM-DD.canvas.tsx`. Copy structure from the latest
  canvas of the same mode, not its inventory.
- Quarter close-out: no markdown packet. Filename `qa-mission-<n>-q<n>-close-out.canvas.tsx`.
  Structure is in [qa-platform-quarter-close-out](../workflows/qa-platform-quarter-close-out.md).
- Kickoff headline stats use `Completed`, `Carry Over`, `New In Sprint`, `Full Workload`; the
  definition table under them is where the board/column meaning lives. Week two canvas is a
  check-in: last week landed plus this week's focus. Do not put Carry Over, New In Sprint, or
  Full Workload on it.
- Link the packet with `useCanvasAction` + `Button` `openFile` (workspace path). Do not use a
  browser `Link` for a local markdown file.
- Kickoff "who is working on what": one `Card` per contributor — short speakable focus, Carry/New
  `Stat`s, then a label-free `UsageBar` (shared `total` = heaviest load). Do not put focus prose
  into `UsageBar` `topLeftLabel`. Week two cards are focus-only; no Carry/New stats or load bar.
- Percent-complete bars: `UsageBar` with `total` = denominator and a single segment = numerator.
  Put the name, `%`, and raw `done / total` in a `Row` above it (spacer `<span style={{flex:1}}/>`
  to push the numbers right), and one line of "why this number" below. Colour by band (green /
  orange / gray), not one hue per row.

## Gotchas

- Prop types are closed and do not extend React's intrinsic attributes, so `key` is a type error
  on SDK components (`Stat`, `Stack`, `Callout`) **and** on your own local components. Wrap the
  mapped element in a plain `<div key={...}>` — native elements still accept `key`, and that beats
  enumerating the list by hand.
- The `React` namespace is not in scope. It fails both as a helper return type
  (`React.ReactNode[][]`) and as a prop annotation (`{ children: React.ReactNode }`). Let
  TypeScript infer return types, and type children concretely (`{ children: string }`).
- Emoji are forbidden in a canvas even when the source document uses them as section markers.
  Carry the meaning with tone (`Stat tone`, `Callout tone`, `rowTone`) instead.
- Never render empty sections. A source document that intentionally lists empty buckets should
  collapse into one coverage table with counts, not a section per empty bucket.
- Charts need a self-describing title, units, and a source/time-range caption. `BarChart` has no
  axis-label props, so put the axis meaning and units in the caption and `valueSuffix`.
- `BarChart` cannot put freeform "what" text on stacked bars (`showValues` is numeric-only and
  disabled when stacked). Prefer contributor cards + `UsageBar` for kickoff load.
- A `Stat` strip is the easiest place to lose the reader: `Stat` takes only `value` and a one-line
  `label`. Keep the strip labels short (`Completed`, `Carry Over`, `New In Sprint`,
  `Full Workload`) and put meaning in a definition table (`Label | What it counts | Source`)
  under it. Never show a derived total without naming its inputs.
- Canvas SDK has no `Image` component and forbids `fetch()`. To attach Allure (or other) failure
  screenshots: download PNGs into the workspace, then put a `Button` + `useCanvasAction`
  `openFile` under each finding. Pick one representative frame per task from
  `all_error_details.json` → `test_runs[-1].screenshot` → `data/attachments/<hash>.png`.
  Do not try to inline data-URI galleries in the `.canvas.tsx`.

## Related

- [qa-platform-sprint-kickoff](../workflows/qa-platform-sprint-kickoff.md)
- [qa-platform-quarter-close-out](../workflows/qa-platform-quarter-close-out.md)
- [mobile-ci-allure-failure-triage](../workflows/mobile-ci-allure-failure-triage.md) — screenshot
  filenames live on `all_error_details.json` test runs.
- [circleci-api-artifact-download](circleci-api-artifact-download.md) — pull Allure attachment PNGs.
