---
title: Instawork automation API (/automation/api/v2)
updated: 2026-10-28
tags: [tooling, testing, backend, local, qa, gotcha]
area: tooling
---

## Context

The local Instawork backend on `:8080` exposes a test-data automation surface at
`/automation/api/v2`. Both the web Playwright suite and the mobile Instatest suite depend on it
for seeding and verification. Confirm it before blaming a test runner.

## What works

Preflight the API in about 10 seconds before running any suite:

```bash
# swagger should return 200 when the stack is automation-ready
curl -s -o /dev/null -w "%{http_code}" http://localhost:8080/automation/api/v2/swagger
# a real endpoint (no trailing slash)
curl -s http://localhost:8080/automation/api/v2/constance-variables/PARTNER_PHONE_VERIFICATION_REQUIRED
```

Bulk env cleanup (demo/QA) via per-id `DELETE` — there is no wipe-all endpoint. Practical order:

1. `shift-groups` (often clears related bookings; transactions may drop with them)
2. leftover `transactions`
3. `company-payment-sources` then `companies` then `businesses` — **before** workers
4. worker dependents (`gig_position`, `payment-settings`, `worker-score`, …)
5. `user-incentives` via `DELETE /user-incentives/delete-by-worker?user=<id>&incentive__goal=<exact goal>`
6. `workers` (this is the user-profile surface — no separate `/user-profiles`)

Drain by repeatedly deleting page-1 results until `count=0`; do not trust a single paged sweep.
Delete companies before workers: wiping owners first can leave companies that
`DELETE /companies/{id}` cannot remove (`UserProfile matching query does not exist`).

Staging2 uses the same paths on `https://admin.staging2.instawork.com/automation/api/v2`. `GET .../swagger` returns the OpenAPI JSON (`swagger.json` 404s). No auth header is required from a machine that can already reach admin. Worker detail accepts the numeric userprofile id from the admin URL. Company and business detail accept the hash id from the admin change URL.

## Gotchas

- A shift `starts_at` / `ends_at` with no offset is parsed in the request timezone. An unauthenticated call has none, so that timezone is Pacific. `business_date_now` prints the place's wall clock and then drops the offset. On a Pacific place the window is right. On any other place the stored window slides by the Pacific offset, a clock-out one minute after `starts_at` records no payable time, and the transaction task writes nothing. The shared shift format has to include the offset (`%z`). A naive datetime still formats with no offset, so those callers stay on the Pacific default.
- `CurrentTest.api_v2_data` is a client-side ledger, not a server-backed relation cache. A business created with only `company_name` is appended to `businesses`, but its company is not added to `companies`. Before a following step reads the latest company (for example to create a payment source), fetch and append that exact company or use a setup helper that creates the full place/company/business set.
- This swagger view is `public=True`, so DRF builds every view with `request=None`. A `get_serializer_class` that reads `self.request.method` raises `AttributeError` and 500s the whole document. The DecimalField and "not compatible with schema generation" lines above that traceback are warnings. Guard `request` before using it, and fall back to the list serializer when it is missing.
- No trailing slashes. Endpoints 404 with a trailing `/`.
  - Good: `/automation/api/v2/constance-variables/PARTNER_PHONE_VERIFICATION_REQUIRED`
  - Bad:  `/automation/api/v2/constance-variables/PARTNER_PHONE_VERIFICATION_REQUIRED/`
- A 404 HTML page (not JSON) means wrong URL or the backend is not automation-ready.
- `phone-verification/direct-verify` returning `{"error":"User not found"}` is expected without a
  real user, not a failure.
- A local database can have no live terms row that QA already has. `GET /tos` omits `content` (write-only). List QA with the numeric type and country, take the newest result, then read the body from `/legal/{country}/awfs-agreement/{id}/`. `POST /tos` wants the choice label (`AWS Agreement`), not the number.
- If endpoints return 5xx, run migrations/demo data from `instawork/`: `just migrate`,
  `just demodata --sync`.
