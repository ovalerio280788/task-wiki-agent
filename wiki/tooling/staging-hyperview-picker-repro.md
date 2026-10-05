---
title: Staging Hyperview picker repro (no OpenSearch)
updated: 2026-09-18
tags: [tooling, mobile, backend, testing]
area: tooling
---

## Context

Need a native iOS select wheel (`PickerField` / `UIPickerView` in a Fabric modal) on staging2 without seeding Easy Apply + OpenSearch. v2 can create an artform or syndicated job row; it cannot put the job on the Jobs tab.

## What works

Skip data setup first. Staging is not `PRODUCTION`, so these Hyperview screens exist:

- `/artforms/my-dynamic-form` — unauthenticated demo form; `string_picker` is an enum (not radio), so it uses `PickerField`.
  Open in the applicant app with `instawork://hv-dev/home/worker-app-my-dynamic-form` (Django URL
  name, not the HTTP path). `hv-dev` exists only when the backend is not production. Do not use
  `instawork://artforms/my-dynamic-form` — that slug route is a real artform intro.
- MDS storybook `form_field` / `pickerfield_default` — same widget; e2e reaches it via `mobile-storybook-component`.

Staging repro (picker crash class):

0. **Build from a tag carrying RN 0.86.3**, not merely "the RN 0.86 line". Every observed event is
   on 0.86.3; 0.86.2 shipped on exactly one store build that reached ~1 user, so it has no field
   exposure. A feature branch cut off the earlier tag builds 0.86.2 and will not reproduce.
   RN 0.86.3's changelog includes a Modal fix restoring capture/bubble event propagation to
   ancestors of nested root host views — the same touch-propagation-around-Modal area this crash
   lives in, so treat 0.86.3 as both the repro target and a cause candidate.
1. Applicant **beta / adhoc** build (release configuration, not Debug, not App Store). Point it at
   staging: `https://app.staging2.instawork.com`.
2. **iOS 26.4 or newer.** Older iOS 26.x and iOS 18 do not produce it.
3. Open any Hyperview screen with a `PickerField`
   (`instawork://hv-dev/home/worker-app-my-dynamic-form`, or
   `worker-app-profile-info-self-identification`).

Then take one of the two routes below. Both end in a touch hit-testing a picker whose cells were
already freed; they differ in how the picker goes stale.

### Route A — idle + background cycle (the one that works on internal builds)

4. Tap the picker so the wheel modal opens. **Stop touching the screen.**
5. Background the app (home / app switcher) and return to it.
6. Leave the wheel open and idle for **several minutes** — observed crashes sat at 6 and 10 minutes.
7. Touch the screen. It dies on that touch.

The idle wait is the point: backgrounding plus time lets iOS purge the picker's table cells while the
modal is still presented. Constant interaction prevents it.

### Route B — rapid consecutive opens (what store users hit)

4. Open picker → pick → open the **next** picker → pick → open a third, **2–5 s apart**, without
   leaving the screen.
5. Crash lands ~2–3 s after the third or fourth presentation.

### Device class and free RAM are not a filter

Do not treat `device.free_memory` / `memory_size` as jetsam pressure. iOS keeps a few hundred MB
free by design, so ~5% free on an 8–12 GB phone is normal, not a starvation signal. Across the
issue, ~95% of events are `device.class: high` (flagships: iPhone 16 / 17 families). Medium-class
is a few percent. Slow or low-memory phones are not the population.

Zombie Objects still helps turn a use-after-free into a deterministic abort when running under
Xcode (scheme → Run → Diagnostics). That is a debugger trick, not evidence of a field memory
pattern.

### Why a slow, careful pass never crashes

Holding one wheel open and *actively spinning it* reproduces neither route, no matter how long. The
wheel needs to be left alone (Route A) or the presentations need to stack up (Route B). One modal
open for 30 s of continuous spinning is a single presentation with no idle window.

For a real application-quiz artform without browsing jobs, Django shell on staging:

1. Create a throwaway artform (`is_preview=True`) whose JSON schema has a string `enum` and whose ui schema does **not** set `ui:widget` to `radio`.
2. Open `/artforms/worker-artform-intro/<artform_id>/` as a logged-in pro (intro creates the response).

Do not rebind `SyndicatedJob.application_quiz` on shared open jobs.

## Gotchas

- `ui:widget: radio` is radios, not the wheel. Date `format: date` is the date picker, not this crash.
- `POST /educations` writes dates and never opens the month wheel.
- A new `syndicated_job` stays invisible in the app until OpenSearch upsert; the intro URL above does not need that.

## Related

- [instawork-automation-api](instawork-automation-api.md)
- [instawork-local-hiring-opensearch-setup](../workflows/instawork-local-hiring-opensearch-setup.md)
