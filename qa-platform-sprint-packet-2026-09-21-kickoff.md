# Kickoff 2026-09-21

Mission 2: Scale test authoring automation. Completed 32. Carry Over 39. New In Sprint 14. Full Workload 53 (Carry Over + New In Sprint).

| Label | What it counts | Board and column |
| --- | --- | --- |
| Completed | Tasks completed during the sprint that is ending (after the 2026-09-07 kickoff) | Sprint · Done |
| Carry Over | Tasks left unfinished by the ending sprint, so they continue | Sprint · To-Do, In Progress, In Review |
| New In Sprint | Candidate work for the coming sprint, not moved onto Sprint yet | Planning · Needs Triage and Next |
| Full Workload | What the team would hold if every proposed task is accepted | Carry Over + New In Sprint |

Counting note: 5 of the 32 completed tasks are sleep removals that are marked complete but still parked in Sprint · In Review. They are counted as Completed, not Carry Over. See Blocking.

## ✅ Completed (ending sprint - 32 tasks)

| Feature | Summary | Tasks |
| --- | --- | --- |
| M6: Automated suite refactor/cleanup > XPath > locators to IDs [16 tasks] | The bulk of the sprint. Sixteen locator families moved off XPath onto stable IDs, including the MDS field components and the Hyperview picker spike. | ✅ [[XPath] to ID: Staffing agency search text-input](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1218556490445016) [Oscar] |
| | | ✅ [[XPath] to ID: MDS PickerField](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1218499628889261) [Oscar] |
| | | ✅ [Spike: Hyperview picker testID on the inner component](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1218556214112547) [Oscar] |
| | | ✅ [[XPath] to ID: Work experience company rows](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1218529954057850) [Oscar] |
| | | ✅ [[XPath] to ID: MDS DateField](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1218494572850406) [Oscar] |
| | | ✅ [[XPath] to ID: MDS PasswordField and AutoCompleteTextField](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1218349334366923) [Oscar] |
| | | ✅ [[XPath] to ID: MDS TextField inner /text-input](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1218416862559588) [Oscar] |
| | | ✅ [[XPath] to ID: Payment destination and frequency](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1218076196607536) [Oscar] |
| | | ✅ [[XPath] to ID: Rate a Pro thumbs](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1218076077103939) [Oscar] |
| | | ✅ [[XPath] to ID: Connection list row](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1218076439158629) [Oscar] |
| | | ✅ [[XPath] to ID: Screening question first choice](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1218076384216666) [Oscar] |
| | | ✅ [[XPath] to ID: Open shift cells](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1218076370319964) [Oscar] |
| | | ✅ [[XPath] to ID: Earnings payment rows](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1218076172277330) [Oscar] |
| | | ✅ [[XPath] to ID: Captain chat to pro](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1218076077223882) [Oscar] |
| | | ✅ [[XPath] to ID: Short-term work type switch](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1218076172198998) [Oscar] |
| | | ✅ [[XPath] to ID: Header back](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1218076172255734) [Oscar] |
| M6: Automated suite refactor/cleanup > Ruff > instatest leftover [5 tasks] | Five more instatest codes cleared behind the gate that was turned on last sprint. | ✅ [[Fix Ruff code E731][Instatest]: lambda assigned to a variable instead of def](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217986705586463) [Oscar] |
| | | ✅ [[Fix Ruff code PLW1508][Instatest]: invalid type for environment variable default](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217986499876111) [Oscar] |
| | | ✅ [[Fix Ruff code PLE1205][Instatest]: too many arguments for logging format string](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217986521870586) [Oscar] |
| | | ✅ [[Fix Ruff code F821][Instatest]: undefined name](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217986705262580) [Oscar] |
| | | ✅ [[Fix Ruff code B012][Instatest]: jump statement in finally](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217986500344831) [Oscar] |
| M6: Automated suite refactor/cleanup > Sleeps [5 tasks] | Shift transfer landed. Four more finished on 09-18 but are still sitting in In Review. | ✅ [[Sleep removals] Shift transfer](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1218200775733315) [Sneha] |
| | | ✅ [[Sleep removals] Lottie real sleeps](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1218200870758761) [Sneha] |
| | | ✅ [[Sleep removals] My network](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1218200808034145) [Sneha] |
| | | ✅ [[Sleep removals] Payment settings](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1218200870848751) [Sneha] |
| | | ✅ [[Sleep removals] MDS permissions](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1218200651200923) [Sneha] |
| M6: Automated suite refactor/cleanup > Swipes [4 tasks] | Three swipe slices replaced plus the reusable pull-to-refresh method they all depend on. | ✅ [Replace fixed-count swipes in applicant profile-setting MDS features](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1218370876207644) [Dimpal] |
| | | ✅ [Replace fixed-count swipes in applicant shift and booking features](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217535276201468) [Dimpal] |
| | | ✅ [Replace fixed-count swipes in applicant profile-settings features](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217533917199735) [Dimpal] |
| | | ✅ [Implement Pull to refresh method to replace swipe up count](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1218408153060227) [Dimpal] |
| M6: Automated suite refactor/cleanup > Retries > instrumentation [2 tasks] | Retry data now emits in Allure and lands in Redshift. The measurement half of retry removal is done. | ✅ [[Retry Instrument]— Ingest step retries into Redshift](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1218114218753015) [Dimpal] |
| | | ✅ [[Retry Instrument]— Emit step_retry_summary in Allure](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1218114218675041) [Dimpal] |

