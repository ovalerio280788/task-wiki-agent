---
title: "mobile: ABILITIES Verify Now post-booking flow"
area: repos/mobile
updated: 2026-06-25
---

# Mobile: ABILITIES "Verify Now" post-booking flow

## What this is

After a worker books a shift that has ABILITIES requirements (`partner_requirements`), the Gig
Booked screen shows a "Verify now" bottom sheet. The worker taps it, answers yes/no to each
ABILITIES question, and reaches a "Yay! You meet all the requirements" completion screen.

## When it appears (all conditions must be true)

1. The gig's `gig_template.partner_requirements` contains at least one requirement.
2. The gig's `gig_template.requirement_presets` does **NOT** include the "Professional" grooming
   preset (preset id=19). The Professional preset adds an inline ABILITIES acceptance term to the
   Gig Confirmation screen, so the worker verifies abilities inline instead of post-booking.
3. The worker has **no prior ability answers** for those requirements. A worker who previously
   verified "Stand for long hours" (or any other requirement in the list) will not see Verify Now
   for subsequent bookings.

## Data setup to trigger the flow (QA automation API)

```bash
# 1. Create shift group with ONLY the attire preset, NO grooming preset
curl -s https://qa.instawork.com/automation/api/v2/shift-groups \
  -H "Content-Type: application/json" \
  -d '{
    "business_name": "<business>",
    "position_name": "Bartender",
    "starts_at": "...",
    "ends_at": "...",
    "attire_preset": "Black Bistro",
    "add_attire_preset": true,
    "add_grooming_preset": false,
    "geofence_radius_meters": 1500
  }'

# 2. Add ABILITIES requirement to the new shift group
curl -s https://qa.instawork.com/automation/api/v2/shift-groups/<SHIFT_GROUP_ID>/add-requirement \
  -H "Content-Type: application/json" \
  -d '{"gig_requirement_id": 1}'
```

After creation, verify:
- `gig_template.requirement_presets == [1]` (Black Bistro only, no Professional)
- `gig_template.partner_requirements == [1]` (Stand for long hours)

## Why the Professional preset blocks the flow

- Bartender gig templates get `add_grooming_preset=true` by default.
- This adds the "Professional" grooming preset (id=19), which injects an extra `instruction/N`
  term on the Gig Confirmation screen that also serves as an inline ABILITIES acceptance.
- Accepting that term satisfies the ABILITIES requirement immediately during booking.
- The post-booking Verify Now flow checks if requirements are already verified — finds they are —
  and does not show the bottom sheet.

## Worker requirements

- Must be a **fresh worker** (no prior ABILITIES answers) or a worker whose ability answers
  have been cleared.
- `is_internal_user: true` — bypasses geofence check for BrowserStack device location.
- `regionmapping: bay_area_ca_us` when the gig business is in San Francisco.
- Full setup: `worker_score (90/90)`, `gig_position (Bartender, active, quiz_completed)`,
  TOS agreements (ids 68/69/104/106), Bartender certificates, `auto_login_token`.

## Verified in

Run25 of the `pro-book-shift-with-prerequisites` scenario on 2026-06-25:
- Worker 10541849, Shift 53v8QBj, Gig template owOYeQo
- All 21 steps passed including Verify Now → "Stand for long hours" → "Yes" → "Yay!"

## Related pages

- [instawork-automation-api](../tooling/instawork-automation-api.md) — API preflight and gotchas
- [mobile-automation-pipeline](../workflows/mobile-automation-pipeline.md) — full end-to-end pipeline
