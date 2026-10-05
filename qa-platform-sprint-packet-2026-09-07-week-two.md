# Week two 2026-09-07

Mission 2: Scale test authoring automation. Completed 12. Carry Over 47. New In Sprint 13. Full Workload 60 (Carry Over + New In Sprint).

| Label | What it counts | Board and column |
| --- | --- | --- |
| Completed | Tasks completed in the last 7 days | Sprint · Done |
| Carry Over | Tasks still unfinished on Sprint, so they continue | Sprint · To-Do, In Progress, In Review |
| New In Sprint | Candidate work for this week, not moved onto Sprint yet | Planning · Needs Triage and Next |
| Full Workload | What the team would hold if every proposed task is accepted | Carry Over + New In Sprint |

## ✅ Completed (last 7 days - 12 tasks)

| Feature | Summary | Tasks |
| --- | --- | --- |
| M6: Automated suite refactor/cleanup > XPath > pipeline [1 task] | Shipped the xpath-to-id agent prompt and pipeline. | ✅ [Create XPath migration prompt / Cursor automation and pipeline](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1216745426906684) [Oscar] |
| M6: Automated suite refactor/cleanup > XPath > locators to IDs [2 tasks] | Login phone/verify and clock-out adjust end time moved to IDs. | ✅ [[XPath] to ID: Login phone and verify](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1218076370443882) [Oscar] |
| | | ✅ [[XPath] to ID: Clock-out adjust end time](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1218076384368429) [Oscar] |
| M6: Automated suite refactor/cleanup > Ruff > instatest gate [5 tasks] | Turned on the lib/instatest gate and cleared T201, E722, F841, W291. | ✅ [Ruff: turn on the lib/instatest gate (config, lockfile, just, CI)](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217915354105478) [Oscar] |
| | | ✅ [[Fix Ruff code T201][Instatest]: print found](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217986225168259) [Oscar] |
| | | ✅ [[Fix Ruff code E722][Instatest]: bare except](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217986491752734) [Oscar] |
| | | ✅ [[Fix Ruff code F841][Instatest]: local variable assigned but never used](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217986491808651) [Oscar] |
| | | ✅ [[Fix Ruff code W291][Instatest]: trailing whitespace](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217986521872191) [Oscar] |
| M6: Automated suite refactor/cleanup > Dead-step > PR lint scanner [1 task] | Scanner and CircleCI lint-e2e landed. | ✅ [Add dead-step scanner script and CircleCI lint-e2e on PR lint](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217005797237873) [Dimpal] |
| M6: Automated suite refactor/cleanup > Sleeps > first cuts [2 tasks] | Prompts written; first 60s QR clock-in sleep removed. | ✅ [[Sleep removals] Classify findings and write reusable agent prompts](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1218092474297854) [Sneha] |
| | | ✅ [[Sleep removals] Business 60s QR clock-in](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1218200609553397) [Sneha] |
| M6: Automated suite refactor/cleanup > Retries > Allure [1 task] | Step retry summary now emits in Allure. | ✅ [[Retry Instrument]— Emit step_retry_summary in Allure](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1218114218675041) [Dimpal] |

## 🔄 Carrying over (47 tasks)

