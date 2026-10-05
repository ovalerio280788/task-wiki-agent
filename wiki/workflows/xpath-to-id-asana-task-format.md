---
title: XPath-to-id Asana task format
updated: 2026-09-08
tags: [testing, mobile, runbook]
area: workflows
---

## Context

How to author cards on the Automation XPath to ID Pipeline board so a Cursor cloud agent can
migrate e2e locators from XPath to id. Checkout for agent work is **instawork only**. The task
body is the spec; optional screenshot attachments are screen context only.

## What works

### Title

`[XPath] to ID: <family>`

### Sections (only these)

Summary · Steps · Locator map · Done when

Do not add Metadata, Out of scope, Automation locator, Target, or Related.

Write as Asana `html_notes`: real headings, nested lists. Put section body immediately after
`</h2>` with **no** newline or space (Asana treats `</h2>\n` as a blank line under the heading).
Between sections use exactly one blank line (`\n\n`). No `<table>` or `<pre>` for the locator
rows. Do not put a PR link in the description (use a comment when a PR exists).

Example fragment:

```
<h2>Summary</h2>Migrate … checkout is instawork only.

<h2>Steps</h2><ol><li>…</li></ol>
```

### Summary

One short paragraph: what to migrate, plus that this checkout is instawork only.

### Steps (shared baseline)

A card may add flow-specific verify notes after step 2. Keep the rest shared.

1. Edit only the automation files in the locator map (this repo).
2. Verify Candidate id against the live screen owner before using it. Follow the href/behavior the
   test actually hits.
3. If Candidate id does not match the live owner, resolve the id from that owner instead. Do not
   invent a new id.
4. If the current XPath already includes a stable id (`@resource-id`, `@name`, or the same string
   as Candidate id), use that id as the locator and remove the XPath.
5. Otherwise replace each Automation variable XPath with an id locator using the verified
   Candidate id.
6. Do not keep or add XPath fallbacks.
7. If App id source is a path in this repo and the id is missing on that element, add only that
   id. Match nearby naming and placement. No layout, copy, or behavior change.
8. If you cannot find the element in this repo and cannot add an id without guessing, stop. Do not
   invent an id.

### Locator map

One nested list row per Automation variable:

- Automation variable
- Automation file (`e2e/mobile/applicant-app/target/screens/...`)
- Candidate id (not "App target id")
- App id source: a path in this repo (`backend/`, `apps/`, `e2e/`), or
  `not in this checkout` / `not in this checkout (native RN)`

Never put mobile-repo paths such as `ids.ts` or `applicant-app/src/...` as App id source when
those files are outside the agent checkout.

### Done when

- Every row in the locator map is migrated
- No new XPath introduced

### Attachments (optional)

One failure or screen PNG per card is fine for human and agent screen context. Locator map stays
source of truth. Automation prompt may tell the agent to download an image if present and ignore
attachments when absent.

## Gotchas

- Field name **Candidate id** signals triage values need live-owner verification.
  **App target id** led agents to treat the triage string as settled.
- Dual-path XPaths that OR Hyperview and RN ids are not proof of which owner is live. Resolve
  per [xpath-to-id-app-target-resolution](xpath-to-id-app-target-resolution.md).
- `_android` / `_ios` siblings with the same stem are one control on two OSs. Migrating only the
  mapped platform leaves the other on XPath. Apply the same id to both in that file even if the
  locator map lists only one.
- Put section body immediately after `</h2>` with no newline. A single `\n` after `</h2>` is
  enough for Asana to render a blank line under the heading.

## Related

- [xpath-to-id-app-target-resolution](xpath-to-id-app-target-resolution.md)
- [documentation](../preferences/documentation.md) — Asana `html_notes` list rules
