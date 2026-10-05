# Week two 2026-09-14

Mission 2: Scale test authoring automation. Completed 16. Carry Over 47. New In Sprint 4. Full Workload 51 (Carry Over + New In Sprint).

| Label | What it counts | Board and column |
| --- | --- | --- |
| Completed | Tasks completed in the last 7 days | Sprint · Done |
| Carry Over | Tasks still unfinished on Sprint, so they continue | Sprint · To-Do, In Progress, In Review |
| New In Sprint | Candidate work for this week, not moved onto Sprint yet | Planning · Needs Triage and Next |
| Full Workload | What the team would hold if every proposed task is accepted | Carry Over + New In Sprint |

## ✅ Completed (last 7 days - 16 tasks)

| Feature | Summary | Tasks |
| --- | --- | --- |
| M6: Automated suite refactor/cleanup > XPath > locators to IDs [10 tasks] | Ten more screens moved off xpath, including earnings, payment destination, and MDS text input. | ✅ [[XPath] to ID: MDS TextField inner /text-input](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1218416862559588) [Oscar] |
| | | ✅ [[XPath] to ID: Payment destination and frequency](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1218076196607536) [Oscar] |
| | | ✅ [[XPath] to ID: Rate a Pro thumbs](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1218076077103939) [Oscar] |
| | | ✅ [[XPath] to ID: Connection list row](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1218076439158629) [Oscar] |
| | | ✅ [[XPath] to ID: Screening question first choice](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1218076384216666) [Oscar] |
| | | ✅ [[XPath] to ID: Open shift cells](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1218076370319964) [Oscar] |
| | | ✅ [[XPath] to ID: Earnings payment rows](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1218076172277330) [Oscar] |
| | | ✅ [[XPath] to ID: Captain chat to pro](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1218076077223882) [Oscar] |
| | | ✅ [[XPath] to ID: Short-term work type switch](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1218076172198998) [Oscar] |
| | | ✅ [[XPath] to ID: Header back](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1218076172255734) [Oscar] |
| M6: Automated suite refactor/cleanup > Ruff > instatest leftover [5 tasks] | Cleared B012, F821, PLE1205, PLW1508, and E731 on the instatest gate. | ✅ [[Fix Ruff code B012][Instatest]: jump statement in finally](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217986500344831) [Oscar] |
| | | ✅ [[Fix Ruff code F821][Instatest]: undefined name](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217986705262580) [Oscar] |
| | | ✅ [[Fix Ruff code PLE1205][Instatest]: too many arguments for logging format string](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217986521870586) [Oscar] |
| | | ✅ [[Fix Ruff code PLW1508][Instatest]: invalid type for environment variable default](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217986499876111) [Oscar] |
| | | ✅ [[Fix Ruff code E731][Instatest]: lambda assigned to a variable instead of def](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217986705586463) [Oscar] |
| M6: Automated suite refactor/cleanup > Sleeps > remaining [1 task] | Shift transfer sleep removed. | ✅ [[Sleep removals] Shift transfer](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1218200775733315) [Sneha] |

## 🔄 Carrying over (47 tasks)

