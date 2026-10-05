# Kickoff 2026-08-31

Mission 2: Scale test authoring automation. Completed 24. Carry Over 16. New In Sprint 38. Full Workload 54 (Carry Over + New In Sprint).

| Label | What it counts | Board and column |
| --- | --- | --- |
| Completed | Tasks completed during the sprint that is ending | Sprint · Done |
| Carry Over | Tasks left unfinished by the ending sprint, so they continue | Sprint · To-Do, In Progress, In Review |
| New In Sprint | Candidate work for the coming sprint, not moved onto Sprint yet | Planning · Needs Triage and Next |
| Full Workload | What the team would hold if every proposed task is accepted | Carry Over + New In Sprint |

## ✅ Completed (ending sprint - 24 tasks)

| Feature | Summary | Tasks |
| --- | --- | --- |
| M6: Automated suite refactor/cleanup > XPath > locators to IDs [6 tasks] | Audited xpath in applicant, business, and shared code; moved 3 locators to stable IDs. | ✅ [Migrate JobListScreen job search locator to stable ID](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217871156698532) [Oscar] |
| | | ✅ [Migrate emergency contact Save locator to stable ID](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217801299354165) [Oscar] |
| | | ✅ [Migrate JobDetailsScreen back button locator to stable ID](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217830003651098) [Oscar] |
| | | ✅ [Audit XPath usage in applicant-app automation](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1216745081814394) [Oscar] |
| | | ✅ [Audit XPath usage in business-app automation](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1216745011136001) [Oscar] |
| | | ✅ [Audit XPath usage in e2e/mobile shared code](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1216744903988925) [Oscar] |
| M6: Automated suite refactor/cleanup > Ruff > e2e/mobile lint gate [10 tasks] | Formatted e2e/mobile, cleared most lint batches, turned the CI gate on. | ✅ [1. Format and safe autofix for e2e/mobile](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217646540372558) [Oscar] |
| | | ✅ [2. Replace star imports (F403/F405)](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217646270070529) [Oscar] |
| | | ✅ [3.1 B011 assert-false](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217796901559148) [Oscar] |
| | | ✅ [3.2 T201 print](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217797723895892) [Oscar] |
| | | ✅ [3.3 S113 missing request timeout](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217800454435487) [Oscar] |
| | | ✅ [3.4 F821 undefined name](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217804090255955) [Oscar] |
| | | ✅ [3.5 INP001 implicit-namespace-package](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217805132348240) [Oscar] |
| | | ✅ [3.6 E722 bare except](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217840603430727) [Oscar] |
| | | ✅ [3.10 B012](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217844099822072) [Oscar] |
| | | ✅ [4. Turn on the e2e/mobile gate](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217646130774451) [Oscar] |
| M6: Automated suite refactor/cleanup > Instatest > repo cutover [4 tasks] | Moved Test Automation into Instawork, cut mobile over, cleaned leftover packaging and dead code. | ✅ [Migrate and locally validate Test Automation in Instawork](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1216738598015561) [Sneha] |
| | | ✅ [PR 2 — Cut mobile automation over to lib/instatest/](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217321628380466) [Sneha] |
| | | ✅ [Clean up dead code in lib/instatest (post-import)](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217395786652313) [Sneha] |
| | | ✅ [1. Remove dead packaging files from lib/instatest](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217774042775378) [Sneha] |
| M6: Automated suite refactor/cleanup > Coordinate clicking > unused helpers [1 task] | Removed unused coordinate-tap helpers. | ✅ [P4.1 Delete unused coordinate-tap helpers](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217788726054029) [Sneha] |
| M6: Automated suite refactor/cleanup > Retries > dashboard [1 task] | Shipped the most-retried-tests dashboard. | ✅ [Dashboard: most-retried tests](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217559725605915) [Dimpal] |
| M6: Automated suite refactor/cleanup > context.execute > remaining cleanup [2 tasks] | Finished remaining keyword-arg and hooks/env cleanup. | ✅ [Follow up Context.execute cleanup](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217516615436766) [Dimpal] |
| | | ✅ [Hooks/env + residual: environment.py / safe_execute_steps](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1216766700124654) [Dimpal] |

## 🔄 Carrying over (16 tasks)