| Feature | Summary | Tasks |
| --- | --- | --- |
| M3: Local/CI parity > local equals CI [1 task] | Moved onto Sprint To-Do. Not started. | [Update local run setup to use the same CI device config](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217984217668643) [Oscar] |
| M6: Automated suite refactor/cleanup > XPath > locators to IDs [7 tasks] | Three xpath-to-id slices in progress; four older locator migrations still To-Do. | [[XPath] to ID: Earnings payment rows](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1218076172277330) [Oscar] |
| | | [[XPath] to ID: Syndicated job cards](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1218076439171301) [Oscar] |
| | | [[XPath] to ID: Payment destination and frequency](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1218076196607536) [Oscar] |
| | | [Migrate short-term work type XPath locator to ID](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217872147853687) [Oscar] |
| | | [Migrate auto-add calendar XPath locator to stable ID](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217835086038617) [Oscar] |
| | | [Migrate microphone granted status XPath to ID](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217810866708400) [Oscar] |
| | | [(Draft to followup) Migrate applicant microphone Prompt XPath to test ID](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217797935621209) [Oscar] |
| M6: Automated suite refactor/cleanup > Ruff > e2e/mobile leftover [14 tasks] | Leftover e2e/mobile codes moved onto Sprint To-Do, including 3.7. | [3.7 Ruff: TID251 replace datetime.now in e2e/mobile](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217843568957130) [Oscar] |
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
| M6: Automated suite refactor/cleanup > Ruff > instatest leftover [12 tasks] | Gate is on; leftover instatest codes still on Sprint To-Do. | [[Fix Ruff code PLW2901][Instatest]: loop variable overwritten inside the loop body](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217986491855925) [Oscar] |
| | | [[Fix Ruff leftover codes][Instatest]: B018, F403, F405, PLW0133, B011, W293, B006, RUF043](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217987206122193) [Oscar] |
| | | [[Fix Ruff code E731][Instatest]: lambda assigned to a variable instead of def](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217986705586463) [Oscar] |
| | | [[Fix Ruff code F811][Instatest]: redefinition of unused name](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217986455926424) [Oscar] |
| | | [[Fix Ruff code B012][Instatest]: jump statement in finally](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217986500344831) [Oscar] |
| | | [[Fix Ruff code E402][Instatest]: module-level import not at top of file](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217986455937988) [Oscar] |
| | | [[Fix Ruff code PLE1205][Instatest]: too many arguments for logging format string](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217986521870586) [Oscar] |
| | | [[Fix Ruff code S113][Instatest]: requests call without a timeout](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217986500336541) [Oscar] |
| | | [[Fix Ruff code PLC0415][Instatest]: import should be at top-level of file](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217986521484304) [Oscar] |
| | | [[Fix Ruff code TID251][Instatest]: banned API / banned module member](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217986521432847) [Oscar] |
| | | [[Fix Ruff code PLW1508][Instatest]: invalid type for environment variable default](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217986499876111) [Oscar] |
| | | [[Fix Ruff code F821][Instatest]: undefined name](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217986705262580) [Oscar] |
| M6: Automated suite refactor/cleanup > Sleeps > remaining [4 tasks] | MDS slices parked in To-Do; My network and Gigs details in review. | [[Sleep removals] MDS permissions](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1218200651200923) [Sneha] |
| | | [[Sleep removals] MDS uploads](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1218200651173131) [Sneha] |
| | | [[Sleep removals] My network](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1218200808034145) [Sneha] |
| | | [[Sleep removals] Gigs details](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1218200609986587) [Sneha] |
| M6: Automated suite refactor/cleanup > Retries > instrumentation [4 tasks] | Allure landed. Redshift ingest in progress. Parent instrument card and removal plan still To-Do. | [[Retry Instrument]— Ingest step retries into Redshift](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1218114218753015) [Dimpal] |
| | | [[Retry Instrument]— Mode chart for step retries](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1218122799955362) [Dimpal] |
| | | [[P4.0] [Remove retries] Instrument step-level @retry so retries are visible](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217559725849406) [Dimpal] |
| | | [[P4.1] Analyze retry signals and write the removal plan](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217559797472024) [Dimpal] |
| M6: Automated suite refactor/cleanup > queued To-Do > fixed-count swipes [4 tasks] | Already on Sprint To-Do. Not started. | [Replace fixed-count swipes in applicant profile-settings features](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217533917199735) [Dimpal] |
| | | [Replace fixed-count swipes in applicant shift and booking features](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217535276201468) [Dimpal] |
| | | [Replace fixed-count swipes in applicant profile-tab features](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217535727744498) [Dimpal] |
| | | [Replace fixed-count swipes in remaining applicant feature flows](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217535727747267) [Dimpal] |
| M6: Automated suite refactor/cleanup > queued To-Do > Gherkin step cleanup [1 task] | Already on Sprint To-Do. Not started. | [[Cleanup] Drop dual an\|a matcher on business account role step](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217507348921000) [Dimpal] |