- For skills-hub mobile flows (Profile -> Add skills -> category -> skill -> phone assessment), the
  Automation API surface has zero `Skill`/`PositionSkill` endpoints (verified against the full QA
  swagger dump: no matching paths or schema defs). This is not a data-setup blocker: `Skill` and
  `PositionSkill` are pre-existing backend/product catalog data, not per-test objects Stage 2 needs
  to create. Treat "a category mapped to an active skill" as already satisfied by QA's real state,
  and let device-session steps observe whichever real first category/skill renders live instead of
  trying to pre-select one via the automation API.
- List-endpoint docstrings that promise "Default ordering -- by creation date" are not proof. Several
  list views expose no ordering and their model has no `Meta.ordering`, so `?page=N` returns an
  unstable slice. Never page-scan such a list to prove a record is absent — a clean scan means
  nothing. Check the view's `filterset_fields` and the model `Meta` before trusting any paged sweep.
- Filtering seeded objects by a human name (business, company) does not narrow anything: factories
  reuse the same names across thousands of rows. To locate leftovers from an earlier seeding loop,
  probe the specific hash IDs that loop created and check the status code per ID. Detail lookups
  usually reject the numeric PK, so keep the hash IDs from each create response for later cleanup.
- Deleting a shift group cascades to its shifts, but a worker assignment record is a separate row
  from the shift's own worker FK. Removing an assignment does not unbook the shift, and an app list
  driven by the shift FK keeps showing it. Delete the owning group to actually clear a booking.
- Shift-group `shareable_link` is not gig details. The pro-app shift details link is
  `instawork://open-gigs/<shift group hash>`.
- `POST /shift-groups` with `worker_id` sets the shift's worker and does not write a Worker Assigned
  log unless `resolved` is also true. My Shifts then 500s in
  `has_lost_top_pro_bonus_from_future_shifts` (`None.created_at`) for a worker who lost Top Pro.
  Add the log with `POST /worker-logs` `{"event": 9, "worker": <id>, "shift": "<shift hash>"}`.
  The shift field accepts the hash even though the schema says integer.
- `GET /gig_template/{id}` omits `company` and `business`. To get the numeric company id after
  creating a shift group, take `contacts[0]` from that template and `GET /company_user?cuser=<id>`.
- Booking-prerequisites attire upload is not a shift-group flag. It needs an attire preset on the
  gig (so an AttireType can be resolved) plus an active `vetting-modules` row with
  `vetting_type=attire_upload` and that company in `rules.companies.include`. Without the module,
  the step still appears but as optional. Grant bookability with
  `POST /shift-groups/{id}/worker_access` and leave the shift unbooked/`is_public=true`.
- **Eligibility Questions** prerequisite: no v2 endpoint to attach `SpecialRequirement` rows. Seed via
  `POST /run-celery-task`: define a setup function in `task_import`, call it from `task_call`.
  `task_call` is `eval()`'d (expression only — not statements). Use
  `SpecialRequirementFactory` + `SpecialRequirementMapping` + `shift_group.special_requirements.add`.
- **Success guide** prerequisite: PATCH `worker-score` with `etiquette_score` below 80 (e.g. 50).
- Legacy automation webhooks can cover setup actions that `/automation/api/v2/swagger` does not
  expose. For orchestrated mobile automation, do not treat those legacy webhook helpers as satisfying
  the Stage 2 API-v2 endpoint coverage gate unless the orchestrator explicitly allows non-v2
  endpoints for that run. A test-scoped missing v2 endpoint remains a data-setup blocker even when
  an existing Behave helper calls a legacy webhook.
- When using `/automation/api/v2/run-celery-task` to pay invoices created by backend helpers, do not
  assume an invoice identifier echoed by a serializer is the task-ready primary key. Resolve the
  invoice inside the backend runtime from the transaction or shift, then pass that object ID to the
  task call.
- `DELETE /companies/{id}` returning HTML 500 usually means a `company-payment-sources` row still
  points at that company — delete the payment source first (soft-deleted rows may still list and
  still block).
- `DELETE /companies/{id}` returning `UserProfile matching query does not exist` (and
  `GET /company_user` 404 with the same detail) means an owner/company-user profile was already
  wiped. Prefer deleting companies before workers. Last resort on a stuck company: peel child
  rows via `POST /run-celery-task` with
  `task_import: from django.db import connection` and
  `task_call: (lambda c: (c.execute("DELETE FROM <child> WHERE company_id=<pk>"), c.rowcount))(connection.cursor())`,
  repeating on each FK table named in the 1451 error, then `DELETE FROM backend_company`.
