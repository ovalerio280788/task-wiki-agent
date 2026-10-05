# Kickoff 2026-08-30

Mission 2: Scale test authoring automation. Two-week plan confirmed by Oscar.

## ✅ Completed (ending sprint - 24 tasks)


| Feature                                                                              | Summary                                                                                          | Tasks                                                                                                                                                         |
| ------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| M6: Automated suite refactor/cleanup > XPath > locators to IDs [6 tasks]             | Audited xpath in applicant, business, and shared code; moved 3 locators to stable IDs.           | ✅ [Migrate JobListScreen job search locator to stable ID](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217871156698532) [Oscar]      |
|                                                                                      |                                                                                                  | ✅ [Migrate emergency contact Save locator to stable ID](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217801299354165) [Oscar]        |
|                                                                                      |                                                                                                  | ✅ [Migrate JobDetailsScreen back button locator to stable ID](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217830003651098) [Oscar]  |
|                                                                                      |                                                                                                  | ✅ [Audit XPath usage in applicant-app automation](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1216745081814394) [Oscar]              |
|                                                                                      |                                                                                                  | ✅ [Audit XPath usage in business-app automation](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1216745011136001) [Oscar]               |
|                                                                                      |                                                                                                  | ✅ [Audit XPath usage in e2e/mobile shared code](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1216744903988925) [Oscar]                |
| M6: Automated suite refactor/cleanup > Ruff > e2e/mobile lint gate [10 tasks]        | Formatted e2e/mobile, cleared most lint batches, turned the CI gate on.                          | ✅ [1. Format and safe autofix for e2e/mobile](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217646540372558) [Oscar]                  |
|                                                                                      |                                                                                                  | ✅ [2. Replace star imports (F403/F405)](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217646270070529) [Oscar]                        |
|                                                                                      |                                                                                                  | ✅ [3.1 B011 assert-false](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217796901559148) [Oscar]                                      |
|                                                                                      |                                                                                                  | ✅ [3.2 T201 print](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217797723895892) [Oscar]                                             |
|                                                                                      |                                                                                                  | ✅ [3.3 S113 missing request timeout](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217800454435487) [Oscar]                           |
|                                                                                      |                                                                                                  | ✅ [3.4 F821 undefined name](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217804090255955) [Oscar]                                    |
|                                                                                      |                                                                                                  | ✅ [3.5 INP001 implicit-namespace-package](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217805132348240) [Oscar]                      |
|                                                                                      |                                                                                                  | ✅ [3.6 E722 bare except](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217840603430727) [Oscar]                                       |
|                                                                                      |                                                                                                  | ✅ [3.10 B012](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217844099822072) [Oscar]                                                  |
|                                                                                      |                                                                                                  | ✅ [4. Turn on the e2e/mobile gate](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217646130774451) [Oscar]                             |
| M6: Automated suite refactor/cleanup > Instatest > repo cutover [4 tasks]            | Moved Test Automation into Instawork, cut mobile over, cleaned leftover packaging and dead code. | ✅ [Migrate and locally validate Test Automation in Instawork](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1216738598015561) [Sneha]  |
|                                                                                      |                                                                                                  | ✅ [PR 2 — Cut mobile automation over to lib/instatest/](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217321628380466) [Sneha]        |
|                                                                                      |                                                                                                  | ✅ [Clean up dead code in lib/instatest (post-import)](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217395786652313) [Sneha]          |
|                                                                                      |                                                                                                  | ✅ [1. Remove dead packaging files from lib/instatest](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217774042775378) [Sneha]          |
| M6: Automated suite refactor/cleanup > Coordinate clicking > unused helpers [1 task] | Removed unused coordinate-tap helpers.                                                           | ✅ [P4.1 Delete unused coordinate-tap helpers](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217788726054029) [Sneha]                  |
| M6: Automated suite refactor/cleanup > Retries > dashboard [1 task]                  | Shipped the most-retried-tests dashboard.                                                        | ✅ [Dashboard: most-retried tests](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217559725605915) [Dimpal]                             |
| M6: Automated suite refactor/cleanup > context.execute > remaining cleanup [2 tasks] | Finished remaining keyword-arg and hooks/env cleanup.                                            | ✅ [Follow up Context.execute cleanup](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217516615436766) [Dimpal]                         |
|                                                                                      |                                                                                                  | ✅ [Hooks/env + residual: environment.py / safe_execute_steps](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1216766700124654) [Dimpal] |




## 🔄 Carrying over (19 tasks)