# 🚀 New this week (13 tasks)

## M1: Backlog foundation

Rank 60 automatable tests (30 mobile + 30 web) so weekly authoring can pick them up.

| Feature | Summary | Tasks |
| --- | --- | --- |
| M1: Backlog foundation > Mobile > validate board entries [4 tasks] | New this week. Planning Needs Triage. | [[Mobile] Validate — Profile / settings, skills & quiz](https://app.asana.com/1/47184964732898/project/1214142796595459/task/1216732429388806) [Antony] |
| | | [[Mobile] Validate — Hiring / interviews & job applications](https://app.asana.com/1/47184964732898/project/1214142796595459/task/1216732429453873) [Antony] |
| | | [[Mobile] Validate — My Shifts / clock-in, provisional & HyperTrack](https://app.asana.com/1/47184964732898/project/1214142796595459/task/1216732268037018) [Antony] |
| | | [[Mobile] Validate — Open Shifts / booking & overbooking](https://app.asana.com/1/47184964732898/project/1214142796595459/task/1216732477337957) [Antony] |
| M1: Backlog foundation > Web > select TCM entries [1 task] | New this week. Planning Needs Triage. | [[Web] Select TCM board entries](https://app.asana.com/1/47184964732898/project/1214142796595459/task/1216735074459861) [Antony] |

## M2: Weekly authoring cadence

Ship about 1 new automated test per QA engineer per week from that backlog.

| Feature | Summary | Tasks |
| --- | --- | --- |
| M2: Weekly authoring cadence > first candidates [2 tasks] | New this week. Planning Next. | [Change password](https://app.asana.com/1/47184964732898/project/1214142796595459/task/1216733962710626) [Oscar] |
| | | [Grooming preset required error](https://app.asana.com/1/47184964732898/project/1214142796595459/task/1216733962592917) [Dimpal] |

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

| Feature | Summary | Tasks |
| --- | --- | --- |
| M6: Automated suite refactor/cleanup > Ruff > instatest format [1 task] | New this week. Planning Needs Triage. | [1. Ruff: format and safe autofix for lib/instatest](https://app.asana.com/1/47184964732898/project/1214142796595459/task/1217925757819299) [Oscar] |
| M6: Automated suite refactor/cleanup > Coordinate clicking > P4 Next [5 tasks] | Out of this sprint. Still on Planning Next; not the spoken plan. | [[P4.2] Migrate DOB picker wheels to NativeDatePicker](https://app.asana.com/1/47184964732898/project/1214142796595459/task/1217788768659415) [Sneha] |
| | | [[P4.3] Replace tap_element_at_center with element.click()](https://app.asana.com/1/47184964732898/project/1214142796595459/task/1217788777590486) [Sneha] |
| | | [[P4.4] Retire _click_on_cordinates (LTA acknowledge, gig end time, captain clock-in)](https://app.asana.com/1/47184964732898/project/1214142796595459/task/1217789055221913) [Sneha] |
| | | [[P4.6] Chat keyboard dismiss via the existing hide-keyboard step](https://app.asana.com/1/47184964732898/project/1214142796595459/task/1217788828515676) [Sneha] |
| | | [[P4.7] iOS action sheets and message-action rows in chat](https://app.asana.com/1/47184964732898/project/1214142796595459/task/1217788732318858) [Sneha] |

## Where we are

Last week: lib/instatest ruff gate on, xpath-to-id pipeline prompt shipped, login phone and clock-out locators moved to IDs, dead-step PR scanner landed, sleep-removal prompts plus first QR clock-in, step retries emitted in Allure.

This week: leftover ruff, retries, fixed-count swipes, and sleep removals. Oscar also keeps three xpath-to-id slices in progress. Coordinate clicking is out of this sprint. Carry-over is 47.

## Blocking

- Retry removal still waits on Redshift ingest and the Mode chart.
- Coordinate clicking is out of this sprint; five related tasks remain on Planning Next but are not the spoken plan.
- Leftover ruff stays in this sprint: push as much progress as the remaining days allow, same as retries, swipes, and sleep removals.