| Feature | Summary | Tasks |
| --- | --- | --- |
| M3: Local/CI parity > local equals CI [1 task] | Still on Sprint To-Do. Not started. | [Update local run setup to use the same CI device config](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217984217668643) [Oscar] |
| M6: Automated suite refactor/cleanup > XPath > locators to IDs [6 tasks] | Syndicated job cards and MDS password/autocomplete in progress; four older locator migrations still To-Do. | [[XPath] to ID: Syndicated job cards](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1218076439171301) [Oscar] |
| | | [[XPath] to ID: MDS PasswordField and AutoCompleteTextField](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1218349334366923) [Oscar] |
| | | [Migrate short-term work type XPath locator to ID](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217872147853687) [Oscar] |
| | | [Migrate auto-add calendar XPath locator to stable ID](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217835086038617) [Oscar] |
| | | [Migrate microphone granted status XPath to ID](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217810866708400) [Oscar] |
| | | [(Draft to followup) Migrate applicant microphone Prompt XPath to test ID](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217797935621209) [Oscar] |
| M6: Automated suite refactor/cleanup > Ruff > e2e/mobile leftover [14 tasks] | Leftover e2e/mobile codes still on Sprint To-Do, including 3.7. | [3.7 Ruff: TID251 replace datetime.now in e2e/mobile](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217843568957130) [Oscar] |
| | | [[Fix Ruff code E712][e2e/mobile]: comparison to True/False instead of truth check](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217988522018360) [Oscar] |
| | | [[Fix Ruff leftover codes][e2e/mobile]: PLW2901, B008, B020 (loop / default-arg)](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217988522069969) [Oscar] |
| | | [[Fix Ruff leftover codes][e2e/mobile]: PLC0414, F401, RUF002, E731, PLW1508 (mechanical cleanup)](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217987764192924) [Oscar] |
| | | [[Fix Ruff leftover codes][e2e/mobile]: PLE1205, G002 (logging)](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217988038535990) [Oscar] |
| | | [[Fix Ruff code B018][e2e/mobile]: useless expression statement](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217987518017407) [Oscar] |
| | | [[Fix Ruff code PLW0127][e2e/mobile]: self-assignment of variable](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217987263770261) [Oscar] |
| | | [[Fix Ruff code PLW0603, PLW0602][e2e/mobile]: global updates and unused global](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217987264009929) [Oscar] |
| | | [[Fix Ruff code PLW0128][e2e/mobile]: name redeclared in the same assignment](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217987701143178) [Oscar] |
| | | [[Fix Ruff code E402][e2e/mobile]: module-level import not at top of file](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217987542033182) [Oscar] |
| | | [[Fix Ruff code I001][e2e/mobile]: import block unsorted or unformatted](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217987517684009) [Oscar] |
| | | [[Fix Ruff code PLC0415][e2e/mobile]: import should be at top-level of file](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217987517684073) [Oscar] |
| | | [[Fix Ruff code F811][e2e/mobile]: redefinition of unused name](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217987542236389) [Oscar] |
| | | [[Fix Ruff code F841][e2e/mobile]: local variable assigned but never used](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217987263656753) [Oscar] |
| M6: Automated suite refactor/cleanup > Ruff > instatest leftover [7 tasks] | Five instatest codes landed last week; seven leftover codes remain, one in progress. | [[Fix Ruff code PLC0415][Instatest]: import should be at top-level of file](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217986521484304) [Oscar] |
| | | [[Fix Ruff code PLW2901][Instatest]: loop variable overwritten inside the loop body](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217986491855925) [Oscar] |
| | | [[Fix Ruff leftover codes][Instatest]: B018, F403, F405, PLW0133, B011, W293, B006, RUF043](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217987206122193) [Oscar] |
| | | [[Fix Ruff code F811][Instatest]: redefinition of unused name](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217986455926424) [Oscar] |
| | | [[Fix Ruff code E402][Instatest]: module-level import not at top of file](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217986455937988) [Oscar] |
| | | [[Fix Ruff code S113][Instatest]: requests call without a timeout](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217986500336541) [Oscar] |
| | | [[Fix Ruff code TID251][Instatest]: banned API / banned module member](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217986521432847) [Oscar] |
| M6: Automated suite refactor/cleanup > Sleeps > remaining [8 tasks] | Five slices in review; gigs list and two MDS slices still To-Do. | [[Sleep removals] MDS permissions](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1218200651200923) [Sneha] |
| | | [[Sleep removals] MDS uploads](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1218200651173131) [Sneha] |
| | | [[Sleep removals] My network](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1218200808034145) [Sneha] |
| | | [[Sleep removals] Gigs details](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1218200609986587) [Sneha] |
| | | [[Sleep removals] Lottie real sleeps](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1218200870758761) [Sneha] |
| | | [[Sleep removals] Gigs list](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1218200610119361) [Sneha] |
| | | [[Sleep removals] MDS address + select + bottomsheet](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1218200651229346) [Sneha] |
| | | [[Sleep removals] MDS avatar uploads](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1218200651211737) [Sneha] |
| M6: Automated suite refactor/cleanup > Retries > instrumentation [4 tasks] | Allure landed earlier. Redshift ingest in progress. Mode chart, parent instrument card, and removal plan still To-Do. | [[Retry Instrument]— Ingest step retries into Redshift](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1218114218753015) [Dimpal] |
| | | [[Retry Instrument]— Mode chart for step retries](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1218122799955362) [Dimpal] |
| | | [[P4.0] [Remove retries] Instrument step-level @retry so retries are visible](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217559725849406) [Dimpal] |
| | | [[P4.1] Analyze retry signals and write the removal plan](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217559797472024) [Dimpal] |
| M6: Automated suite refactor/cleanup > queued To-Do > fixed-count swipes [6 tasks] | Profile-settings and shift/booking in progress. Pull-to-refresh helper in review. Three slices still To-Do. | [Replace fixed-count swipes in applicant profile-settings features](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217533917199735) [Dimpal] |
| | | [Replace fixed-count swipes in applicant shift and booking features](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217535276201468) [Dimpal] |
| | | [Implement Pull to refresh method to replace swipe up count that does pull to refresh](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1218408153060227) [Dimpal] |
| | | [Replace fixed-count swipes in applicant profile-setting MDS features](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1218370876207644) [Dimpal] |
| | | [Replace fixed-count swipes in applicant profile-tab features](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217535727744498) [Dimpal] |
| | | [Replace fixed-count swipes in remaining applicant feature flows](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217535727747267) [Dimpal] |
| M6: Automated suite refactor/cleanup > queued To-Do > Gherkin step cleanup [1 task] | Still on Sprint To-Do. Not started. | [[Cleanup] Drop dual an\|a matcher on business account role step](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217507348921000) [Dimpal] |