| Feature | Summary | Tasks |
| --- | --- | --- |
| M6: Automated suite refactor/cleanup > XPath > locators to IDs [5 tasks] | Locator migrations still in progress or in review. Florent's xpath PR is not on this board. | [Migrate short-term work type XPath locator to ID](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217872147853687) [Oscar] |
| | | [Migrate auto-add calendar XPath locator to stable ID](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217835086038617) [Oscar] |
| | | [Migrate microphone granted status XPath to ID](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217810866708400) [Oscar] |
| | | [(Draft to followup) Migrate applicant microphone Prompt XPath to test ID](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217797935621209) [Oscar] |
| | | [Create XPath migration agent prompt / Cursor automation and pipeline](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1216745426906684) [Oscar] |
| | | ❌ Florent in-progress xpath PR (not on this board) |
| M6: Automated suite refactor/cleanup > Ruff > remaining lint and instatest gate [2 tasks] | e2e/mobile leftover 3.7 still in progress; lib/instatest gate now in progress. | [3.7 Ruff: TID251 replace datetime.now in e2e/mobile](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217843568957130) [Oscar] |
| | | [Ruff: turn on the lib/instatest gate (config, lockfile, just, CI)](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217915354105478) [Oscar] |
| M6: Automated suite refactor/cleanup > Dead-step > PR lint scanner [1 task] | Scanner and CircleCI lint-e2e still in progress. | [Add dead-step scanner script and CircleCI lint-e2e on PR lint](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217005797237873) [Dimpal] |
| M6: Automated suite refactor/cleanup > Retries > instrumentation [2 tasks] | Step-level retry visibility still in progress. Removal plan is back in To-Do. | [Instrument step-level @retry so retries are visible](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217559725849406) [Dimpal] |
| | | [Analyze retry signals and write the removal plan](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217559797472024) [Dimpal] |
| M6: Automated suite refactor/cleanup > Coordinate clicking > NativeDatePicker [1 task] | DOB picker wheels still on coordinate flow. | [P4.2 Migrate DOB picker wheels to NativeDatePicker](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217788768659415) [Sneha] |
| M6: Automated suite refactor/cleanup > queued To-Do > fixed-count swipes [4 tasks] | Already on Sprint To-Do. Not started; leftover from the ending sprint, not a new add. | [Replace fixed-count swipes in applicant profile-settings features](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217533917199735) [Dimpal] |
| | | [Replace fixed-count swipes in applicant shift and booking features](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217535276201468) [Dimpal] |
| | | [Replace fixed-count swipes in applicant profile-tab features](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217535727744498) [Dimpal] |
| | | [Replace fixed-count swipes in remaining applicant feature flows](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217535727747267) [Dimpal] |
| M6: Automated suite refactor/cleanup > queued To-Do > dual matcher [1 task] | Already on Sprint To-Do. Not started; leftover from the ending sprint, not a new add. | [Drop dual an\|a matcher on business account role step](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217507348921000) [Dimpal] |

# 🚀 New in coming sprint (38 tasks)

## M1: Backlog foundation

Rank 60 automatable tests (30 mobile + 30 web) so weekly authoring can pick them up.

