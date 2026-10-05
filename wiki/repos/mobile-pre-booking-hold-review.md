---
title: "mobile: pre-booking hold / Please review the shift details"
area: repos/mobile
updated: 2026-08-31
---

# Mobile: "Please review the shift details" + Shift reserved

Sticky CTA on gig details after the worker starts booking. Copy lives in
`worker/fragments/gig_details_cta/pre_booking_hold/{hold_timer,holding}.xml`.

## Exact screenshot state

- "Please review the shift details"
- "Shift reserved for MM:SS"
- **Continue booking** disabled

That needs `disable_continue_button=True`: active `HOLD_GIG_DETAILS_REVIEW` hold, worker has
**no completed** past shifts at this company, and the extra first-company review window is still
running. Workers who already completed a shift at the company still get a hold, but Continue is
enabled immediately (no review copy).

## Seed to reach it

1. Create an **unbooked** marketplace shift group (open slot). Do **not** `PATCH` `worker_id` /
   book via Automation API — that skips this UI.
2. Gig must **not** be LTA / multi-day-required / via-application / interview.
3. `starts_at` must be farther out than Constance `GIG_HOLD_GIG_DETAILS_TIME_TO_START_HOURS`.
   Check live value before seeding — **QA has been 360h (15 days)**, not the code default.
4. Use a company where the worker has **zero completed** shifts
   (`get_past_shifts_at_company(..., completed_only=True)`). Booked-then-deleted is fine; completed
   is not.
5. Grant `POST /shift-groups/{id}/worker_access` with `{"worker": "<id>"}` so the worker can open
   the gig. Leave the shift unbooked.
6. In the app: open that gig → tap Book. The POST creates the hold and shows this footer.
7. Continue enables after `GIG_HOLD_GIG_DETAILS_FIRST_COMPANY_SHIFT_MINUTES` (client timer event
   `enable-continue-booking-button`; QA has been 1 minute).

## Related

- [instawork-automation-api](../tooling/instawork-automation-api.md) — leave shifts unbooked for
  in-app booking UI