## 🔄 Carrying over (39 tasks)

| Feature | Summary | Tasks |
| --- | --- | --- |
| M3: Local/CI parity > local equals CI [1 task] | Still parked on Sprint To-Do. Untouched for two sprints. | [Update local run setup to use the same CI device config](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217984217668643) [Oscar] |
| M6: Automated suite refactor/cleanup > XPath > locators to IDs [6 tasks] | Sendbird row locator is in review behind a PR pair. Five locator cards remain on To-Do. | [Locate Sendbird Chats rows by channel url](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1218649045394234) [Oscar] |
| | | [[XPath] to ID: Syndicated job cards](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1218076439171301) [Oscar] |
| | | [Migrate auto-add calendar XPath locator to stable ID](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217835086038617) [Oscar] |
| | | [Migrate short-term work type XPath locator to ID](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217872147853687) [Oscar] |
| | | [Migrate microphone granted status XPath to ID](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217810866708400) [Oscar] |
| | | [(Draft to followup) Migrate applicant microphone Prompt XPath to test ID](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217797935621209) [Oscar] |
| M6: Automated suite refactor/cleanup > Ruff > e2e/mobile leftover [14 tasks] | Untouched this sprint. All fourteen still on Sprint To-Do. | [3.7 Ruff: TID251 replace datetime.now in e2e/mobile](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217843568957130) [Oscar] |
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
| M6: Automated suite refactor/cleanup > Ruff > instatest leftover [7 tasks] | Down from 12 to 7. PLC0415 is in progress. | [[Fix Ruff code PLC0415][Instatest]: import should be at top-level of file](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217986521484304) [Oscar] |
| | | [[Fix Ruff code PLW2901][Instatest]: loop variable overwritten inside the loop body](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217986491855925) [Oscar] |
| | | [[Fix Ruff leftover codes][Instatest]: B018, F403, F405, PLW0133, B011, W293, B006, RUF043](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217987206122193) [Oscar] |
| | | [[Fix Ruff code F811][Instatest]: redefinition of unused name](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217986455926424) [Oscar] |
| | | [[Fix Ruff code E402][Instatest]: module-level import not at top of file](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217986455937988) [Oscar] |
| | | [[Fix Ruff code S113][Instatest]: requests call without a timeout](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217986500336541) [Oscar] |
| | | [[Fix Ruff code TID251][Instatest]: banned API / banned module member](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217986521432847) [Oscar] |
| M6: Automated suite refactor/cleanup > Sleeps > in review [4 tasks] | Open review queue. Not yet merged. | [[Sleep removals] MDS avatar uploads](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1218200651211737) [Sneha] |
| | | [[Sleep removals] MDS uploads](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1218200651173131) [Sneha] |
| | | [[Sleep removals] Gigs details](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1218200609986587) [Sneha] |
| | | [[Sleep removals] Gigs list](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1218200610119361) [Sneha] |
| M6: Automated suite refactor/cleanup > Retries > removal path [3 tasks] | Instrumentation is done; the chart and the removal plan are what gate every removal card. | [[Retry Instrument]— Mode chart for step retries](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1218122799955362) [Dimpal] |
| | | [[P4.1] Analyze retry signals and write the removal plan](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217559797472024) [Dimpal] |
| | | [[P4.0] [Remove retries] Instrument step-level @retry so retries are visible](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217559725849406) [Dimpal] |
| M6: Automated suite refactor/cleanup > Swipes > in review [2 tasks] | Last two swipe slices awaiting review. | [Replace fixed-count swipes in remaining applicant feature flows](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217535727747267) [Dimpal] |
| | | [Replace fixed-count swipes in applicant profile-tab features](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217535727744498) [Dimpal] |
| M6: Automated suite refactor/cleanup > screen-ready assertion [1 task] | In progress. Unblocks the sleep removals that currently have nothing to wait on. | [[P3] Make the 'screen should be loaded' step actually assert (or remove it)](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1216772040011133) [Oscar] |
| M6: Automated suite refactor/cleanup > Gherkin step cleanup [1 task] | Still on Sprint To-Do. Not started, two sprints running. | [[Cleanup] Drop dual an\|a matcher on business account role step](https://app.asana.com/1/47184964732898/project/1211863943364586/task/1217507348921000) [Dimpal] |

# 🚀 New in coming sprint (14 tasks)

## M1: Backlog foundation

Rank 60 automatable tests (30 mobile + 30 web) so weekly authoring can pick them up.

| Feature | Summary | Tasks |
| --- | --- | --- |
| M1: Backlog foundation > Mobile > validate board entries [4 tasks] | Carried on Planning Needs Triage since 09-07 without being picked up. | [[Mobile] Validate — Profile / settings, skills & quiz](https://app.asana.com/1/47184964732898/project/1214142796595459/task/1216732429388806) [Antony] |
| | | [[Mobile] Validate — Hiring / interviews & job applications](https://app.asana.com/1/47184964732898/project/1214142796595459/task/1216732429453873) [Antony] |
| | | [[Mobile] Validate — My Shifts / clock-in, provisional & HyperTrack](https://app.asana.com/1/47184964732898/project/1214142796595459/task/1216732268037018) [Antony] |
| | | [[Mobile] Validate — Open Shifts / booking & overbooking](https://app.asana.com/1/47184964732898/project/1214142796595459/task/1216732477337957) [Antony] |

## M2: Weekly authoring cadence

Ship about 1 new automated test per QA engineer per week from that backlog.

ℹ️ Nothing here for the starting sprint. The two candidates the 09-07 packet listed were already completed in July.

## M3: Local/CI parity

Make local emulator/simulator runs use the same device config as CI.

ℹ️ Nothing new. One card carries over on Sprint To-Do.

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
| M6: Automated suite refactor/cleanup > Sleeps > next slice [10 tasks] | Sneha's stated plan for the coming sprint: pull these ten off Planning Next. | [[Sleep removals] MDS address + select + bottomsheet](https://app.asana.com/1/47184964732898/project/1214142796595459/task/1218200651229346) [Sneha] |
| | | [[Sleep removals] MDS Python helpers](https://app.asana.com/1/47184964732898/project/1214142796595459/task/1218200610207808) [Sneha] |
| | | [[Sleep removals] Lottie I wait for N seconds (implicitly_wait)](https://app.asana.com/1/47184964732898/project/1214142796595459/task/1218200610270585) [Sneha] |
| | | [[Sleep removals] Business early clock-out](https://app.asana.com/1/47184964732898/project/1214142796595459/task/1218200775800411) [Sneha] |
| | | [[Sleep removals] Business Python](https://app.asana.com/1/47184964732898/project/1214142796595459/task/1218200775829031) [Sneha] |
| | | [[Sleep removals] Hiring + cancel + deactivation](https://app.asana.com/1/47184964732898/project/1214142796595459/task/1218200651277805) [Sneha] |
| | | [[Sleep removals] Profile satellite steps](https://app.asana.com/1/47184964732898/project/1214142796595459/task/1218200651329550) [Sneha] |
| | | [[Sleep removals] Payment + settings Python](https://app.asana.com/1/47184964732898/project/1214142796595459/task/1218200610175424) [Sneha] |
| | | [[Sleep removals] Profile tab (except My network)](https://app.asana.com/1/47184964732898/project/1214142796595459/task/1218200775705968) [Sneha] |
| | | [[Sleep removals] Hiring + interview Python](https://app.asana.com/1/47184964732898/project/1214142796595459/task/1218200870905486) [Sneha] |

## In flight outside the board

Oscar's resource-id migration runs as three integration branches plus two follow-up PR pairs. None of this has its own Asana card beyond the XPath parent.

| Work | PRs | Status |
| --- | --- | --- |
| Integration: testID → resource_id | [instawork#47197](https://github.com/Instawork/instawork/pull/47197), [mobile#8588](https://github.com/Instawork/mobile/pull/8588), [hyperview#1312](https://github.com/Instawork/hyperview/pull/1312) | mobile and hyperview approved, hyperview checks green. instawork is draft with automation and lint red. Android automation still red on mobile. |
| Sendbird chat row locator | [instawork#47643](https://github.com/Instawork/instawork/pull/47643), [mobile#8686](https://github.com/Instawork/mobile/pull/8686) | Both open. lint-e2e red on instawork, Android automation red on mobile. Fixes roughly 4 tests. |
| Hyperview picker field | [hyperview#1353](https://github.com/Instawork/hyperview/pull/1353), [hyperview#1354](https://github.com/Instawork/hyperview/pull/1354) | Both draft, checks green. Exposes the selected label as `{id}/field-label`. |

Reported suite state on the integration branch is about 94% passing with roughly 27 tests still failing, not all of them resource-id root cause. This number is user-reported and could not be verified: the CircleCI Allure artifact URL returns 404.

## Where we are

Mission 2: add automated coverage faster without QA owning every test.

Last sprint landed:

- 16 locator families off XPath onto stable IDs
- 5 more ruff codes cleared
- 5 arbitrary sleeps removed
- 3 swipe slices plus a reusable pull-to-refresh method
- Step retries now land in Allure and Redshift

This sprint:

- Oscar: finish resource-id (~6% of tests still to review), make screen-loaded actually assert, then more XPath and ruff
- Sneha: 10 more sleep-removal slices
- Dimpal: retry dashboard, then a removal plan, plus remaining swipes
- Parked: coordinate clicking, BrowserStack ranking
- Still not started: local/CI parity (third sprint)
- Carry over is 39; 21 of those are ruff, and 14 of the ruff cards have not moved in two sprints
- Against the Q3 target of 60 new automated tests, nothing queued produces coverage. Weekly authoring has zero live candidates. Skills rollout and cloud agent readiness are empty.

## Blocking

- Retry removal has no child tasks yet. Every removal card depends on the Mode chart, then `[P4.1]`. If the chart slips, Dimpal's retry half of the sprint is empty.
- 21 ruff cards carrying over is larger than one sprint of the capacity Oscar actually has once resource-id and XPath work is counted.