| Feature | Summary | Tasks |
| --- | --- | --- |
| M1: Backlog foundation > Mobile > validate board entries [4 tasks] | New this sprint. Planning Needs Triage. | [[Mobile] Validate — Profile / settings, skills & quiz](https://app.asana.com/1/47184964732898/project/1214142796595459/task/1216732429388806) [Antony] |
| | | [[Mobile] Validate — Hiring / interviews & job applications](https://app.asana.com/1/47184964732898/project/1214142796595459/task/1216732429453873) [Antony] |
| | | [[Mobile] Validate — My Shifts / clock-in, provisional & HyperTrack](https://app.asana.com/1/47184964732898/project/1214142796595459/task/1216732268037018) [Antony] |
| | | [[Mobile] Validate — Open Shifts / booking & overbooking](https://app.asana.com/1/47184964732898/project/1214142796595459/task/1216732477337957) [Antony] |

## M2: Weekly authoring cadence

Ship about 1 new automated test per QA engineer per week from that backlog.

ℹ️ Nothing here for the starting sprint.

## M3: Local/CI parity

Make local emulator/simulator runs use the same device config as CI.

| Feature | Summary | Tasks |
| --- | --- | --- |
| M3: Local/CI parity > local equals CI [1 task] | New this sprint. Planning Needs Triage. Skin and screen size later. | [Update local run setup to use the same CI device config](https://app.asana.com/1/47184964732898/project/1214142796595459/task/1217984217668643) [Oscar] |

## M4: Skills rollout

Teach engineers to add E2E coverage with the authoring skills so QA does not own every test.

ℹ️ Nothing here for the starting sprint.

## M5: Cloud agent readiness

Let cloud agents author and validate tests without a local machine.

ℹ️ Nothing here for the starting sprint.

## M6: Automated suite refactor/cleanup

Cut suite cost: XPath to stable IDs, ruff/instatest, drop duplicate coverage.

| Feature | Summary | Tasks |
| --- | --- | --- |
| M6: Automated suite refactor/cleanup > Coordinate clicking > P4 Needs Triage [4 tasks] | New this sprint. Planning Needs Triage. | [[P4.3] Replace tap_element_at_center with element.click()](https://app.asana.com/1/47184964732898/project/1214142796595459/task/1217788777590486) [Sneha] |
| | | [[P4.4] Retire _click_on_cordinates (LTA acknowledge, gig end time, captain clock-in)](https://app.asana.com/1/47184964732898/project/1214142796595459/task/1217789055221913) [Sneha] |
| | | [[P4.6] Chat keyboard dismiss via the existing hide-keyboard step](https://app.asana.com/1/47184964732898/project/1214142796595459/task/1217788828515676) [Sneha] |
| | | [[P4.7] iOS action sheets and message-action rows in chat](https://app.asana.com/1/47184964732898/project/1214142796595459/task/1217788732318858) [Sneha] |
| M6: Automated suite refactor/cleanup > Ruff > e2e/mobile leftover [13 tasks] | New this sprint. Planning Next. Leftover codes after the e2e/mobile gate. | [[Fix Ruff code E712][e2e/mobile]: comparison to True/False instead of truth check](https://app.asana.com/1/47184964732898/project/1214142796595459/task/1217988522018360) [Oscar] |
| | | [[Fix Ruff leftover codes][e2e/mobile]: PLW2901, B008, B020 (loop / default-arg)](https://app.asana.com/1/47184964732898/project/1214142796595459/task/1217988522069969) [Oscar] |
| | | [[Fix Ruff leftover codes][e2e/mobile]: PLC0414, F401, RUF002, E731, PLW1508 (mechanical cleanup)](https://app.asana.com/1/47184964732898/project/1214142796595459/task/1217987764192924) [Oscar] |
| | | [[Fix Ruff leftover codes][e2e/mobile]: PLE1205, G002 (logging)](https://app.asana.com/1/47184964732898/project/1214142796595459/task/1217988038535990) [Oscar] |
| | | [[Fix Ruff code B018][e2e/mobile]: useless expression statement](https://app.asana.com/1/47184964732898/project/1214142796595459/task/1217987518017407) [Oscar] |
| | | [[Fix Ruff code PLW0127][e2e/mobile]: self-assignment of variable](https://app.asana.com/1/47184964732898/project/1214142796595459/task/1217987263770261) [Oscar] |
| | | [[Fix Ruff code PLW0603, PLW0602][e2e/mobile]: global updates and unused global](https://app.asana.com/1/47184964732898/project/1214142796595459/task/1217987264009929) [Oscar] |
| | | [[Fix Ruff code PLW0128][e2e/mobile]: name redeclared in the same assignment](https://app.asana.com/1/47184964732898/project/1214142796595459/task/1217987701143178) [Oscar] |
| | | [[Fix Ruff code E402][e2e/mobile]: module-level import not at top of file](https://app.asana.com/1/47184964732898/project/1214142796595459/task/1217987542033182) [Oscar] |
| | | [[Fix Ruff code I001][e2e/mobile]: import block unsorted or unformatted](https://app.asana.com/1/47184964732898/project/1214142796595459/task/1217987517684009) [Oscar] |
| | | [[Fix Ruff code PLC0415][e2e/mobile]: import should be at top-level of file](https://app.asana.com/1/47184964732898/project/1214142796595459/task/1217987517684073) [Oscar] |
| | | [[Fix Ruff code F811][e2e/mobile]: redefinition of unused name](https://app.asana.com/1/47184964732898/project/1214142796595459/task/1217987542236389) [Oscar] |
| | | [[Fix Ruff code F841][e2e/mobile]: local variable assigned but never used](https://app.asana.com/1/47184964732898/project/1214142796595459/task/1217987263656753) [Oscar] |
| M6: Automated suite refactor/cleanup > Ruff > instatest leftover [16 tasks] | New this sprint. Planning Next. Leftover codes before the lib/instatest gate. | [[Fix Ruff code T201][Instatest]: print found](https://app.asana.com/1/47184964732898/project/1214142796595459/task/1217986225168259) [Oscar] |
| | | [[Fix Ruff code F841][Instatest]: local variable assigned but never used](https://app.asana.com/1/47184964732898/project/1214142796595459/task/1217986491808651) [Oscar] |
| | | [[Fix Ruff code PLW2901][Instatest]: loop variable overwritten inside the loop body](https://app.asana.com/1/47184964732898/project/1214142796595459/task/1217986491855925) [Oscar] |
| | | [[Fix Ruff code E722][Instatest]: bare except](https://app.asana.com/1/47184964732898/project/1214142796595459/task/1217986491752734) [Oscar] |
| | | [[Fix Ruff leftover codes][Instatest]: B018, F403, F405, PLW0133, B011, W293, B006, RUF043](https://app.asana.com/1/47184964732898/project/1214142796595459/task/1217987206122193) [Oscar] |
| | | [[Fix Ruff code E731][Instatest]: lambda assigned to a variable instead of def](https://app.asana.com/1/47184964732898/project/1214142796595459/task/1217986705586463) [Oscar] |
| | | [[Fix Ruff code F811][Instatest]: redefinition of unused name](https://app.asana.com/1/47184964732898/project/1214142796595459/task/1217986455926424) [Oscar] |
| | | [[Fix Ruff code B012][Instatest]: jump statement in finally](https://app.asana.com/1/47184964732898/project/1214142796595459/task/1217986500344831) [Oscar] |
| | | [[Fix Ruff code W291][Instatest]: trailing whitespace](https://app.asana.com/1/47184964732898/project/1214142796595459/task/1217986521872191) [Oscar] |
| | | [[Fix Ruff code E402][Instatest]: module-level import not at top of file](https://app.asana.com/1/47184964732898/project/1214142796595459/task/1217986455937988) [Oscar] |
| | | [[Fix Ruff code PLE1205][Instatest]: too many arguments for logging format string](https://app.asana.com/1/47184964732898/project/1214142796595459/task/1217986521870586) [Oscar] |
| | | [[Fix Ruff code S113][Instatest]: requests call without a timeout](https://app.asana.com/1/47184964732898/project/1214142796595459/task/1217986500336541) [Oscar] |
| | | [[Fix Ruff code PLC0415][Instatest]: import should be at top-level of file](https://app.asana.com/1/47184964732898/project/1214142796595459/task/1217986521484304) [Oscar] |
| | | [[Fix Ruff code TID251][Instatest]: banned API / banned module member](https://app.asana.com/1/47184964732898/project/1214142796595459/task/1217986521432847) [Oscar] |
| | | [[Fix Ruff code PLW1508][Instatest]: invalid type for environment variable default](https://app.asana.com/1/47184964732898/project/1214142796595459/task/1217986499876111) [Oscar] |
| | | [[Fix Ruff code F821][Instatest]: undefined name](https://app.asana.com/1/47184964732898/project/1214142796595459/task/1217986705262580) [Oscar] |

## Where we are

New work is 38 Planning tasks under Mission 2: four M1 mobile backlog validates, M3 local-equals-CI, four M6 P4 coordinate slices, and 29 leftover ruff codes on Next (13 e2e/mobile, 16 lib/instatest). Carry-over stays 16 on Sprint. M2, M4, and M5 have no Planning Next or Needs Triage work.

## Blocking

- ❌ Florent xpath PR is not on this board.
- ~20 xpath locators is a stretch against current board volume.
- Retry removal waits on instrumentation.