| Feature                                                                                   | Summary                                                                                     | Tasks                                                                                                                                                                     |
| ----------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| M6: Automated suite refactor/cleanup > XPath > locators to IDs [5 tasks]                  | Locator migrations still in progress or in review. Florent's xpath PR is not on this board. | [Migrate short-term work type XPath locator to ID](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217872147853687) [Oscar]                         |
|                                                                                           |                                                                                             | [Migrate auto-add calendar XPath locator to stable ID](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217835086038617) [Oscar]                     |
|                                                                                           |                                                                                             | [Migrate microphone granted status XPath to ID](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217810866708400) [Oscar]                            |
|                                                                                           |                                                                                             | [(Draft to followup) Migrate applicant microphone Prompt XPath to test ID](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217797935621209) [Oscar] |
|                                                                                           |                                                                                             | [Create XPath migration agent prompt](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1216745426906684) [Oscar]                                      |
|                                                                                           |                                                                                             | ❌ Florent in-progress xpath PR (not on this board)                                                                                                                        |
| M6: Automated suite refactor/cleanup > Ruff > remaining lint and instatest gate [5 tasks] | e2e/mobile lint leftover (3.7 in progress, 3.8/3.9/remaining still To-Do); lib/instatest gate not on yet. | [3.7 Ruff: TID251 replace datetime.now in e2e/mobile](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217843568957130) [Oscar] |
|                                                                                           |                                                                                             | [3.8 Ruff: F811 redefined-while-unused in e2e/mobile](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217843834010835) [Oscar]                                  |
|                                                                                           |                                                                                             | [3.9 Ruff: PLW0603 globals in e2e/mobile](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217843874899775) [Oscar]                                  |
|                                                                                           |                                                                                             | [3. Ruff: fix remaining lint findings in e2e/mobile](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217646130844398) [Oscar]                                  |
|                                                                                           |                                                                                             | [4. Ruff: turn on the lib/instatest gate](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217915354105478) [Oscar]                                  |
| M6: Automated suite refactor/cleanup > Dead-step > PR lint scanner [1 task]               | Scanner and CircleCI lint-e2e still in progress.                                            | [Add dead-step scanner script and CircleCI lint-e2e on PR lint](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217005797237873) [Dimpal]           |
| M6: Automated suite refactor/cleanup > Retries > instrumentation [2 tasks]                  | Step-level retry visibility still in progress. Removal plan waits on that.                   | [Instrument step-level @retry so retries are visible](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217559725849406) [Dimpal]                     |
|                                                                                           |                                                                                             | [Analyze retry signals and write the removal plan](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217559797472024) [Dimpal]                     |
| M6: Automated suite refactor/cleanup > Coordinate clicking > NativeDatePicker [1 task]    | DOB picker wheels still on coordinate flow.                                                 | [P4.2 Migrate DOB picker wheels to NativeDatePicker](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217788768659415) [Sneha]                       |
| M6: Automated suite refactor/cleanup > queued To-Do > fixed-count swipes [4 tasks]         | Already on Sprint To-Do. Not started; leftover from the ending sprint, not a new add.          | [Replace fixed-count swipes in applicant profile-settings features](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217533917199735) [Oscar] |
|                                                                                           |                                                                                             | [Replace fixed-count swipes in applicant shift and booking features](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217535276201468) [Oscar] |
|                                                                                           |                                                                                             | [Replace fixed-count swipes in applicant profile-tab features](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217535727744498) [Oscar] |
|                                                                                           |                                                                                             | [Replace fixed-count swipes in remaining applicant feature flows](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217535727747267) [Oscar] |
| M6: Automated suite refactor/cleanup > queued To-Do > dual matcher [1 task]              | Already on Sprint To-Do. Not started; leftover from the ending sprint, not a new add.          | [Drop dual an\|a matcher on business account role step](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217507348921000) [Dimpal] |

# 🚀 New in coming sprint (5 tasks)

## M1: Backlog foundation

Rank 60 automatable tests (30 mobile + 30 web) so weekly authoring can pick them up.

ℹ️ Nothing here for the starting sprint.

## M2: Weekly authoring cadence

Ship about 1 new automated test per QA engineer per week from that backlog.

ℹ️ Nothing here for the starting sprint.

## M3: Local/CI parity

Make local emulator/simulator runs use the same device config as CI.

| Feature | Summary | Tasks |
| --- | --- | --- |
| M3: Local/CI parity > local equals CI [1 task] | New this sprint: same device config as CI. Skin and screen size later. | [Update local run setup to use the same CI device config](https://app.asana.com/1/47184964732898/project/1214142796595459/task/1217984217668643) [Oscar] |

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
| M6: Automated suite refactor/cleanup > Coordinate clicking > P4 Needs Triage [4 tasks] | New this sprint from Planning Needs Triage. Not leftover Sprint work. | [[P4.3] Replace tap_element_at_center with element.click()](https://app.asana.com/1/47184964732898/project/1214142796595459/task/1217788777590486) [Sneha] |
| | | [[P4.4] Retire _click_on_cordinates (LTA acknowledge, gig end time, captain clock-in)](https://app.asana.com/1/47184964732898/project/1214142796595459/task/1217789055221913) [Sneha] |
| | | [[P4.6] Chat keyboard dismiss via the existing hide-keyboard step](https://app.asana.com/1/47184964732898/project/1214142796595459/task/1217788828515676) [Sneha] |
| | | [[P4.7] iOS action sheets and message-action rows in chat](https://app.asana.com/1/47184964732898/project/1214142796595459/task/1217788732318858) [Sneha] |

## Where we are

New adds are the M3 local-equals-CI slice and four M6 P4 coordinate-clicking tasks (all still on Planning Needs Triage). Remaining ruff, xpath, dead-step, retries, swipe To-Do, and P4.2 NativeDatePicker stay in carry-over. M1, M2, M4, and M5 have no new work.

## Blocking

- ❌ Florent xpath PR is not on this board.
- ~20 xpath locators is a stretch against current board volume.
- Retry removal waits on instrumentation.

