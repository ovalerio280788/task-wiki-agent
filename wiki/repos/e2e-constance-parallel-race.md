---
title: e2e constance overrides race across parallel scenarios
updated: 2026-10-27
tags: [testing, qa, mobile, gotcha]
area: repos/instawork
---

## Context

A Behave scenario tagged `@set_constance::<KEY>::<VALUE>::before` and
`@set_constance::<KEY>::<DEFAULT>::after` writes a Constance value that is shared by the whole
backend environment. The step is a plain API write with no scoping and no save/restore: `::after`
writes a hardcoded literal, not the value that was there before.

Any scenario that reads the same key is a victim, including scenarios in other feature files that
never set it.

## Gotchas

Repeating the same scenario in parallel still races, even when every copy sets the same value.
Each copy sets the override at the start, and the first copy to finish runs `::after` and restores
the default while the others are still inside the window that reads the flag.

A later failure in that window is the override race. It does not say anything about an earlier
step that does not read the flag.

**The race is not only intra-run.** Each mobile platform runs as its own CI pipeline and its own
Allure launch, but both point at the same shared backend and start close together. Two launches
therefore execute the same feature file at overlapping times, and each one's `before`/`after`
hooks clobber the other's. Collocating scenarios in one feature file only serialises them inside
a single worker, so it does not fix this and the flake survives.

**The value on screen can be recomputed at render time.** A record created while the override was
correct may still be displayed using whatever the config says at the moment the screen is
requested. Assume the exclusive window must cover every assertion, not just setup.

**`::after` encodes an assumption about the environment baseline.** It restores a literal that
someone believed the env held, which may differ from the code default and drifts when the env is
re-synced. Two scenarios that disagree about the baseline will poison each other even without
concurrency.

## What works

Recognising it:

- In the Allure warehouse, self-join scenario results on overlapping `starts_at`/`stops_at` across
  *different* `launch_id`s, then compare the non-passed rate for overlapping versus non-overlapping
  runs. A clear multiple is the signature.
- Prove it at HTTP level with the BrowserStack network log (HAR): the response body carries the
  *other* scenario's configured value, which is far stronger evidence than a screenshot. Symmetric
  pairs (each scenario receiving the other's setting) rule out a product bug.

Fixing it — prefer per-entity test data over any global write:

- Backend config lookups often consult a **per-worker override record first** (a payment-feature
  style row whose `metadata` carries the same keys) and fall back to the global key only when
  absent. Creating that record isolates the scenario completely, and the automation API usually
  already accepts the metadata object. Put integers in that metadata (a fee in cents). A fraction
  written as JSON (`0.05`) is read back as a float, and the reader passes it straight to
  `Decimal()`, which is not the written fraction and drifts by a cent. Leave the fraction off the
  metadata so the existing decimal config value is used.
- When the global key is a **list matched against an entity attribute** (state codes, country
  codes), pick an entity value that is on the wanted side of the existing list instead of rewriting
  the list. Same code path, no shared write. Confirm which record the code actually reads. A worker
  profile state and the shift's business place are different checks.
- Do not patch a place, company, or business that other features share. Append a second place
  (real address, state already on the wanted side), then its own company and business, before the
  step that reads `businesses[-1]`. `create_place` merges a San Francisco default, so override
  `google_place_id` and the address fields; sending the default id returns the existing SF row.
  Look the place up by full address first.
- If the scenario also waits for "close to location", the `@gps_` tag has to be that place's
  coordinates. The check is a geofence around the gig place (default 500m), applied from the
  scenario tag during the location-permission step. A step that asserts the shared company name
  will fail once the shift is on the new business.
- A W-2 shift on a place outside the W-2-only state list is rejected until that business's
  subsidiary is a W-2 subsidiary. The businesses detail route is retrieve and delete only, so an
  existing row cannot be updated. The create endpoint's help text says subsidiary is writable, but
  its custom create ignores that field and the factory stamps the 1099 subsidiary. Sending it on
  POST does not stick. A get-or-create by name keeps that row. Prefer a place whose state is
  already W-2-only and already on the wanted side of the other list. That state forces the W-2
  classification, so the subsidiary field never has to be written. Gig details shows the company
  name, so the new business can belong to the existing company. Give the business its own name or
  the lookup returns the original location. Point `@gps_` at the new place: the distance check
  uses that place's coordinates.

**A shared catalog delete is the same race, even when a missing row counts as success.** Deleting every live document so a check becomes vacuously true still collides: the delete removes other scenarios' per-user acceptance rows, and a concurrent create becomes the new latest for everyone. Leave the catalog row in place. Attach a per-user acceptance to the latest live row. Latest is the highest live id; a list ordered by created time is not that row, and a newer not-live row must not be the one accepted. Scenarios that must show the unsigned state use a fresh user and the UI, and they must not delete the catalog. Creating one newer live version still moves the latest pointer. That write is safe once the row already exists.

Only if no per-entity escape hatch exists: serialise. Put every scenario that writes the key in one
feature file so one worker runs them sequentially, and accept that this still leaves the
cross-launch race open unless the pipelines are also prevented from overlapping.

## Related

- [browserstack-failure-triage](../tooling/browserstack-failure-triage.md) — session and network
  log grounding for the HAR evidence step.
- [redshift-allure-ops](../tooling/redshift-allure-ops.md) — where the launch/scenario timing and
  failure text live.