- `DELETE /workers/{id}` HTML 500 is almost always leftover dependents. Plain
  `DELETE /user-incentives/{id}` can also 500; use `delete-by-worker` with the exact
  `incentive__goal` string from the validator error (e.g. `Instantly earn Top Pro Gold`). The
  swagger query name `campaign__goal` is wrong for that endpoint — the API expects
  `incentive__goal`.

## Fast manual mobile repro loop (skip the full instatest/behave run per iteration)

When iterating on a hypothesis about *why* a mobile UI element/id is missing (not just
reconfirming that it's missing), a full `docker compose run ... instatest run` cycle is slow
(~2 min) per attempt. Faster loop, reusing an already-booted local emulator + Appium server:

1. Create a worker directly and grab its auth token:
   ```bash
   curl -s -X POST http://localhost:8080/automation/api/v2/workers \
     -H "Content-Type: application/json" \
     -d '{"json": {"name": "Debug", "gigs_automation_test_user": true, "given_name": "Debug", "family_name": "Loop", "regionmapping": "bay_area_ca_us", "add_profile_picture": false, "is_internal_user": true}}'
   ```
   Response includes `token` and `token_id`.
2. Build the same authenticated deep link the mobile test suite uses (see
   `mobile/shared_qa_utils/applicant_deep_link.py`, `AuthenticatedDeepLink`): `instawork://<path>?token=<token>&token_id=<token_id>&instawork_url=http://localhost:8080`
   (`<path>` is e.g. `profile`, `earnings`, `open-gigs`; see `DeepLinkTarget` in that file).
3. Launch it with `adb shell`, **single-quoting the deep link inside one command string**:
   ```bash
   CMD="am start -W -a android.intent.action.VIEW -d '${DEEPLINK}' com.instaworkmobile.automation/com.instaworkmobile.MainActivity"
   adb shell "$CMD"
   ```
   Passing the URL as a bare double-quoted shell variable to separate `adb shell` args (`adb shell am start ... -d "$DEEPLINK" ...`) gets it re-tokenized by the device's remote shell, which treats the unescaped `&` in the query string as a background operator and silently truncates the URL after the first param — the intent still "succeeds" (`Status: ok`) but auth fails and the app lands on the logged-out intro screen with no obvious error. Wrapping the whole remote command in one string, with the deep link single-quoted *inside* that string, is what actually protects the `&`s.
4. Drive/inspect with a raw Appium HTTP session (no MCP tool needed): `POST /wd/hub/session` with
   `{"capabilities":{"alwaysMatch":{"platformName":"Android","appium:automationName":"UiAutomator2","appium:udid":"emulator-5554","appium:noReset":true}}}`,
   then repeated `GET /session/:id/source` (JSON-wrapped XML string; parse with `json.load(...)['value']`,
   don't grep the raw file directly — the XML is escaped inside a JSON string) and W3C `POST
   /session/:id/actions` for taps. Reuse one session across many iterations; `noReset: true` avoids
   reinstalling/relogging in every time.
5. **To force a fresh screen fetch (not just bring an already-mounted screen back into focus),
   relaunch the app via the deep link** (`adb shell am force-stop <pkg>` then the intent again).
   Tapping between bottom tab-bar items on an already-running app does not remount the screen (React
   Navigation keeps tab screens mounted), so a template/style edit on the server has no visible
   effect until the screen is actually re-fetched from scratch. This cost a full debugging round
   before being caught — a template change showed "no effect" purely because the test loop was
   switching tabs instead of relaunching.
6. If testing a Django template/Hyperview change: watch `docker logs instawork-backend` for a
   `"<file> template changed, clearing cache."` line to confirm the edit was actually picked up
   before concluding an experiment had no effect (see `instawork/instawork/reloader_template_cache.py`).

## Related

- [web e2e local](../workflows/web-e2e-local.md)
- [mobile android login local](../workflows/mobile-android-login-local.md)