# 🚀 New this week (4 tasks)

## M1: Backlog foundation

Rank 60 automatable tests (30 mobile + 30 web) so weekly authoring can pick them up.

| Feature | Summary | Tasks |
| --- | --- | --- |
| M1: Backlog foundation > Mobile > validate board entries [4 tasks] | Out of spoken plan. Still on Planning Needs Triage. | [[Mobile] Validate — Profile / settings, skills & quiz](https://app.asana.com/1/47184964732898/project/1214142796595459/task/1216732429388806) [Antony] |
| | | [[Mobile] Validate — Hiring / interviews & job applications](https://app.asana.com/1/47184964732898/project/1214142796595459/task/1216732429453873) [Antony] |
| | | [[Mobile] Validate — My Shifts / clock-in, provisional & HyperTrack](https://app.asana.com/1/47184964732898/project/1214142796595459/task/1216732268037018) [Antony] |
| | | [[Mobile] Validate — Open Shifts / booking & overbooking](https://app.asana.com/1/47184964732898/project/1214142796595459/task/1216732477337957) [Antony] |

## M2: Weekly authoring cadence

Ship about 1 new automated test per QA engineer per week from that backlog.

ℹ️ Nothing here for this week.

## M3: Local/CI parity

Make local emulator/simulator runs use the same device config as CI.

ℹ️ Nothing here for this week.

## M4: Skills rollout

Teach engineers to add E2E coverage with the authoring skills so QA does not own every test.

ℹ️ Nothing here for this week.

## M5: Cloud agent readiness

Let cloud agents author and validate tests without a local machine.

ℹ️ Nothing here for this week.

## M6: Automated suite refactor/cleanup

Cut suite cost: XPath to stable IDs, ruff/instatest, drop duplicate coverage.

ℹ️ Nothing here for this week.

## Where we are

Last week: ten more xpath-to-id slices, five leftover instatest ruff codes, and shift-transfer sleep removal. Syndicated job cards and MDS password fields are still in progress.

This week: finish those xpath slices, leftover ruff, sleep removals in review, swipe replacement, and retry ingest. Mobile backlog validation stays on Planning Needs Triage but is not the spoken plan. Coordinate clicking is off Planning Next. Carry-over is 47.

## Blocking

- Retry removal still waits on Redshift ingest (schema was redesigned after duplicate-row issues) and the Mode chart.
- Local/CI device match is still parked on Sprint To-Do.
- Coordinate clicking stays out of this sprint.
