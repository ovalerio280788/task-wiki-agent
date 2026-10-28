# Wiki log

> Append-only chronological record of captured learnings. Newest entries at the top.
> Format per entry: `## [YYYY-MM-DD] <action> | <page>` where action is one of: create, update, archive, lint.
> When adding an entry, insert it directly below this header, above the previous newest entry.
> Rotate when this file exceeds ~500 entries or at year end: rename to `log-YYYY.md` and start fresh.

## [2026-10-28] update | workflows/mobile-browserstack-qa-docker.md
- Android BrowserStack launches add `@set_capability::interactiveDebugging=true` on each scenario that opens a device. Do not add it for iOS.

## [2026-10-28] update | workflows/mobile-browserstack-qa-docker.md
- Pass `-e BACKEND_INSTANCE=online_qa_server`. Compose interpolates that key from `e2e/mobile/.env`, so `online_demo_server` still calls `demo-1234.instawork.com` when only `BACKEND_URL` is overridden. `browserstack.timezone` is always sent and a plan without that capability never opens a session.

## [2026-10-28] update | workflows/mobile-browserstack-qa-docker.md
- Pass `++processes` and `++parallel-element` as separate `--behave-args` tokens. One quoted string is a single argument and behave exits unrecognized before upload.

## [2026-10-28] update | workflows/mobile-browserstack-qa-docker.md
- Run preflight on every BrowserStack launch, including a repeat. Run the tests with `docker compose -p mobile`. Pass `-e BUILD_ID` on that compose run so preflight's dated `.env` value is not the session build.

## [2026-10-28] update | workflows/mobile-browserstack-qa-docker.md
- Fast start: read app ids and devices from `.env`, skip upload and preflight, one build id per feature per OS, at most 10 parallel scenarios. Preflight rewrites `BUILD_ID` to today's date, so a repeat run must pass `-e BUILD_ID` instead.

## [2026-10-28] update | workflows/mobile-browserstack-qa-docker.md
- After a `--rm` BrowserStack run, score the JUnit file under `e2e/mobile/<app>/out/reports/`. The mobile-repo reports path is not this tree.

## [2026-10-28] update | repos/mobile-step-definition-conventions.md
- A missing step is Behave Broken with an unimplemented-step snippet, before any assertion. Grep feature files for the phrase; a Background migration can leave the same lines copied inside one scenario.

## [2026-10-27] update | workflows/mobile-browserstack-qa-docker.md
- Scenario parallelism needs `++processes` because `+` is rewritten to `-` once. Select specific scenarios with an `@` list of `file.feature:LINE` paths. `@broken` and line-skipped scenarios do not open a BrowserStack session.

## [2026-10-27] update | tooling/cursor-canvas-authoring.md
- Every canvas table starts with a `#` column. Row numbers stay tied to the full list when a filter hides rows.

## [2026-10-27] update | preferences/global.md
- Canvas tables always lead with a `#` column of 1-based row numbers.

## [2026-10-27] update | repos/mobile-e2e-endpoint-coverage.md
- No command rewrites a coverage mapping after a rename. Reconcile only deletes files whose names match no current scenario. A title-only change updates the label and renames the file; re-collect when the requests changed.

## [2026-10-27] create | repos/mobile-e2e-endpoint-coverage.md
- A renamed Gherkin scenario leaves committed endpoint-coverage YAML stale. Update the label and rename the file; the same validator fails both coverage CI jobs.

## [2026-10-28] update | tooling/instawork-automation-api.md
- Treat `CurrentTest.api_v2_data` as a client-side ledger: creating a business with a company name does not add that company to the ledger for later setup steps.

## [2026-10-27] update | repos/e2e-constance-parallel-race.md
- A shared catalog delete races the same way as a global config write, including when a missing row counts as success. Accept the latest live row per user. Latest is the highest live id, not the newest created row, and a not-live row does not count.

## [2026-10-27] update | workflows/mobile-browserstack-qa-docker.md
- BrowserStack API scoring should use the e2e `.env` pair. The shell profile username may not be exported, and its access key can 401 while `.env` works. macOS `urllib` needs certifi for that host.

## [2026-10-27] update | workflows/mobile-browserstack-qa-docker.md
- Scenario-level parallelism is already in behave-parallel. Instatest appends `--parallel-element feature` after user behave args, so `--threads` cannot be overridden. Omit `--threads` and pass `+processes` plus `+parallel-element scenario`, with `MULTI_THREAD=YES`, only when the BrowserStack app id is already set. `@serial` keeps a feature on one worker. Scenario mode reruns `before_feature` per scenario.

## [2026-10-27] update | tooling/instawork-automation-api.md
- The offset belongs on the shared shift format. Naive datetimes still omit it, so those callers stay on the Pacific default.

## [2026-10-27] update | tooling/instawork-automation-api.md
- When a shared shift step must keep naive timestamps, put the place in Pacific instead of adding an offset to that step.

## [2026-10-27] update | tooling/instawork-automation-api.md
- Shift timestamps with no offset are parsed as Pacific on an unauthenticated automation call. A non-Pacific place needs the offset on the string, or the stored window slides and clock-out records no payable time.

## [2026-10-27] update | tooling/hyperview-stale-press-handler.md
- A recording alone proves the press never ran: with `show-during-load`/`hide-during-load` on the behavior, count distinct renderings of the button region across the whole wait. One rendering means the behavior never fired, separating a lost press from a failed request with no network log.
- Rule out a disabled touchable first; a button exposing only `label` and `label_loading` cannot be inert for that reason.
- An inline-edit save on the same screen ("Saving…" then new derived values) is a document replacement, so the next press on any reused touchable is exposed.

## [2026-10-27] update | tooling/ios-visible-false-under-transparent-overlay.md
- Corrected the previous entry: the Android inert-press failure on the same screen is a *different* defect, not the sheet swallowing the tap. Decide between them from before/after values; a single end-of-run screenshot has no baseline, so "the value never changed" is not readable from it.

## [2026-10-27] update | tooling/ios-visible-false-under-transparent-overlay.md
- An unclosed selection bottom sheet emits both platform signatures at once: iOS asserts the primary button "not visible", Android clicks it successfully and the screen never advances. Check whether the preceding selection step closed the sheet before treating them as two bugs.
- A generic fixed-fraction "refresh" swipe is not a sheet-dismiss fallback; it usually starts on the backdrop above the sheet.

## [2026-10-27] update | tooling/allure-testops-screenshots.md
- Broadened from screenshots to TestOps failure artifacts: find the result via `/api/testresult?launchId=`, read the ordered step tree from `/execution`, and prefer the `step_retry_summary` and `API Call Timeline` attachments over the screenshot alone.
- `rql`/`name` params are ignored on `/api/testresult` and `launchId` is required, so a filtered-looking request silently returns the whole page.
- The token sits in `~/.zprofile` as `TESTOPOS_TOKEN`; a missing `testops_*` tool means a missing token, not a missing MCP server.
- Compare a repeating scenario's non-pass rate against a sibling that runs the block once to tell compounding per-block risk from a regression.

## [2026-10-27] update | tooling/redshift-allure-ops.md
- `scenarios.id` is the TestOps test result id, so the warehouse covers history and the API covers one run's steps.
- The step tables stay unusable even when scoped to the run's own time window; `name LIKE '%<n>]%'` is required because `test_case_id` collides with bracketed titles, and runs duplicate rows so count distinct `launch_id`.

## [2026-10-26] update | workflows/instawork-local-hiring-opensearch-setup.md
- Local Recruitics readiness is five steps: OpenSearch, Kafka topics plus consumer restart, the feed URL and both sync flags, then two posts to the readiness endpoint.

## [2026-10-26] update | workflows/instawork-local-hiring-opensearch-setup.md
- Local Recruitics ingest uses the QA feed URL in `RECRUITICS_JOB_SYNC_URL`, plus `ENABLE_HIRING_JOBS_SYNC` and `ENABLE_RECRUITICS_JOBS_SYNC`, before the sync script.
- Local Recruitics ingest is the Kafka consumer, and compose does not create `instawork.shared_message_bus_tasks`. Run `check_and_create_kafka_topics` and restart the consumer before the sync.

## [2026-10-27] update | repos/e2e-constance-parallel-race.md
- Gig details shows the company name, so a new place can hang off the existing company. The business needs its own name. The distance check uses that place's coordinates, so the GPS tag has to match them.

## [2026-10-26] update | repos/e2e-constance-parallel-race.md
- When business create ignores subsidiary, pick a place in a W-2-only state that is already on the wanted side of the other list. The state forces W-2, so the subsidiary write is unnecessary.

## [2026-10-26] update | repos/e2e-constance-parallel-race.md
- Businesses POST documents subsidiary as writable, but the custom create drops it and the factory stamps the 1099 subsidiary. Deleting the row and posting again does not change that.

## [2026-10-26] update | repos/e2e-constance-parallel-race.md
- A W-2 shift outside the W-2-only state list needs the subsidiary on create. The businesses detail route is retrieve and delete only, so get-or-create by name cannot update an existing row. Use a new name, or delete and create again.

## [2026-10-26] update | repos/e2e-constance-parallel-race.md
- A W-2 shift outside the W-2-only state list needs the business subsidiary set to a W-2 subsidiary. Get-or-create by name keeps the old subsidiary, so patch the existing row.

## [2026-10-26] create | tooling/hyperview-stale-press-handler.md
- First press after a document update fires the previous element's href because the press handlers are built from a behavior cache that only refreshes in `componentDidUpdate`, one commit late. Label is already correct, so the screen looks right.
- Recognition pair: a long wait never helps, but navigating away and back does. That rules out load races and points at a committed React closure.
- Instance reuse (positional keys), no intervening re-render, and the concurrent root all have to hold; the middle one is the intermittency lever.

## [2026-10-26] update | repos/e2e-constance-parallel-race.md
- A list-valued key may be matched against the shift's business place, not the worker profile. Do not patch the shared place. Append a second place whose state is already on the wanted side, with its own company and business, before the step that reads `businesses[-1]`. Override `google_place_id` on create or the SF default comes back. Point `@gps_` at that place: "close to location" is a geofence around the gig.

## [2026-10-26] update | repos/e2e-constance-parallel-race.md
- Do not change the reader to absorb a JSON fraction. Leave the fraction out of the override metadata so the existing decimal config value is used. Integers in that metadata are exact.

## [2026-10-26] update | tooling/browserstack-failure-triage.md
- The `/networklogs` HAR includes response bodies, so a 200 rendering the wrong copy is provable by grepping the body for the asserted string.

## [2026-10-26] update | repos/e2e-constance-parallel-race.md
- The race also runs across launches: each platform is its own CI pipeline hitting the same shared backend, so collocating scenarios in one feature file does not fix it.
- Detect by self-joining Allure scenario results on overlapping windows across different launch ids and comparing non-passed rates; confirm with symmetric HAR bodies.
- Prefer per-entity test data over any global write: a per-worker override record whose metadata carries the same keys, or an entity attribute that already sits on the wanted side of a list-valued key.
- A displayed value may be recomputed at render time from live config, so the exclusive window must cover the assertions, not just setup. `::after` restores a hardcoded literal, not the captured prior value.

## [2026-10-23] create | tooling/captain-tooling-shared-business.md
- Captain scenarios that get-or-create one business adopt each other's overlapping shifts. A roster larger than this scenario's workers, then Forbidden "The shift has been cancelled.", is the other scenario's teardown.

## [2026-10-23] update | workflows/mobile-browserstack-qa-docker.md
- Parallel containers that share the bind-mounted tree can corrupt `__pycache__` and exit before a session starts. Set `PYTHONDONTWRITEBYTECODE=1` and clear leftover cache directories first.

## [2026-10-23] update | tooling/ios-visible-false-under-transparent-overlay.md
- Android header controls stay displayed under a root toast top sheet. A successful click dismisses the toast and never reaches the control. The sheet stays up for 3 seconds.

## [2026-10-23] update | workflows/refresh-browserstack-env-app-ids.md
- `instawork` and `instawork_2` each have their own `e2e/mobile/.env`. Write the same four `BS_APP_ID_*` lines to both.

## [2026-10-23] update | workflows/mobile-browserstack-qa-docker.md
- Index line and title now say e2e `.env`, Docker Compose, and parallel runs, so a preflight scan matches that request.

## [2026-10-23] create | workflows/refresh-browserstack-env-app-ids.md
- The four `BS_APP_ID_*` values come from the newest `host_*` App Automate session, one subagent per app. `recent_apps` is too short to be the source.

## [2026-10-23] update | workflows/mobile-browserstack-qa-docker.md
- Point the `BS_APP_ID_*` note at the env refresh workflow.

## [2026-10-23] update | tooling/browserstack-app-automate-api-cloud-agents.md
- The latest CI binary for a suite is `app_details.app_url` on the first `host_*` build, not `recent_apps`.

## [2026-10-23] update | tooling/redshift-allure-ops.md
- Scenario failure text comes from `launch_scenario_results`. `scenario_step_results.scenario_id` does not match `scenarios.id`, and `migration_date` is the load time.

## [2026-10-23] update | tooling/asana-api-cloud-agents.md
- Changing a task's parent is `POST /tasks/<gid>/setParent`. A `parent` field on `PUT /tasks` is rejected.

## [2026-10-22] create | repos/e2e-constance-parallel-race.md
- Parallel copies of a scenario that sets the same Constance key still race: the first `::after` restores the default while the others are still reading it.

## [2026-10-22] update | workflows/mobile-ios-localhost-automation-build.md
- Hermes prebuilt replace fails the same way as RN core when the `-r` tarball is missing. The script deletes the extracted engine when the last-build marker is a different configuration.

## [2026-10-22] update | repos/hyperview-local-mobile-sync.md
- Hyperview `master` is now a Yarn 4 workspace. Stale `node_modules` breaks jest (preset not found). Run `yarn install --immutable` at the root, then run jest from `hyperview/`.

## [2026-10-22] update | repos/mobile-step-definition-conventions.md
- Do not read the window or screen size to decide whether an element is tappable. Swipe with the existing helper and confirm a content id.

## [2026-10-22] update | repos/mobile-automation-waits.md
- A horizontal tab strip's last item can report displayed while clipped. The click returns, the label assertion passes, and the panel-only copy never appears. The shared text step already retries, so this is not a load wait.
- Hyperview option selection is an XML `selected` attribute. It is not copied to the accessibility tree, and the underline is a style modifier. Assert the selected panel, not the tab node.

## [2026-10-22] update | workflows/mobile-browserstack-qa-docker.md
- An iOS Files result row is a cell whose name starts with the file stem. Match it with a predicate. `IdSelector` `StartsWith` is rejected by `AppleSelector`.

## [2026-10-21] update | workflows/mobile-browserstack-qa-docker.md
- The iOS Files picker on a shared BrowserStack device opens on the last-used tab. The sheet title during the open animation is that default. A resume step that only leaves Recents when the empty-state sentence is showing passes only when the device restores Browse.

## [2026-10-20] create | infra/celery-worker-capacity.md
- Deployed Celery parallelism is the worker command and ECS service count in `Instawork/infrastructure`. The app repo only names queues.

## [2026-10-20] update | workflows/mobile-browserstack-qa-docker.md
- Long-lived shared automation companies carry lifecycle timestamps stamped once by periodic tasks from whatever shifts existed then. Scenarios asserting time-windowed partner behaviour must pin that timestamp; otherwise they decay into flakes weeks later.

## [2026-10-20] update | workflows/instawork-local-hiring-opensearch-setup.md
- The recruitics ready check reads reference numbers from the environment's `RECRUITICS_JOB_SYNC_URL` and caches them as `e2e_recruitics_feed_references`. A fixed fixture id list only matches one feed.

## [2026-10-20] update | workflows/instawork-local-hiring-opensearch-setup.md
- Jobs features prepare the recruitics feed with `I ensure recruitics jobs are searchable`. That step creates the hiring alias only when it is missing and serializes the sync on the cache key `e2e_recruitics_jobs_searchable`.

## [2026-10-20] update | workflows/instawork-local-hiring-opensearch-setup.md
- Parallel jobs features on a shared env delete each other's OpenSearch documents: the sync finalizer deletes every OPEN job not stamped with its own `sync_process_id`. Recovery is partial because only easy-apply jobs get the artform re-index.
- `CACHE_SYNDICATED_JOB_IDS_FROM_OPENSEARCH` caches a partial list per pro for 300s, which outlives every step retry.

## [2026-10-19] update | workflows/instawork-local-hiring-opensearch-setup.md
- A Jobs tab interview step looking for "Apply to interview" on an APPLIED / Continue Application screen opened the application intro. `DIRECT_TO_SHIFT_REDIRECT` defaults off and the suite never sets it.
- The same scenario's booked headline is skipped when the shift starts 2 days out. Confirmation opens 24 hours before start, and the default flag keeps the locked confirm task on screen.

## [2026-10-16] update | workflows/mobile-browserstack-qa-docker.md
- Interview-shift applicant scenarios stay off demo because setup waits for an OpenShift row that demo does not write. The skip applies only when the launch tags include `~exclude_from_demo`.

## [2026-10-16] update | tooling/instawork-automation-api.md
- Local terms rows are missing when QA has them. List with the numeric type, read the body from the legal page, and create with the choice label.

## [2026-10-16] create | repos/instawork-docker-uv-data-stale.md
- A missing import for a package already in requirements.txt is the `uv_data` volume, not the image. `just pip-sync` writes that volume.

## [2026-10-16] update | workflows/mobile-browserstack-qa-docker.md
- Parallel applicant runs on one shared backend both create the same fixed company in `before_workers`. The loser logs a hook error and can still report the scenario passed.

## [2026-10-16] update | tooling/django-get-or-create-conflict-test.md
- An unsaved one-to-one instance built with its parent set is cached on that parent. `hasattr` then skips the database lookup. Build the instance inside the mocked call.

## [2026-10-16] update | tooling/browserstack-failure-triage.md
- A refused device-port connection while starting the session means the scenario never ran. Later errors on a missing driver are that same failure.

## [2026-10-16] create | tooling/django-get-or-create-conflict-test.md
- A row inserted inside `create()` during `get_or_create` is rolled back with the failed insert. Insert it after the lookup misses and before the savepoint so the recovery lookup can see it.

## [2026-10-15] update | repos/instawork-worker-app-shell-sendbird-500.md
- Another token request does not create a missing Sendbird user. A refresh loop stays empty and adds more token posts.

## [2026-10-15] update | repos/instawork-worker-app-shell-sendbird-500.md
- HTTP 400 with Sendbird code 400201 means the token call ran and that user id is missing from the Sendbird app. The refresh does not create the user.

## [2026-10-15] update | repos/instawork-worker-app-shell-sendbird-500.md
- The empty-token path does not log or raise, so Sentry will not show it. A profile-update connection error is a different call.

## [2026-10-15] update | repos/instawork-worker-app-shell-sendbird-500.md
- A fast empty `session-token-refresh` after a burst of new workers is Sendbird answering with no token. The retries are more token posts and keep the shared app saturated.

## [2026-10-15] update | workflows/mobile-browserstack-qa-docker.md
- Many BrowserStack Local binaries started in the same second can all fail DNS for the binary host. The scenario still runs with the tunnel down, so stagger the containers and treat `Browserstack local running: False` as a failed run.

## [2026-10-15] update | tooling/instawork-cursor-backend-debugging.md
- A Django breakpoint that logs `unable to find translation` against `e2e/mobile` paths is bound to the mobile automation session. A demo backend host means the local Django process never runs that line.

## [2026-10-15] update | tooling/instawork-cursor-backend-debugging.md
- The gitignored `.vscode/launch.json` already attaches mobile automation on ports 15678–15685. Add the Django attach configs there.

## [2026-10-15] create | tooling/instawork-cursor-backend-debugging.md
- Attach Cursor to the Dockerized Django server on port 5678 or a waiting pytest process on 5700, with local root mapped to container `/app`.

## [2026-10-15] create | tooling/hyperview-navigator-event-behaviors.md
- A navigator only runs load and on-event behaviors. A behavior cannot have an id, so a follow-up fragment is appended onto the navigator.

## [2026-10-16] update | tooling/redshift-copy-load-errors.md
- When load errors are not visible, check the S3 file locally against the schema. Varchar limits are bytes. Compare with the last good file.

## [2026-10-15] create | tooling/redshift-copy-load-errors.md
- A Redshift COPY failure in an Airflow log is only the generic XX000 message. The bad row is in stl_load_errors, and a non-superuser session sees only its own loads.

## [2026-10-15] update | workflows/mobile-browserstack-qa-docker.md
- The business suite has no demo backend entry. Use the QA profile and override the backend URL to the demo host, or every step stays untested.

## [2026-10-15] update | repos/instawork-local-sendbird-user-id-collision.md
- A missing empty-inbox string needs the screenshot. Gray placeholder rows mean the list is still loading. Named conversations mean the Sendbird user already had channels.

## [2026-10-15] update | repos/airflow-allure-launch-without-scenarios.md
- A launch missing scenario rows in the warehouse can still have leaf results in TestOps. Check `totalElements` before treating the source as gone.

## [2026-10-15] update | repos/airflow-allure-launch-without-scenarios.md
- Skipped launches stay out of the table and keep the four-launch window. Deleting an older hole does not guarantee the next run reaches it.

## [2026-10-15] update | repos/airflow-allure-launch-without-scenarios.md
- Re-ingest copies statistics, environments, jobs, and tags. Delete those with the launch row, or the next run duplicates them.

## [2026-10-15] create | repos/instawork-local-sendbird-user-id-collision.md
- A new local worker can show old Messages because Sendbird identity is the numeric user id on the shared app. A fresh session token does not create an empty inbox.

## [2026-10-15] update | workflows/mobile-browserstack-qa-docker.md
- A real BrowserStack device white-screens when the app still points at localhost. Recreate the backend so `IW_DOMAIN` is `http://bs-local.com:8080`, and use that same host for the launch override.

## [2026-10-15] create | repos/airflow-allure-launch-without-scenarios.md
- A launch id in `allure_ops.launches` is treated as done before scenario rows exist. An empty TestOps leaf or a failed scenario fetch stays missing on every later run.

## [2026-10-15] create | tooling/redshift-allure-ops.md
- Allure warehouse tables are the `allure_ops` schema on `instawork-dw` / `instawork`. The Finch prod Redshift plugin can read them. There is no allureops database.

## [2026-10-14] update | repos/instawork-worker-app-shell-sendbird-500.md
- A shell 500 on `GET /biz_app/app` after partner login is often the duplicate Sendbird session-token insert, not the connection error. Match the request id.

## [2026-10-14] update | tooling/asana-api-cloud-agents.md
- A story written with unsupported rich-text tags is stored as escaped markup and renders as raw tags. Use the same XML rules as a task description, and check the saved html for real tags.

## [2026-10-14] create | repos/instawork-worker-app-shell-sendbird-500.md
- A generic Hyperview 500 on the authenticated worker app shell maps to an uncaught Sendbird session-token fetch. Match the response request id in Sentry before treating it as test data.

## [2026-10-14] update | tooling/browserstack-app-automate-api-cloud-agents.md
- App Automate network logs are HAR JSON; failed app responses include response bodies and tracing headers that can connect an error screen to an endpoint. Exclude driver and OS/CDN noise, and preserve an explicit no-failed-response result when the HAR is inconclusive.

## [2026-10-14] update | tooling/android-rn-native-crash-repro.md
- Galaxy A16 replays that end on this fatal time are mostly shifts home or gig details while scrolling. One is the screening call. No shared control.

## [2026-10-13] update | tooling/android-rn-native-crash-repro.md
- Worker-app replay API event times are UTC. The project UI timezone is America/Los_Angeles, so do not compare that clock to a Sentry Z timestamp.

## [2026-10-13] update | tooling/browserstack-app-automate-api-cloud-agents.md
- App Live's device picker is not the Appium catalog. Galaxy A16 5G and Galaxy A17 on Android 16 show in App Live and are absent from GET /app-automate/devices.json. Those sessions stay manual.

## [2026-10-13] update | tooling/browserstack-app-automate-api-cloud-agents.md
- App Live recent apps are the named beta and production APKs. App Automate apps are CI builds with a version suffix. Match the store build via the -ci- sha on versionCode. An old bs:// id is still on the session that used it. An App Live bs:// id does not run in App Automate.

## [2026-10-13] update | tooling/android-rn-native-crash-repro.md
- A system-only MotionEvent getRawAxisValue abort is not the Fabric teardown race. No shared screen and store-only means no tap sequence. getRawX(pointerCount) forces the OS fatal only.

## [2026-10-13] update | tooling/sentry-mobile-rn-upgrade-filter.md
- Events-list Title is the top symbol. Aggregate stack.function before splitting an abort issue from an IdGenerator SIGSEGV.

## [2026-10-13] update | tooling/instawork-automation-api.md
- Public swagger schema generation builds views with request=None. A serializer switch that reads request.method 500s the whole document.

## [2026-10-12] update | tooling/instawork-automation-api.md
- Focused demo cleanup: delete companies before workers. Owner-less companies fail DELETE with
  UserProfile missing; peel FK children via run-celery-task SQL when stuck.

## [2026-10-12] update | tooling/allure-testops-screenshots.md
- The ios tag on a mobile test result is the scenario tag. The device platform is the launch name prefix.

## [2026-10-12] update | workflows/mobile-browserstack-qa-docker.md
- The BrowserStack build name is the application name plus BUILD_ID, so iOS and Android stay separate unless a set_capability build tag overrides it.

## [2026-10-12] update | tooling/allure-testops-screenshots.md
- Platform for a TestOps result is the ios or android tag. If the tag is missing, use the launch name prefix.

## [2026-10-12] update | tooling/allure-testops-screenshots.md
- The BrowserStack recording link is the first dashboard session href in the HTML attachment named BrowserStack session links.

## [2026-10-12] create | tooling/allure-testops-screenshots.md
- TestOps screenshots are attachments on the tree's test result id. An image with content-length 0 is a missing file.

## [2026-10-10] update | workflows/mobile-browserstack-qa-docker.md
- Applicant behave exclude drops a feature filename that contains `deep_`. The run exits 0 with 0 features queued.

## [2026-10-09] update | workflows/mobile-browserstack-qa-docker.md
- A fresh demo hostname can resolve on public DNS before Docker does. Point the automation service at public DNS and wait for the health check SHA before launching.

## [2026-10-09] update | workflows/mobile-browserstack-qa-docker.md
- BrowserStack Local runs in the automation container and only for a :8080 backend URL. The device uses bs-local.com:8080, which is the container port. Confirm the running backend checkout is the one under test.

## [2026-10-09] update | tooling/appium-mcp-browserstack-device-session.md
- The "Open this page in Instawork?" sheet belongs to the Safari handoff. `mobile: deepLink` does not show it, so the suite helper does not tap Open.

## [2026-10-09] update | repos/mobile-automation-waits.md
- `mobile: deepLink` returns when the URL is open. A sleep after it does not wait for screen content. `screen should be loaded` does not either.

## [2026-10-09] update | tooling/appium-mcp-browserstack-device-session.md
- `mobile: deepLink` with the session bundle id opens an `instawork://` login link on both a local simulator and a BrowserStack real device.

## [2026-10-09] update | workflows/mobile-browserstack-qa-docker.md
- `--threads` parallelizes by feature. Repeating one scenario needs one compose run per copy. A `--rm` run loses logs if the client exits before the container does.

## [2026-10-09] update | tooling/appium-mcp-browserstack-device-session.md
- XCUITest 5.14 `driver.get` is `simctl openurl` on a simulator and WDA `/url` without `bundleId` on a real device. `mobile: deepLink` is the call that names the bundle.

## [2026-10-09] update | tooling/appium-mcp-browserstack-device-session.md
- Suite BrowserStack sessions default to Appium 2.4.1 (XCUITest 5.14.0), which implements `mobile: deepLink`. The MCP lifecycle unknown-command error is that tool, not the suite execute script. The same call works on a local simulator.

## [2026-10-09] update | tooling/appium-mcp-browserstack-device-session.md
- iOS can open a deep link without Safari via `mobile: deepLink` on driver 4.17+ and iOS 17+. `driver.get` is not that API. An unknown-command response means the hub still needs Safari.

## [2026-10-09] update | tooling/circleci-api-job-logs.md
- An approve-hold step that exits after only "Checking type of …" lost the workflow jobs call (`curl -f`). The wrong-type branch prints a different line and exits 0. A jobs 404 on a workflow id that otherwise loads means rerun the workflow, not the job.

## [2026-10-08] update | tooling/ios-safari-deeplink-address-bar.md
- A cold start after Safari `Go` either never requests `deep_links`, or receives the login document and never runs its load behaviors. Both stay on the intro screen.

## [2026-10-08] update | tooling/ios-safari-deeplink-address-bar.md
- Intro screen plus a Safari back button means the app opened and the token query did not arrive. The step retry re-reads page source and does not reopen Safari.

## [2026-10-08] update | workflows/mobile-browserstack-qa-docker.md
- A new compose project name builds a new `<project>-automation` image. Parallel BrowserStack runs should stay on `-p mobile` and use a unique `--name` per container.

## [2026-10-08] update | tooling/ios-safari-deeplink-address-bar.md
- `setValue` `socket hang up` on the Safari URL field is the XCUITest driver losing WebDriverAgent, not the app. The locators have already succeeded.

## [2026-10-08] update | tooling/ios-safari-deeplink-address-bar.md
- The keyboard-up Safari screen can be a retry. The first failure can be `setValue` `socket hang up` after a successful `TabBarItemTitle` click and `URL` find. The click is what raises the keyboard.

## [2026-10-08] update | tooling/ios-safari-deeplink-address-bar.md
- Safari can open with the keyboard down, then focus the address field on its own within about two seconds. A `TabBarItemTitle` wait that starts after launch misses it.

## [2026-10-08] update | tooling/ios-safari-deeplink-address-bar.md
- Keyboard down keeps `name` `TabBarItemTitle`. Keyboard up renames that field to `URL`. `Go` stays disabled until the field has text.

## [2026-10-08] update | tooling/ios-safari-deeplink-address-bar.md
- Expanded Safari address field keeps id `URL`. "Search or enter website name" is the empty value. Relaunch leaves the field expanded, so each retry misses `TabBarItemTitle`.

## [2026-10-08] create | tooling/ios-safari-deeplink-address-bar.md
- A deep-link timeout on `TabBarItemTitle` is Safari's collapsed address button. An already-expanded address field means the URL was never typed.

## [2026-10-08] update | workflows/web-ui-smoke-failure-triage.md
- An empty tests API list means Playwright never ran. Read the failed v1.1 step. The alert SHA is the pipeline commit; a repeated pre-test failure belongs to the earlier commit that introduced it.

## [2026-10-07] update | repos/mobile-step-retry-retrying.md
- Prefer an in-step wait or in-step poll over `@retry`. Keep the decorator only when the whole step is safe to repeat.

## [2026-10-07] update | repos/mobile-step-retry-retrying.md
- `retrying` 1.4.2 has no settable defaults. Each `@retry(...)` stands alone. A bare `@retry` retries forever with no wait. `Exception` is caught; `BaseException` is not.

## [2026-10-07] create | repos/mobile-step-retry-retrying.md
- `retrying` re-runs the whole step. `@step` must sit above `@retry`. `wait_fixed` is milliseconds and does nothing if omitted. Scenario retry is already `TEST_GLOBAL_RETRIES`.

## [2026-10-07] create | tooling/ios-visible-false-under-transparent-overlay.md
- iOS `visible` is a hit test. A transparent absolute sibling covering a field marks that field not visible; Android `displayed` stays true.

## [2026-10-07] create | repos/hyperview-optional-field-label.md
- A Hyperview "(optional)" heading suffix is appended by the label template for non-required fields. A `label=` edit does not remove it.

## [2026-10-06] update | workflows/mobile-test-impact-from-pr-or-branch.md
- A store version is not a `/run_test` parameter. Use the release tag's per-platform CircleCI automation job number; `specific-build` cannot be `platform=both`.

## [2026-10-06] create | tooling/git-blame-phantom-file-add.md
- Blame stops when a commit records a file as added but the parent diff is empty. Blame the parent; a later same-line rewrite is not when a token was introduced.

## [2026-10-06] update | workflows/mobile-ios-localhost-automation-build.md
- Device beta IPA: `yarn build:ios:beta` in the app dir, ad-hoc match signing needs GitHub SSH + API key json; output `bin/<app>-beta.ipa`.

## [2026-10-06] update | tooling/mobile-datadog-sdk-metric-check.md
- Mobile SDK sessions are in RUM Explorer. APM Traces with `env:beta` returns no services.

## [2026-10-06] update | tooling/mobile-datadog-sdk-metric-check.md
- Beta builds show `env:beta`. iOS is the unsuffixed bundle id; Android is the `.beta` service. Filter by store version.

## [2026-10-06] update | tooling/mobile-datadog-sdk-metric-check.md
- Mobile Datadog services are split by bundle id and flavor. Staging2 appears as backend hostnames, not as a mobile env or service name.

## [2026-10-06] update | tooling/mobile-datadog-sdk-metric-check.md
- Staging and production share one RUM application. Datadog `env` is the build flavor. Each backend's Constance decides whether that environment returns a config.

## [2026-10-06] create | tooling/mobile-datadog-sdk-metric-check.md
- Mobile Datadog "metrics reported" checks need a non-debug build, non-zero sampling, and an uncancelled startup report. Custom metrics are `tti-` / `metric-` logs.

## [2026-10-06] update | tooling/instawork-automation-api.md
- Pro gig details deep link is `instawork://open-gigs/<shift group hash>`. The shift-group `shareable_link` field is not that screen.

## [2026-10-06] update | tooling/instawork-automation-api.md
- Booking via shift-groups `worker_id` skips the Worker Assigned log unless `resolved` is true. My Shifts 500s on a lost-Top-Pro worker until `POST /worker-logs` event 9. The shift field accepts the hash.

## [2026-10-06] update | tooling/instawork-automation-api.md
- Staging2 automation API is the same v2 surface on admin.staging2. Swagger is `GET .../swagger`. Worker detail takes the numeric profile id; company and business detail take the admin hash id.

## [2026-10-05] update | workflows/asana-bug-report.md
- Translation bugs: locate the msgid and template, grep the `es` catalog for how sibling msgids are rendered, propose the dominant convention; obsolete `#~` entries can still match a newer live backend.

## [2026-10-05] update | repos/mobile-automation-waits.md
- The navigation-tab displayed step polls `current_screen.<tab>_tab`. Assign open shifts or profile first; UI login still leaves the verification screen.

## [2026-10-05] update | workflows/ruff-noqa-cleanup-pipeline.md
- F401 / unused-module cleanup in `lib/instatest` can delete a re-export that `e2e/mobile` steps still import. Grep the other tree before deleting; Behave fails at step load.

## [2026-10-05] update | workflows/browserstack-release-feature-testing.md
- Biz P0 plans clone `TestPlan Templates`. Drop Staging P0/P1. Feature run uses the version folder, not the template's Login placeholder.

## [2026-10-05] update | workflows/browserstack-release-feature-testing.md
- A P0 release plan keeps Prod P0, Sentry Review, Automation review, and the version feature-testing run. Drop the copied Staging run.

## [2026-10-05] update | tooling/browserstack-test-management-api.md
- Delete a test run with POST `.../test-runs/{id}/delete`. HTTP DELETE 404s.

## [2026-10-05] create | workflows/browserstack-release-feature-testing.md
- Release feature cases go under the project's Release Feature Testing folder, one case per note, links in preconditions. Create response often omits the TC id.

## [2026-10-05] update | tooling/browserstack-test-management-api.md
- Cross-link the release feature-testing workflow.

## [2026-10-02] create | repos/instawork-instatest-unit-tests.md
- `lib/instatest` pytest files run inside the `automation` compose service with `-w /workspace/lib/instatest`. No CI job runs them; failures may pre-date the branch.

## [2026-10-01] create | repos/hyperview-tab-replace-dismisses-welcome.md
- Tab REPLACE must navigate the root `tabs-route`. `setParams` hits the focused welcome sibling. My shifts is `money-route`.

## [2026-10-01] update | tooling/appium-mcp-browserstack-device-session.md
- Token login deep links are internal only: automation API worker tokens and the admin Mobile login link. Not a pro path.

## [2026-10-01] update | tooling/appium-mcp-browserstack-device-session.md
- Missing tab bar is the login deep link only. An already open session keeps the tab focused, so the same link does not rewrite the welcome modal.

## [2026-10-01] update | tooling/appium-mcp-browserstack-device-session.md
- Welcome GET records the feature view before paint. A fast deep link can skip Congrats and still leave My shifts with no tab bar.

## [2026-10-01] update | tooling/appium-mcp-browserstack-device-session.md
- Fresh-worker deep link: mark `worker_welcome_popup` viewed before login. Closing Congrats after My shifts replaces it does not restore the tab bar.

## [2026-10-01] update | workflows/mobile-android-resource-id-locator-fix.md
- `@elements(id=)` counts both the touchable and its label when they share an id. A text-only xpath count then doubles.

## [2026-09-30] update | workflows/ruff-noqa-cleanup-pipeline.md
- Reproduce `lint-e2e` with the `ruff==` pin in `e2e/mobile/requirements_qa.txt`. PATH ruff can be newer and flag rules that job does not run.

## [2026-09-30] update | repos/mobile-automation-waits.md
- No wait-until-not-stale exists; staleness is one-way. Re-locate via selector-based waits instead.

## [2026-09-30] update | workflows/mobile-test-impact-from-pr-or-branch.md
- Applicant `@smoke` is gigs, login, navigation, and profile. Hiring, MDS viewer, and most messages are outside it.

## [2026-09-29] update | tooling/browserstack-app-automate-api-cloud-agents.md
- `~/.zprofile` can hold two access keys. The `export` line can 401; the plain assignment is the one that authenticates. Confirm with `plan.json`.

## [2026-09-29] create | tooling/appium-webview-context-chromedriver.md
- A listed WEBVIEW context still fails `switch_to.context` when no Chromedriver matches that WebView's Chrome. Click the native node if the hierarchy already exposes it.

## [2026-09-29] update | workflows/mobile-android-login-local.md
- `NoneType` on `implicitly_wait` means the Appium session never started. Read the `WebDriverException` above it; usual cause is host `adb install` past `androidInstallTimeout`.

## [2026-09-29] update | workflows/mobile-ios-localhost-automation-build.md
- Business app variant of the xcodebuild recipe. Stale node_modules/Pods and missing RN core prebuilt tarballs break the build; script-phase failures hide from `error:` greps.

## [2026-09-29] update | workflows/mobile-android-resource-id-locator-fix.md
- `{id}/text-input` is the editable test id on iOS and Android. One `@element(id=)` covers both.

## [2026-09-29] update | workflows/mobile-android-resource-id-locator-fix.md
- Native Android select-dialog rows share `android:id/text1`. Scroll `select_dialog_listview` to the row text.

## [2026-09-29] update | workflows/web-ui-smoke-failure-triage.md
- Rerunning an old master deploy fails shallow checkout once master moves past the pipeline SHA.

## [2026-09-29] create | workflows/web-ui-smoke-failure-triage.md
- Master web-ui-smoke includes `@p1 @flaky`; skip-tags omit flaky. Prefer tests API + test-results.zip.

## [2026-09-29] update | workflows/mobile-android-resource-id-locator-fix.md
- Hyperview autocomplete `setText` misses the wrapper. The EditText resource-id is `{id}/text-input`.

## [2026-09-29] update | workflows/qa-platform-quarter-close-out.md
- A parent card's subtask badge is direct children. Leaf percent diverges when a direct child has children. Name the denominator.

## [2026-09-29] update | tooling/granola-mcp-bulk-export.md
- `list_meetings` rejects `time_range: custom`. Use `this_week`, `last_week`, or `last_30_days`, then `query_granola_meetings` or `get_meetings`.

## [2026-09-29] update | workflows/mobile-test-impact-from-pr-or-branch.md
- `@ios` selects a mobile scenario for Android and iOS. No `@ios` means Android only. There is no `@android` tag.

## [2026-09-28] update | repos/mobile-step-definition-conventions.md
- A deleted Behave step can leave the screen locator. Prove it with a name search plus `git log -S` on the public method, and check `getattr(screen, ...)` cannot build that name. Unused-step scans do not see it.

## [2026-09-28] update | tooling/hyperview-button-enabled-attribute.md
- One `button()` with `state=` is enough when a child behavior is already on that call.
  `is_enabled()` on the id node follows it on current Android and iOS. An empty press
  behavior only withholds the action from clients that never get `disabled`.

## [2026-09-28] update | tooling/hyperview-button-enabled-attribute.md
- `button()` always renders children. A `state=` ternary only drops the button's own
  action/href/verb/target on older clients. Child press behaviors stay live.

## [2026-09-27] update | workflows/qa-platform-sprint-kickoff.md
- Mid-sprint, middle of the sprint, and the mid-sprint Monday are week two. Do not ask the mode when the user already said that.

## [2026-09-25] update | tooling/hyperview-button-enabled-attribute.md
- A press behavior with no action defaults to `push`, but a missing href becomes `#`, which
  is rewritten to a navigate with an empty route id and then skipped. It does not throw.
  `action="close"` would dismiss the screen if that press runs.

## [2026-09-25] update | tooling/hyperview-button-enabled-attribute.md
- iOS `is_enabled()` is true for an enabled touchable MDS button. The disabled touchable
  still needs its own iOS assertion; an older page source showed `enabled="true"`.

## [2026-09-25] update | workflows/mobile-android-resource-id-locator-fix.md
- Nested-wrapper enabled xpath still is not a migration. `clickable` is only the signal when
  the button is not a touchable; native `enabled` is documented on the new tooling page.

## [2026-09-25] create | tooling/hyperview-button-enabled-attribute.md
- Native `enabled` requires a touchable (`href`, `action`, or child behavior) plus
  `disabled="true"` (Hyperview >= 0.110). Assert the id node. A behavior with no action
  defaults to `push`.

## [2026-09-25] update | workflows/xpath-to-id-app-target-resolution.md
- MDS `{id}-disabled` is only the hidden twin of an enabled button with a behavior. A
  `state=disabled` button keeps the base id; Android `enabled` stays true, `clickable` is the
  signal. A 32-char hex resource-id is `@auto_id()` (state is hashed, children are not).

## [2026-09-25] update | workflows/mobile-android-resource-id-locator-fix.md
- Structural "nested clickable wrapper" xpath for Hyperview button enabled state dies on RN view
  flattening; screenshot shows enabled, xpath says disabled. Not migration; `clickable` is the signal.

## [2026-09-25] update | workflows/mobile-android-resource-id-locator-fix.md
- Id-less text child under an id'd card: IdSelector parent + scoped TextSelector via
  find_child_element replaces the content-desc descendant xpath; comma-joined content-desc is
  RN's derived label, not the id.

## [2026-09-25] update | tooling/asana-api-cloud-agents.md
- Board custom fields rejected on subtask create; create, addProject, then PUT custom_fields.

## [2026-09-25] update | workflows/mobile-android-resource-id-locator-fix.md
- Migrated selector API lives only on the migration integration branch; a locator fix using it
  cannot target master (import-time TypeError across the whole suite). Base the PR accordingly.

## [2026-09-25] update | workflows/mobile-android-resource-id-locator-fix.md
- After a locator fix, a later wait for permission copy is not this migration if the dump is
  already the in-app camera capture control.

## [2026-09-24] update | preferences/global.md
- Programmatic mobile id lookups use `IdSelector`, not `Selector.by_id`. `@element(id=)` stays for
  static locators.

## [2026-09-24] update | repos/mobile-step-definition-conventions.md
- Anchored id regex example constructs `IdSelector` directly.

## [2026-09-24] update | workflows/mobile-android-resource-id-locator-fix.md
- Report step/verdict can be stale after a merged locator PR; classify from the live run, and treat
  a later failing step as its own classification.

## [2026-09-24] update | workflows/mobile-android-resource-id-locator-fix.md
- `-f` is a single path; a second `-f` overwrites the first.

## [2026-09-24] update | workflows/mobile-android-resource-id-locator-fix.md
- Old `Selector.by_id` was accessibility-id, so a previously passing `id=` constant is proof the
  token sits on content-desc; classify without a dump.

## [2026-09-24] update | workflows/mobile-android-resource-id-locator-fix.md
- Skill lives in the wiki workspace `.cursor/skills`, not in instawork.

## [2026-09-24] create | workflows/mobile-android-resource-id-locator-fix.md
- One-scenario local Android loop: vscode compose path, classify resource-id miss, fix like 47197, re-run once.

## [2026-09-23] update | tooling/ios-native-tabbar-liquid-glass-locators.md
- Name union is existing `AppleSelector`/`RegexSelector` Matches; `AND type` is the only gap.

## [2026-09-23] create | tooling/ios-native-tabbar-liquid-glass-locators.md
- Native vs Hyperview tab trees: Button + title-case `name` vs Other + id `name`.
- Locator must MATCH both names and allow Button or Other; TabBar ancestor is native-only.

## [2026-09-23] update | repos/mobile-automation-waits.md
- `wait_for_condition(predicate, *, timeout_s=5, poll_freq=0.25)`: zero-arg callable only;
  close over extra data. Timeout/poll are keyword-only. No `*args`/`**kwargs` forwarding.

## [2026-09-23] update | repos/mobile-automation-waits.md
- `wait_for_condition` now defaults `timeout_s=5` and `poll_freq=0.25`; predicate is first
  so callers can omit both. `poll_freq > 1` is still milliseconds.

## [2026-09-19] update | workflows/qa-platform-quarter-close-out.md
- Full runbook for the end-of-quarter canvas: confirm umbrellas, leaf-walk REST,
  sanity-check dates and capacity share, agree verdicts, then canvas. No markdown
  packet. Skip blockers and new-ideas unless they are real. Why column: one clause,
  no estimates. Filename `qa-mission-<n>-q<n>-close-out.canvas.tsx`.

## [2026-09-19] update | tooling/cursor-canvas-authoring.md
- Quarter close-out canvas filename and pointer to the close-out workflow.

## [2026-09-19] update | workflows/qa-platform-sprint-kickoff.md
- Related: quarter close-out is the skip-level sibling of kickoff.

## [2026-09-19] update | workflows/qa-platform-quarter-close-out.md
- Skip new-quarter ideas when the plan is carry: they restate milestones or
  contradict the verdict table. Only keep ideas that are actually new work.

## [2026-09-19] update | workflows/qa-platform-quarter-close-out.md
- Skip the blockers section when the miss is sequencing and capacity inside the
  same mission. Progress bars plus shipped / not shipped already answer what's left.

## [2026-09-19] create | workflows/qa-platform-quarter-close-out.md
- Quarterly mission status: leaf-task % per milestone (recursive subtask walk,
  `num_subtasks == 0`), but lead with the committed-outcome %. Activity % and
  outcome % disagree and the flattering one is usually the artefact of a milestone
  nobody created tasks for. Carry vs pause turns on whether a dormant milestone was
  starved or had its precondition never met.

## [2026-09-19] create | tooling/python-local-https-certifi.md
- macOS framework Python has no CA bundle, so stdlib `urllib` raises
  CERTIFICATE_VERIFY_FAILED while `curl` works. Pass
  `ssl.create_default_context(cafile=certifi.where())`; never disable verification.

## [2026-09-19] update | tooling/cursor-canvas-authoring.md
- Percent-complete bar = `UsageBar` with one segment and `total` as denominator.
- `key` is rejected on local components too, not just SDK ones; wrap the mapped
  element in a native `<div key>`. `React` namespace is out of scope for prop
  annotations as well as return types.

## [2026-09-19] update | workflows/qa-platform-sprint-kickoff.md
- Kickoff canvas "Where we are" is speakable bullets (last sprint / this sprint),
  not two paragraphs.

## [2026-09-19] update | workflows/qa-platform-sprint-kickoff.md
- Sprint `Done` is never cleared, so scope Completed by `completed_at` after the
  previous dated packet. Board column and `completed` disagree both ways. Re-walk
  umbrella parent chains every packet; parenting changes between sprints.

## [2026-09-19] update | tooling/asana-api-cloud-agents.md
- `memberships` on a project-scoped task list returns sections from every board the
  task is multi-homed into; filter by the target board's section gids.

## [2026-09-18] update | repos/mobile-automation-waits.md
- Empty Android Chats after a messages deep link: remount via Unread/All chip ids,
  not celery or a community GET. `wait_for_condition` poll_freq `1` is one second.

## [2026-09-18] update | repos/mobile-automation-waits.md
- Community create `sendbird_channel_url` is not a Chats-list ready signal; the 1:1
  Sendbird verify step does not cover that wait.

## [2026-09-18] update | repos/mobile-automation-waits.md
- Hyperview ready is three layers: native fetch loader id, namespaced XML `id` as
  RN `testID`, then optional `replace-inner` content. XML xmlns and page-object
  `hyperview` are not identity; `wait_for_screen_id` is a text regex.

## [2026-09-18] update | tooling/sentry-mobile-rn-upgrade-filter.md
- Join Amplitude product `userId` to Sentry `user.id`. Amplitude events after the crash timestamp
  are a relaunch; cut at Sentry time. Amplitude does not log picker-wheel opens.

## [2026-09-18] update | workflows/mobile-android-login-local.md
- Reverting a yarn patch via `git checkout` leaves `node_modules` patched; rebuilds then ship ids
  absent from every tracked file, which read as pre-existing app behavior in a UI dump. Run
  `yarn install` and grep the installed file after touching `.yarn/patches`, and check
  `node_modules` when a dump shows an id the source does not explain.

## [2026-09-18] update | tooling/staging-hyperview-picker-repro.md
- Do not treat Sentry `device.free_memory` as jetsam pressure on iOS; a few hundred MB free on an
  8–12 GB phone is normal. Check `device.class` across the issue before claiming a slow-device
  pattern. This crash is ~95% high-class.

## [2026-09-18] create | tooling/third-party-compose-sdk-locators.md
- Vendor SDK screens dump as one ComposeView subtree with no `resource-id` under our own package
  name; `testTagsAsResourceId` lives in the vendor's composables, so no app-owned id is reachable.
- These locators are the first to break when `@element(id=)` flips from accessibility id to
  resourceId: vendor-screen lookups fail while in-app ones on the same screen still pass.

## [2026-09-18] update | tooling/sentry-mobile-rn-upgrade-filter.md
- Count `ui.lifecycle` `beingPresented` repeats across events to get the real repro cadence; the
  stack names the victim, not the precondition. Bound the OS floor from the `os` tag first, and
  check `environment`/`release` for internal flavors before designing an in-house repro.

## [2026-09-18] update | tooling/sentry-mobile-rn-upgrade-filter.md + staging-hyperview-picker-repro.md
- Map crashing `dist` → git tag → `package.json` before accepting "cannot reproduce": same marketing
  version can carry a different RN patch. A release with ~1 user is not a clean control.
- Picker crash is RN 0.86.3-only in the field; 0.86.2 shipped to ~nobody. Build the 0.86.3 tag.

## [2026-09-18] update | tooling/staging-hyperview-picker-repro.md
- Cell-purge crashes need memory pressure, not just the right gesture: crashing devices sat at ~5%
  free RAM. Force it (simulator memory warning, Zombie Objects diagnostic) instead of hand-racing.
  Read `device.free_memory` vs `memory_size` in the event context before writing a manual repro.

## [2026-09-18] update | tooling/staging-hyperview-picker-repro.md
- Picker crash has two routes. Internal beta builds reproduce via idle: open the wheel, stop
  touching, background/foreground, wait 6–10 min, then touch. Store users hit it via 3+ consecutive
  picker opens 2–5s apart. Continuous spinning reproduces neither. iOS 26.4+, release-config build.
- Internal and store populations can need different routes to one fingerprint; check the internal
  environment's own events before handing someone the store repro.

## [2026-09-18] update | tooling/staging-hyperview-picker-repro.md
- Open my-dynamic-form in-app via `instawork://hv-dev/home/worker-app-my-dynamic-form`
  (Django URL name). Path `/artforms/my-dynamic-form` is not a deep link; `hv-dev` is non-prod only.

## [2026-09-18] update | tooling/staging-hyperview-picker-repro.md
- Fabric `UIPickerView` hit-test crash: iOS 26 + my-dynamic-form / storybook
  PickerField, touch the wheel while the modal is presenting.

## [2026-09-18] update | tooling/sentry-mobile-rn-upgrade-filter.md
- iOS `_endSuspendingUpdates` + `UIPickerView hitTest` is one crash: group by
  Fabric picker modal + native stack, not by which Hyperview screen opened it.

## [2026-09-18] update | workflows/mobile-android-login-local.md
- Gradle's JS bundle task ignores `node_modules`, so a yarn-patch-only change rebuilds an APK
  that still embeds the previous bundle. Compare `assets/index.android.bundle` md5 across APKs;
  clear `generated/assets/react/<variant>` to force it.
- Orphaned `io.appium.uiautomator2.server` makes `uiautomator dump` exit 137; force-stop it.
- Automation APK carries a bundletool-normalised `Jan 1 1981` mtime; judge build freshness by
  ctime. Gradle output is `outputs/bundle/automationRelease`, not an `apk/automation*` dir.

## [2026-09-18] update | workflows/xpath-to-id-app-target-resolution.md
- Sendbird chat rows are UIKit; APIv2 already returns `sendbird_channel_url` on community
  create and relationship GET. Stashing it in e2e does not locate the row until native
  `testID` is on the Pressable.

## [2026-09-17] update | tooling/appium-native-camera-locators.md
- iOS video start and stop both keep `name` `VideoCapture`; a stop locator on label
  `Stop Recording Video` is unnecessary.

## [2026-09-17] create | tooling/appium-native-camera-locators.md
- OS camera controls: one `IdSelector` Matches alternating Android resource id and iOS name
  replaces the platform branch. `id=` is resource id, so `id="Done"` / `id="Take photo"` miss
  even when the dump shows those as content-desc. Android reuses `shutter_button` for confirm.
- iOS camera `name` and `label` disagree; `IdSelector` predicates on `name`, so the visible
  caption (`Take Picture`, `Use Photo`) is never a usable id token.

## [2026-09-17] update | workflows/xpath-to-id-app-target-resolution.md
- An OS-owned duplicate text node cannot be hidden from Appium: instatest sets
  `ignoreUnimportantViews: False`, so `importantForAccessibility` binds TalkBack, not the dump.
- Third-party RN wrappers may forward an explicit prop allowlist; inherited `ViewProps` silently
  never reach the native node. Read the wrapper's render before relying on a passthrough prop.

## [2026-09-16] create | tooling/ios-accessible-hides-child-testids.md
- XCUITest prunes the subtree of any view with `isAccessibilityElement`; RN `Touchable*` sets it
  by default, so child `testID`s show on Android and vanish on iOS. `accessible={false}` restores
  them; the wrapper then loses its aggregated `label`/`content-desc`.

## [2026-09-16] update | repos/hyperview-local-mobile-sync.md
- Prove which JS the app holds before blaming sync: automation builds embed `main.jsbundle` and
  ignore Metro (no dev menu). Diff a marker count between Metro's served bundle and the installed
  app's bundle via `xcrun simctl get_app_container`.

## [2026-09-16] update | workflows/xpath-to-id-app-target-resolution.md
- Android picker: `testID` on the Spinner host cannot own `.text`; an RN `Text` overlay can.
  Neutralise Instawork Android `field-text-style` compensation margins on that overlay.
  Instatest `id=` is UiAutomator `resourceId`, not Inspector `By.ID` autocompletion.

## [2026-09-16] update | repos/hyperview-local-mobile-sync.md
- Debug `fullstoryDevelopmentDebug` `No space left on device` is disk full, not Fullstory.
  Reclaim `app/build`, `app/.cxx`, and Android `sdk/.temp`.

## [2026-09-16] update | repos/hyperview-local-mobile-sync.md
- Yarn 4 vs Classic lockfile: skip `yarn sync`; run `ts-node ./scripts/sync` from
  `node_modules` instead of `yarn install` under Yarn 4. Dest path must be relative;
  an absolute path is joined onto the Hyperview repo.
- Root cause of the regression: corepack's global default Yarn flipped to 4.x (cache still holds
  1.x), so a repo with no `packageManager` pin silently changes toolchain. Not a broken install.

- First `yarn run:android` is a native compile. NDK 28 clang-19 crashes in `.cxx` are toolchain
  flakes; delete `applicant-app/android/app/.cxx` and retry. Not a Hyperview JS issue.

## [2026-09-16] update | workflows/xpath-to-id-app-target-resolution.md
- Inner picker `testID` labels the Spinner / UIPickerView host, not `text1` or iOS FieldLabel.
- Hyperview dropped the iOS-testID/Android-accessibilityLabel helper for plain `testID` on both;
  check which shape the build ships before concluding an Android id is impossible.
- Moving an id off a wrapper breaks index-based lookups; prefer a derived `{id}/…` child id.
- Shared MDS input steps both click and read `.text` off one element, so "one id for both" needs a
  node doing both; OS-inflated value children (`text1`) can never take an app id.

## [2026-09-14] update | workflows/mobile-android-login-local.md
- After a long Android local run, session create can stay wedged through cold boot.
  Uninstall Appium Settings + UiAutomator2 server packages, then retry the failed
  scenario alone.

## [2026-09-14] update | repos/mobile-automation-waits.md
- Screen-loaded: Gherkin still instantiates only; reuse `wait_for_load` /
  `is_loaded`. Applicant override waits on `screen_id` text regex, not
  `load_check_selector`; Hyperview chrome id is not content-ready.

## [2026-09-14] create | tooling/staging-hyperview-picker-repro.md
- Staging select-wheel repro: skip v2/OpenSearch; use my-dynamic-form or a preview artform intro URL.

## [2026-09-14] update | workflows/app-store-connect-unsubmit-build.md
- Pause is only for a phased iOS release (≤30 days). A 100% live version cannot be paused.

## [2026-09-14] update | preferences/global.md
- Store-version error triage: only flag issues with more events on this
  version than prior ones; 1–4 unique users on that version is not critical.

## [2026-09-14] update | tooling/sentry-mobile-rn-upgrade-filter.md
- After a Fabric hotfix, query the hotfix `release:` for native SIGSEGV, not only
  `firstRelease:` of the hotfix. Enumerate crashes split by caller into multiple
  issue ids; sum by title.

## [2026-09-11] update | preferences/global.md
- Added: do not include test-run invocation details unless the user asks for them.

## [2026-09-11] update | workflows/mobile-android-login-local.md
- Compose from the repo root needs `-e HOST_WORKSPACE=<repo>/e2e/mobile` or Appium looks for the
  APK under `<repo>/applicant-app/bin`. `--name '[25022]...'` is a regex; use the digits only.

## [2026-09-11] update | workflows/xpath-to-id-app-target-resolution.md
- A shared step's widget helper belongs in a widget mixin mixed into the hosting screens
  (`GigsListTabBarMixin` pattern), not on a screen base; ids stay on the `*Component` class.
- Component-declared `@elements` resolve scoped to the wrapped element, so the lookups live on the
  mixin. Screen bases cannot import components (cycle); judge cycles from module-body imports only.

## [2026-09-11] update | repos/mobile-automation-waits.md
- `driver_context.wait_until` returns False on timeout instead of raising, takes any predicate, and
  zeroes the implicit wait while polling: wait for a *matching* element, then re-read and assert.

## [2026-09-11] update | repos/mobile-vendored-instatest.md
- instawork keeps its own vendored copy at `instawork/lib/instatest/instatest/`; `e2e/mobile/lib/instatest`
  is an empty compose mount point, so framework source looks missing from inside `e2e/mobile`.

## [2026-09-11] update | workflows/xpath-to-id-app-target-resolution.md
- Hyperview derives child ids like `{id}/field-label` per element, not by convention: `hv-date-field`
  has one, `hv-picker-field` does not. Verify the suffix in the element source at the hyperview tag
  `core-mobile/package.json` pins before writing it into a locator.

## [2026-09-11] update | workflows/xpath-to-id-app-target-resolution.md
- Stale iOS Automation `.app` (embedded bundle) can hide `{testID}/text-input` and look like a
  platform split; check e2e bin mtime vs the Android APK before keeping an iOS xpath.

## [2026-09-11] update | preferences/global.md
- Added: scope locator/test fixes to cases proven broken by a report or run; do not pre-emptively
  change inference-only matches, let the next run surface them.

## [2026-09-11] update | workflows/xpath-to-id-app-target-resolution.md
- Validation scoping for a locator-class fix: one scenario per unique locator, not per failing test.
- Long scenarios mask later instances of the same defect, so each fix reveals the next a run later;
  sweep typed fields statically instead of iterating one field per device run. Score by which step
  failed, not pass/fail.

## [2026-09-11] update | workflows/mobile-ci-allure-failure-triage.md
- Runs can target a per-PR `demo-<pr>` backend, so server-rendered locator ids come from that
  branch's head; verify ancestry of the id commit against the served SHA. Sibling exact-id
  lookup passing in the same second is the tell for a missing template id, not a driver break.

## [2026-09-11] update | workflows/xpath-to-id-app-target-resolution.md
- `/text-input` migration: repoint the existing id constant; do not add a parallel
  element + platform selector pair for the same field.

## [2026-09-11] update | repos/mobile-automation-waits.md
- `wait_while_loading_screen_exists` is a 5s ProgressBar xpath miss then a no-op;
  full `page_source` equality is not an at-top signal for pull-to-refresh.

## [2026-09-10] update | repos/mobile-automation-waits.md
- Screen readiness is a positive loaded id, not copy and not a global shimmer-gone wait;
  absence-only loader waits race if the skeleton has not mounted yet.

## [2026-09-10] update | workflows/mobile-android-login-local.md
- After a long local Android scenario, the next Appium session can fail on
  `hidden_api_policy` adb timeout; retry that scenario alone, do not change locators.

## [2026-09-10] update | workflows/mobile-android-login-local.md
- instawork e2e/mobile compose cwd is not a git work tree; mount repo `.git` at
  `/workspace/.git` or applicant login_steps `git rev-parse` exits 128 at import.

## [2026-09-10] update | workflows/xpath-to-id-app-target-resolution.md
- Shared Behave steps that used driver-global wait_for cannot move onto one screen class.
  Grep feature usages first; put the helper on the shared component. `@elements` does not wait.

## [2026-09-10] update | workflows/mobile-android-login-local.md
- The automation APK bakes its JS bundle, so RN source edits need `yarn build:android:automation`;
  verify a shipped id by byte-searching `assets/index.android.bundle` inside the zip, not `strings`.

## [2026-09-10] update | workflows/xpath-to-id-app-target-resolution.md
- RN `Touchable > View testID > Text` rows: id node has empty text, label is on the unlabelled
  child. Give the `Text` its own sibling-key id in `ids.ts` and match on it; fix missing `testID`
  in the typography prop types rather than suppressing the type error at the call site.

## [2026-09-09] update | workflows/xpath-to-id-app-target-resolution.md
- RN TextField: container keeps the passed testID; the EditText is `{testID}/text-input`.

## [2026-09-09] update | workflows/xpath-to-id-app-target-resolution.md
- MDS autocomplete puts `id_field` on both the wrapper ViewGroup and the child EditText.
  Id lookup returns the wrapper first; placeholder/value is on the EditText.

## [2026-09-09] update | workflows/xpath-to-id-app-target-resolution.md
- `Matches` patterns must use `[0-9]`, not `\d`: iOS NSPredicate string literals eat the
  backslash, so the same pattern passes on Android and returns nothing on iOS. Also:
  `@element(id=)` is always literal `Equals`; regex requires an explicit Matches operator.

## [2026-09-08] update | tooling/instawork-automation-api.md
- Demo/QA bulk cleanup: delete shift-groups → transactions → company-payment-sources →
  companies → worker dependents → incentives via `delete-by-worker` + exact `incentive__goal`
  → workers. Company/worker DELETE HTML 500s are leftover FK rows, not missing endpoints.

## [2026-09-08] update | workflows/xpath-to-id-app-target-resolution.md
- Entity-id locators must resolve from the current account’s grouping, not a global worker
  list or a `context.*_id` stash. Create snapshots omit write-only `worker_id`; list unique
  groups until the logged-in worker is on one, then pick a coworker.

## [2026-09-08] update | workflows/xpath-to-id-app-target-resolution.md
- Ids embedding an entity id must take that value from screen scope, not a global test-data
  list; the replaced XPath was self-scoping, so the regression is migration-only. `find()`
  returns `default` on a miss and takes `timeout_s`, so probe candidates instead.

## [2026-09-08] update | workflows/ruff-noqa-cleanup-pipeline.md
- Instatest + e2e/mobile Ruff CI is `lint-e2e`; local recipe is `just ruff-e2e`. Root
  `lint-ruff` still excludes `lib/instatest`.

## [2026-09-08] update | workflows/ruff-noqa-cleanup-pipeline.md
- PLC0415 peel: nested imports that break cycles cannot become `from module import Name`;
  use a module alias (often both sides) or TYPE_CHECKING for annotations-only.

## [2026-09-08] update | workflows/xpath-to-id-app-target-resolution.md
- IdSelector/partial_id on florent uses Android resource-id; legacy content-desc xpaths and
  get_attribute("content-desc") uniqueness must be realigned (and dumped) when HV drops the
  accessibilityLabel testID hack.

## [2026-09-08] update | tooling/sentry-mobile-rn-upgrade-filter.md
- Hotfix `firstRelease:` is not the RN-ship marker; prove RN from git tags and
  compare Fabric issue `release` tags. WebView/syscall AppExitInfo ANRs are not
  Fabric.

## [2026-09-08] update | workflows/xpath-to-id-asana-task-format.md
- `_android` / `_ios` same-stem siblings share one id; migrate both even if the map lists one.

## [2026-09-08] create | tooling/git-shallow-fetch-unrelated-histories.md
- A depth-1 fetch of master in a full clone writes `.git/shallow` and leaves master as a
  one-commit island, so merge reports unrelated histories. Unshallow; do not allow-unrelated.

## [2026-09-08] update | preferences/global.md
- Prefer full MDS `{namespace}/…` ids over `partial_id`; namespace is the view attribute.

## [2026-09-08] update | workflows/xpath-to-id-app-target-resolution.md
- MDS `{id}-loading` / `{id}-disabled` siblings are also hit by `partial_id` Contains; use
  anchored `Matches` for namespaced suffixes.

## [2026-09-07] update | workflows/self-review-workflow.md
- `lint-e2e` unused-step scan runs on HEAD, not the three-dot PR diff. A step missing on
  GitHub `master` but present on the branch is stale-base, not a new unused definition.

## [2026-09-07] update | workflows/instawork-local-hiring-opensearch-setup.md
- Disabling the grouping constance flag to fix empty browsing also strips the grouped-jobs UI,
  so those scenarios find nothing. It is DB-backed and persists; check it, don't assume.

## [2026-09-07] update | workflows/self-review-workflow.md
- `lint-e2e` unreferenced-Behave-step scan can fail on master-identical unused
  steps; compare the cited path to master before attributing it to the PR.

## [2026-09-07] update | workflows/mobile-android-login-local.md
- `instatest list` needs a nested subcommand (`devices` / `applications` / `servers`);
  bare `list` is an argparse error, not a successful no-op.

## [2026-09-07] create | workflows/instawork-local-hiring-opensearch-setup.md
- Jobs tab empty locally is an infra gap, not missing data: OpenSearch is behind the `full`
  compose profile. Sync lands jobs in MySQL only; `upsert_all_jobs()` is the step that makes
  them searchable, and reconcile does not do it despite its docstring.

## [2026-09-07] update | repos/instawork-prod-only-settings-local-500.md
- `instawork/local_secrets.py` is the gitignored, last-imported settings override hook — use it
  for dummy URLs instead of tracked settings modules. Added: the crashing URL line sits outside
  the service's `try`, and a constance gate may not cover every caller of the same service.

## [2026-09-07] create | repos/instawork-prod-only-settings-local-500.md
- `NoneType` URL crash = prod-only setting unset locally; prefer the upstream constance gate,
  and verify the fallback on the active branch before trading a 500 for an empty list.

## [2026-09-07] update | workflows/xpath-to-id-app-target-resolution.md
- Unlabeled Hyperview views expose 32-char hex resource-ids that are style hashes, not locators.

## [2026-09-07] update | repos/mobile-automation-waits.md
- Three different wait defaults: `element()` 5s, `wait_for` 10s, suite `wait_for_element` 3s.
  Don't carry a `timeout_s` override across a conversion; `--timeout` raises them all at once.

## [2026-09-07] update | repos/mobile-automation-waits.md
- `element(...)` default wait is 5s (`explicit_wait` env), not the 10s `wait_for` DEFAULT.

## [2026-09-07] update | workflows/qa-platform-sprint-kickoff.md
- Week-two This week is the spoken plan: drop themes the user takes out of the sprint
  from the canvas (keep them in markdown inventory); leftover volume they say to push
  stays speakable even when the pile is large.

## [2026-09-07] update | preferences/global.md
- New: never label a cell/column/card/stat with a bare identifier code (`Mn`, `P4.x`, lint code);
  show the readable title. Generalizes the kickoff-only "speakable focus lines" gotcha.

## [2026-09-07] update | workflows/qa-platform-sprint-kickoff.md
- Week-two Last week table is `Milestone | Slice | Task | Owner`; milestone is an ancestor hop,
  and a uniform column is a valid answer. Cell shows the readable `Mn: <name>` title minus the
  mission prefix; the bare `Mn` code is not speakable.

## [2026-09-07] update | tooling/asana-api-cloud-agents.md
- Parent-chain walk to find the `milestone` ancestor; one `parent` hop returns an epic, and
  title prefixes are derived, not source.

## [2026-09-07] update | tooling/asana-api-cloud-agents.md
- Board URL `/board/<id>` is a view gid; use project sections or project tasks, not that id as
  `section`.

## [2026-09-07] update | workflows/qa-platform-sprint-kickoff.md
- Week-two canvas is a check-in (last week + this week's focus). Keep Carry Over / New In
  Sprint / Full Workload in the markdown only.

## [2026-09-07] update | tooling/cursor-canvas-authoring.md
- Kickoff vs week-two canvases use different layouts; do not copy kickoff inventory chrome
  onto week two.

## [2026-09-04] update | workflows/ruff-noqa-cleanup-pipeline.md
- B012 peel (`return` in `finally`) swallows re-raises; treat like E722 (both-app both-platform
  smoke) unless the `except` already swallows.

## [2026-09-04] update | workflows/mobile-android-login-local.md
- `python -m instatest` must start in the app dir (`./configuration` frozen at import). `list`
  has no `-v`; default console hides DEBUG; `list_config` uses an unconfigured `instatest` logger.

## [2026-09-04] update | repos/instawork-mobile-e2e-shared-pythonpath.md
- Compose PYTHONPATH now `/workspace:/workspace/lib/instatest` (CI-matching); old instatest-only
  claim corrected, verify the service `environment` before assuming.

## [2026-09-04] update | workflows/mobile-browserstack-qa-docker.md
- instawork `e2e/mobile`: framework is the mounted `../../lib/instatest`, no pip copy in the image;
  `.vscode/scripts/run-automation.sh` is the canonical compose-run shape (application platform
  suffix, `.git` mount, HOST_WORKSPACE override); `.env` localhost URLs drift against the QA
  `BACKEND_INSTANCE` default, override with `-e`.

## [2026-09-04] update | workflows/ruff-noqa-cleanup-pipeline.md
- E722 peel in Instatest is a catch-narrowing behavior change; follow with both-app both-platform
  mobile smoke, not lint-only.

## [2026-09-03] update | workflows/ruff-noqa-cleanup-pipeline.md
- T201 peel: logging or delete for debug prints; keep `print` for CLI. `sys.stdout.write`
  is not a fix.

## [2026-09-03] update | workflows/ruff-noqa-cleanup-pipeline.md
- Executor follows the card body (Scope/Steps/Done when). All board cards use the noqa-peel
  shape; ignore-Prompt is obsolete.

## [2026-09-03] update | tooling/asana-api-cloud-agents.md
- Move a task already on a board with `POST /sections/<gid>/addTask`. `addProject` is for
  first-time multi-home.

## [2026-09-03] update | workflows/ruff-noqa-cleanup-pipeline.md
- Card body is the spec: Scope is Code + one Path; Steps peel `# noqa` then fix; Done when is
  a draft PR that keeps the Ruff CI job green. Gold card is E712 on e2e/mobile.

## [2026-09-03] update | workflows/ruff-noqa-cleanup-pipeline.md
- Same Ruff code in `lib/instatest` and `e2e/mobile` is two cards. Executor fails if the diff
  crosses trees or the card names both / neither path.

## [2026-09-03] create | workflows/ruff-noqa-cleanup-pipeline.md
- Ruff board executor peels `# noqa` for title codes then fixes; ignore the card's old autofix
  Prompt; `[e2e/mobile]` is `e2e/mobile`, `[Instatest]` is `lib/instatest`.

## [2026-09-03] update | tooling/asana-api-cloud-agents.md
- Board column work is REST-only: list `/projects/<gid>/sections` for column gids, place tasks
  with `/tasks/<gid>/addProject` (additive, works for subtasks), verify via
  `/sections/<gid>/tasks`. REST is also the fallback when local MCP returns `network_error`.

## [2026-09-02] update | repos/mobile-step-definition-conventions.md
- Anchored id regex via `Selector.by_id(..., operator=MobileOperator.Matches)` is the cross-platform
  answer to prefix collisions; `[0-9]` over `\d` because of UiSelector string embedding.

## [2026-09-02] update | workflows/xpath-to-id-app-target-resolution.md
- Django `id_<field>_<index>` plus a suffixed child is a `partial_id` trap; use exact indexed ids.

## [2026-09-02] update | repos/mobile-step-definition-conventions.md
- `steps.usage` is not a CI unused-step check: Gherkin-only, one app, environment.py still loads.

## [2026-09-02] update | repos/mobile-step-definition-conventions.md
- Unused step definitions: `behave --dry-run -f steps.usage --no-summary`; Instatest
  needs `--behave-args` (`+f` not `-F`); Gherkin-only, tag/app/exclude_re false positives.

## [2026-09-02] create | workflows/xpath-to-id-asana-task-format.md
- Locked Asana card format for the XPath-to-id pipeline: title prefix, four sections only,
  shared Steps, Locator map fields with Candidate id (not App target id), optional attachment
  as screen context only. Cross-linked from app-target-resolution.

## [2026-09-02] update | workflows/xpath-to-id-asana-task-format.md
- After `</h2>`, body must start on the same line with no whitespace; `</h2>\n` renders as a
  blank under the heading. One `\n\n` between sections only.

## [2026-09-02] update | workflows/xpath-to-id-asana-task-format.md
- Description spacing: no blank after headings, one blank between sections; no PR link in the
  body (PR stays in comments).

## [2026-09-02] update | workflows/xpath-to-id-app-target-resolution.md
- Bulk-corrected Candidate ids across xpath-to-id board: attire is OS camera not HV capture;
  artforms options are `id_step-<field>_N`; end-time radios still need an added option id;
  Sendbird channel rows have no app id in checkout.
- Pipeline field renamed from App target id to Candidate id. Verify against live screen owner
  before migrating; do not invent a new id when the candidate misses.

## [2026-09-02] update | tooling/browserstack-app-automate-api-cloud-agents.md
- BrowserStack has no post-session page-source API; Appium logs only contain hierarchy XML if the
  test called getPageSource. Locator misses alone do not dump the DOM.

## [2026-09-02] update | preferences/documentation.md
- Asana comments must be plain `text`; `html_text` is stored as literal escaped markup and the
  reader sees raw tags. Rich text applies to task descriptions only.

## [2026-09-02] create | workflows/xpath-to-id-app-target-resolution.md
- XPath-to-id App target id must follow live screen owner: Hyperview template `id=` /
  Django `id_<field>` vs native `ids.ts`. An OR xpath that includes RN paths is not
  proof RN is live; tracking event names are not element ids.

## [2026-09-02] update | tooling/cursor-canvas-authoring.md
- Canvas has no Image component: attach Allure failure PNGs via workspace download + Button
  openFile, one representative frame per task from all_error_details screenshot field.

## [2026-09-01] create | repos/mobile-automation-waits.md
- Mobile e2e wait primitives inventory plus two traps: a wait step implemented as
  `implicitly_wait` does not pause the scenario and leaks a session-wide timeout, and
  `screen should be loaded` only instantiates the screen class so it cannot replace a sleep.
  Poll-interval sleeps inside retry loops convert to `wait_for_condition` rather than being deleted.

## [2026-09-01] update | preferences/documentation.md
- Asana `html_notes` has no table support and renders `<pre>` as a line-numbered code box, so
  tabular task data must be nested lists. Spacing: one newline between sections, none after a
  heading.

## [2026-09-01] update | tooling/ruff-enable-excluded-tree.md
- Run `ruff check --fix` before `ruff format`; lint autofixes can require a later format pass.

## [2026-08-31] create | workflows/app-store-connect-unsubmit-build.md
- Unsubmit an unpublished App Store version: withdraw from review to Developer Rejected, remove
  the Build with the red minus, attach a higher CFBundleVersion. Version rows cannot be deleted.

## [2026-08-31] create | workflows/play-console-discard-release.md
- Discard an unpublished Play Console release: Publishing overview Remove changes, then Discard
  release on the track version. Production dashboard and Create new release are not discard.

## [2026-09-01] update | tooling/instawork-automation-api.md
- Booking prerequisites seeding: eligibility via run-celery-task + SpecialRequirementFactory;
  success guide via worker-score etiquette_score < 80; optional attire upload needs preset only.

## [2026-08-31] update | repos/mobile-pre-booking-hold-review.md
- QA Constance for hold eligibility can be 360h (15 days); always read live value before seeding.
  Grant worker_access with `worker` (not worker_id).

## [2026-08-31] create | repos/mobile-pre-booking-hold-review.md
- "Please review the shift details" / disabled Continue booking needs an unbooked marketplace
  gig (not API-booked), starts beyond GIG_HOLD_GIG_DETAILS_TIME_TO_START_HOURS, and a worker with
  no completed shifts at that company; Book in-app creates HOLD_GIG_DETAILS_REVIEW.

## [2026-08-31] create | workflows/integration-branch.md
- Integration PRs only collect other PRs. Commits must come from merging those PRs, not unique
  work on the branch. Name `integration/<topic>`, add the `integration` label, and list `#PR`
  (or a hash if there is no PR) under "Integrate the following PRs / commits:".

## [2026-08-31] update | tooling/instawork-automation-api.md
- Gig template GET omits company/business; resolve company via template contacts then company_user.
- Required attire-upload booking prerequisite is a company-scoped attire_upload vetting module plus
  an attire preset, not a shift-group field. Leave the shift unbooked and grant worker_access.

## [2026-08-31] update | preferences/global.md
- Never send Slack messages unless the user explicitly asks. Completing related work is not
  permission to post.

## [2026-08-31] update | workflows/asana-bug-report.md
- Device-screen crop must be measured (longest contiguous light run bounded by bezel bands) and
  eyeballed on a rendered frame; a slightly narrow crop silently clips right-edge UI and still
  looks valid. Added a section for replacing evidence on a live ticket: re-fetch `html_notes` right
  before the write, regex-swap attachment gids only, then delete the superseded attachment.

## [2026-08-31] update | workflows/qa-platform-sprint-kickoff.md, tooling/cursor-canvas-authoring.md, index.md
- Canvas step 6 now locks speaking structure: Mission accent + openFile button, stats +
  definition table, Where we are (mission prose), What is next? (contributor cards),
  Blocking, Completed backup. Canvas-only milestone omit allowed; focus lines must be
  speakable. Authoring page matches Cards + UsageBar, not topLeftLabel prose.

## [2026-08-31] update | tooling/cursor-canvas-authoring.md
- Stacked `BarChart` cannot label "who works on what"; use per-person `UsageBar` with
  `topLeftLabel` / `topRightLabel` and a shared `total` for comparable lengths.

## [2026-08-31] update | workflows/qa-platform-sprint-kickoff.md
- Kickoff markdown and canvas both open with Completed, Carry Over, New In
  Sprint, Full Workload plus the same definition table. Canvas step 6 copies
  those labels; the markdown skeleton is the packet source.

## [2026-08-31] update | tooling/cursor-canvas-authoring.md, workflows/qa-platform-sprint-kickoff.md
- Kickoff headline stats stay short: Completed, Carry Over, New In Sprint, Full
  Workload. Meaning lives in the definition table under the strip.

## [2026-08-31] update | tooling/cursor-canvas-authoring.md
- `Stat` has no sublabel, so the label must name what is counted and its source. Pair a
  stat strip with a `Number | What it counts | Source` table; never show a derived total
  without its inputs.

## [2026-08-31] update | workflows/qa-platform-sprint-kickoff.md, tooling/cursor-canvas-authoring.md, index.md
- Coming sprint is Planning `Needs Triage` and `Next` under confirmed umbrellas, not a
  second user-confirm pass. After the markdown packet, generate a canvas from that file.

## [2026-08-31] update | tooling/asana-api-cloud-agents.md
- Cloning a task's formatting: fetch the model task with `opt_fields=html_notes` and copy the
  markup verbatim rather than re-authoring from the rendered view. Documented tag allowlist is
  incomplete — `<table>`/`<tr>`/`<td>` work; `<pre>` needs `<`/`>` escaped.

## [2026-08-30] create | tooling/cursor-canvas-authoring.md, index.md
- Canvas authoring: managed `canvases/` path is mandatory, SDK prop types reject React `key`
  (no `.map()` over `Stat`), `React.ReactNode` namespace unavailable, emoji forbidden, empty
  sections must collapse into a coverage table, `Canvas TypeScript check` is the truth signal.

## [2026-08-30] update | workflows/qa-platform-sprint-kickoff.md
- Plan section is `# 🚀 New in coming sprint` (week two: `# 🚀 New this week`). Carry-over
  stays in the table above; this section is new adds by milestone only.

## [2026-08-30] update | workflows/qa-platform-sprint-kickoff.md, index.md
- Kickoff packet format is a required file skeleton. Copy structure from the latest
  kickoff packet, not its inventory. Coming sprint is user-confirmed new adds only;
  previous-sprint Sprint To-Do is always carry-over.

## [2026-08-30] update | workflows/qa-platform-sprint-kickoff.md
- Previous-sprint Sprint `To-Do` is carry-over, including not-started. Do not omit it.
  Coming sprint is only work the user is newly adding.

## [2026-08-30] update | workflows/qa-platform-sprint-kickoff.md
- Leftover Sprint To-Do stays in Carrying over, including parked items. Do not omit it
  from the packet. Coming sprint is still new-adds only.

## [2026-08-30] update | workflows/qa-platform-sprint-kickoff.md
- Sprint leftover `To-Do` on an in-flight slice is carry-over, not coming-sprint new work.
  Coming sprint is only what the user is newly adding.

## [2026-08-30] update | workflows/qa-platform-sprint-kickoff.md
- Coming-sprint new work must be on QA Platform - Sprint (or the user explicitly adds it).
  Planning/Backlog-only and no-board tasks stay out.

## [2026-08-30] update | workflows/qa-platform-sprint-kickoff.md
- For coming Sprint is new work only. Never repeat In Progress / In Review from Carrying over.
  A milestone with only carry-over still gets the ℹ️ empty note.

## [2026-08-30] update | workflows/qa-platform-sprint-kickoff.md
- Coming-sprint (and week-two) work uses the same Feature / Summary / Tasks table as
  carry-over. Heading N is two-week-plan Asana tasks only. Empty milestones stay ℹ️.

## [2026-08-30] update | workflows/qa-platform-sprint-kickoff.md
- Coming-sprint work uses the same Feature / Summary / Tasks table as carry-over. Empty
  milestones stay `ℹ️`. Heading total is the two-week plan only, not queued slices.

## [2026-08-30] update | workflows/qa-platform-sprint-kickoff.md
- Recap headings are `## ✅ Completed`, `## 🔄 Carrying over`, then `# 🚀 For coming Sprint`
  (week two: For this week).

## [2026-08-30] update | workflows/qa-platform-sprint-kickoff.md
- Recap/plan break is only `# For coming Sprint` (week two: this week). No `---` and no `<hr>`.

## [2026-08-30] update | workflows/qa-platform-sprint-kickoff.md
- Recap/plan break is an 8px HTML `<hr>` plus `# For coming Sprint` (week two: this week).
  Do not use markdown `---`; it is too thin in preview.

## [2026-08-30] update | workflows/qa-platform-sprint-kickoff.md
- After carry-over, `---` then `## For coming Sprint` (week two: `## For this week`)
  before any `Mn:` heading. That is the recap vs plan break.

## [2026-08-30] update | workflows/qa-platform-sprint-kickoff.md
- Empty milestone note is `ℹ️ Nothing here for the starting sprint.` (week two: this week)
  so the empty state is visible in a scan.

## [2026-08-30] update | workflows/qa-platform-sprint-kickoff.md, preferences/documentation.md
- One task per table row. No multi-line cells at all: the formatter strips `<br>` and flattens
  `<ul><li>` into inline `- `, so shared columns are filled on the first row of the group only.

## [2026-08-30] update | workflows/qa-platform-sprint-kickoff.md, preferences/documentation.md
- Superseded same day: `<ul><li>`/`<div>` in table cells did not survive the formatter either.

## [2026-08-30] update | workflows/qa-platform-sprint-kickoff.md
- Carrying-over Tasks cells start with `- `. Completed stays ✅.

## [2026-08-30] update | workflows/qa-platform-sprint-kickoff.md
- Carrying over uses the same Feature / Summary / Tasks table. No ✅. ❌ does not count toward N.

## [2026-08-30] update | workflows/qa-platform-sprint-kickoff.md
- Completed assignee is `[Firstname]` after the task link, not a nested bullet.

## [2026-08-30] update | workflows/qa-platform-sprint-kickoff.md
- Completed assignee is a nested line under the task (`&nbsp;&nbsp;- Firstname`), not inline.

## [2026-08-30] update | workflows/qa-platform-sprint-kickoff.md
- Completed Tasks column uses ✅, not `-`. Carry-over lists stay dashes.

## [2026-08-30] update | workflows/qa-platform-sprint-kickoff.md
- Feature count is `[n tasks]` on the line after the breadcrumb, not parentheses.

## [2026-08-30] update | workflows/qa-platform-sprint-kickoff.md
- Feature count sits on the line after the breadcrumb: `<br>(n tasks)`.

## [2026-08-30] update | workflows/qa-platform-sprint-kickoff.md
- Completed table assignee suffix is `(Firstname)`, not `-- Firstname`.

## [2026-08-30] update | workflows/qa-platform-sprint-kickoff.md
- Feature breadcrumb count is `(n tasks)` / `(1 task)`, not a bare `(n)`.

## [2026-08-30] update | workflows/qa-platform-sprint-kickoff.md
- Completed table is Feature (milestone > theme > slice) | Summary | Tasks. Outcome text stays out of Feature.

## [2026-08-30] update | workflows/qa-platform-sprint-kickoff.md
- Feature cell is `**Name (n tasks)**` plus one high-level outcome sentence. Do not restate task titles.

## [2026-08-30] update | workflows/qa-platform-sprint-kickoff.md
- Completed table bullets are `- [title](url) -- Firstname`. Unassigned if none.

## [2026-08-30] update | workflows/qa-platform-sprint-kickoff.md
- Completed is a Feature | Tasks table (per-row counts, `<br>`-joined links) and replaces the separate Backup Done list.

## [2026-08-30] update | workflows/qa-platform-sprint-kickoff.md
- Milestone headings include a one-sentence Goal. Completed recap is counts (`Theme (n tasks)`), not a task dump.

## [2026-08-30] update | workflows/qa-platform-sprint-kickoff.md
- Packet opens with completed (ending sprint) and carrying-over bullets. Full Done list stays in Backup.

## [2026-08-30] update | workflows/qa-platform-sprint-kickoff.md
- Packet always emits every confirmed umbrella heading. Empty starting-sprint work is a note, not a skipped section.

## [2026-08-30] update | preferences/documentation.md
- Recorded preference: Asana task descriptions use rich text with real headings, never plain-text markdown.

## [2026-08-30] update | tooling/asana-api-cloud-agents.md
- `html_notes` is required on create too, not only on edit. `XML is invalid` on create is usually `<p>`/`<br/>`; never fall back to plain `notes`, which renders markdown literally.

## [2026-08-30] update | workflows/qa-platform-sprint-kickoff.md
- Never change Asana unless the user explicitly asks. Packet writes are not permission to write Asana.

## [2026-08-30] update | workflows/qa-platform-sprint-kickoff.md
- Packet filename includes date and mode so runs are kept. Same date and mode overwrites that file only.

## [2026-08-30] update | workflows/qa-platform-sprint-kickoff.md
- Packet is `qa-platform-sprint-packet.md` (overwrite, not wiki). Asana permalinks; ❌ if no task. Chat is the path only. Short, no filler.

## [2026-08-30] update | workflows/qa-platform-sprint-kickoff.md
- Removed Local/CI parity device-match sequencing. That is dry-run/sprint packet data, not runbook.

## [2026-08-30] update | workflows/qa-platform-sprint-kickoff.md
- Local/CI parity first slice is local debug/run devices matching CI; skins and screen-size later.

## [2026-08-30] update | workflows/qa-platform-sprint-kickoff.md
- Spoken lists are grouped by milestone, not by theme or person first.

## [2026-08-30] update | workflows/qa-platform-sprint-kickoff.md
- Sprint list API: section or project, not both. Umbrellas often have no membership. Match the Sprint slice by parent chain; owner-assigned work outside the umbrellas stays out unless the user adds it.

## [2026-08-30] update | workflows/qa-platform-sprint-kickoff.md
- Starting-sprint known list is carry-over. New subtasks are reasoned from in-progress work as suggestions only; the user confirms or substitutes.

## [2026-08-30] update | workflows/qa-platform-sprint-kickoff.md
- Kickoff main goal is the starting-sprint task list and who will work on what. Where-we-are and blocking stay supporting; done recap stays backup.

## [2026-08-30] update | workflows/qa-platform-sprint-kickoff.md
- Ask kickoff vs week two before formatting. Kickoff speaks plan, who, where, blocking; ending-sprint done recap stays as a labeled stakeholder backup. Week two uses last-7-day completions, not the full Done column.

## [2026-08-30] update | workflows/qa-platform-story-pointing.md
- Renamed from `qa-story-pointing.md` so QA Platform runbooks share the `qa-platform-` prefix. Inbound links updated.

## [2026-08-30] create | workflows/qa-platform-sprint-kickoff.md
- Mission-owner Monday packet: Confluence working notes as quarter backbone, Asana umbrellas as the tree, Sprint board membership as the speakable slice. Ending-sprint recap plus starting-sprint plan with carry-over and assignees. Re-confirm umbrellas when the quarter page changes.

## [2026-08-30] update | workflows/qa-platform-pre-grooming.md
- Related: sprint kickoff is a different board and meeting than Planning Needs Triage.

## [2026-08-28] update | tooling/ruff-enable-excluded-tree.md
- Grandfather import-order (`I001`) and unused-import (`F401`) with `--add-noqa`; do not `--fix` them.

## [2026-08-28] update | tooling/ruff-enable-excluded-tree.md
- After format lands, keep the tree in `extend-exclude` when a dedicated job is the sole gate (do not fold it into `ruff check .`).

## [2026-08-28] update | tooling/ruff-enable-excluded-tree.md
- Nested checkout without a parent `__init__.py`: use `namespace-packages`, not a per-file `INP001` ignore.

## [2026-08-28] update | tooling/ruff-enable-excluded-tree.md
- Do not use a package-wide `per-file-ignores` peel-off list: it disables rules for new code. Auto-fix safe hits, then `--add-noqa` remaining lines. Format can move a violation off the noqa line.

## [2026-08-28] update | tooling/ruff-enable-excluded-tree.md
- Format-only PRs stay config-free; drop the path from `extend-exclude` on the lint PR after format lands.

## [2026-08-28] update | tooling/browserstack-test-management-api.md
- Resolve `test-plans/<web#>` via `urls.self` → `TP-####`; plan/run `?id=` does not filter.
- Credential load: username may lack `export`; probe username×key pairs — exported access key can be stale.
- MCP Test Management 401: fall back to REST with the probed pair.

## [2026-08-28] update | preferences/global.md
- Commits/pushes require an explicit ask; implementing files is not permission to commit.

## [2026-08-28] create | tooling/ruff-enable-excluded-tree.md
- Keep an excluded tree in `extend-exclude` until format is applied; `[format] exclude` does not protect `ruff format --check .`.
- Parse errors need a file exclude; `INP001` can appear only on a repo-root check.

## [2026-08-27] update | workflows/mobile-test-impact-from-pr-or-branch.md
- `@ios` is a Behave filter, not a runtime lock; Android is the default platform and there is
  no `@android` tag. Platform-specific locator diffs still need a run on that platform.

## [2026-08-27] update | workflows/mobile-test-impact-from-pr-or-branch.md
- Platform-specific locator diffs are only validated on that platform. Caller `@ios`/`@android`
  tags are CI filters, not blast-radius proof; prefer steps that always hit the locator over
  find/tap search fallbacks.

## [2026-08-27] update | tooling/android-rn-native-crash-repro.md
- Don't keep activities is `settings put global always_finish_activities 1`; Samsung UI path is
  Developer options → Apps → Don't keep activities.
- Separate repro amplifiers from user repro steps: a dev-options toggle makes a real lifecycle
  transition deterministic but must never appear in a ticket's Steps to Reproduce, and frequency
  claims come from event/user counts, not from how easily the amplified setup crashes.
- `am kill` mimics the low-memory killer (what real users hit); `force-stop` is closer to a manual
  force-stop and also sets the stopped flag.

## [2026-08-27] create | tooling/android-rn-native-crash-repro.md
- Fabric/native teardown races need the preconditions forced ("Don't keep activities", process kill)
  and a device class that matches the reporter; a warm flagship session surviving the steps is a weak
  negative, not evidence.
- Filtering logcat on the C++ symbol taken from a Sentry native frame never matches; native SIGSEGV
  output lands under the `libc` / `DEBUG` tags.

## [2026-08-27] update | workflows/asana-bug-report.md
- 1 fps frames locate the window but cannot prove sub-second ordering; resample just that window at
  4 fps with `-ss/-t` and count frames between events.

## [2026-08-27] update | tooling/sentry-mobile-rn-upgrade-filter.md
- After store ship, `firstRelease:` the first store string + `environment:release` lists
  issues new to that build. That is not the same as “caused by the upgrade.”

## [2026-08-27] update | tooling/instawork-automation-api.md
- List views whose docstring claims creation-date ordering can have no ordering at all; paged sweeps
  over them cannot prove absence. Verify `filterset_fields` and model `Meta` first.
- Name-based filters are useless against factory-seeded rows; locate leftovers by probing the hash IDs
  a seeding loop produced.
- Worker assignment rows are independent of the shift's worker FK, so deleting an assignment does not
  unbook a shift.

## [2026-08-27] create | repos/hyperview-footer-safe-area.md
- Recipe for localizing a "CTA under the Android system nav bar" report without a device: grep the
  on-screen copy through templates plus the locale catalogs, follow the screen's footer include, and
  check for the missing bottom `safe_area` in the XML and HXPY variants.
- Diffing against the working sibling footer in the same flow both proves the cause and gives the
  fix shape; copy-pasted success/failure footer variants share the defect.

## [2026-08-27] update | workflows/asana-bug-report.md
- Device-console crops are only stable while the console layout holds still; panel hovers and
  device-info flyouts shift the device rectangle, so verify the crop at both ends of the trim window.
- Text covered by a translucent overlay is destroyed by the screen-share encode; recover the exact
  label from the source template instead of upscaling or contrast tricks.

## [2026-08-27] update | SCHEMA.md
- Clarified that the wiki is the workspace-root one; repo-local `wiki/` directories are unrelated and never a capture target.

## [2026-08-27] create | workflows/qa-platform-pre-grooming.md
- Pre-grooming prep for the QA Platform Planning board: confirm the quarter's owned milestones as the root set, recurse into delegated descendants, overlay the last 1–2 weeks of meeting decisions, and propose keep/defer/split/drop plus sequencing in chat.
- Ownership is a union of creator, assignee, and umbrella parent chain; a single assignee query drops most of the tree.

## [2026-08-26] create | repos/instawork-mobile-e2e-shared-pythonpath.md
- Local Docker compose PYTHONPATH is instatest-only; CI prepends e2e/mobile. `shared` imports in modules loaded before environment.py's sys.path insert fail only on the VSCode docker debug path.

## [2026-08-26] update | workflows/mobile-android-login-local.md
- Added gotcha for ModuleNotFoundError: No module named 'shared' on the local docker debug launch.

## [2026-08-26] update | tooling/sentry-mobile-rn-upgrade-filter.md
- Logcat `ReactNoCrashSoftException` on `deleteView` is the non-fatal sibling of the `updateOverflowInset` `RetryableMountingLayerException` crash.

## [2026-08-26] update | repos/mobile-abilities-verify-now-flow.md
- Repeatable Verify Now loop: mint fresh ABILITIES GigRequirements via POST /gig-requirements instead of hunting unanswered stock ones; shift-groups support DELETE for cleanup.

## [2026-08-26] update | repos/mobile-abilities-verify-now-flow.md
- After Verify Now completes, v2 cannot reset qualification/log; re-seed a new gig with a different unanswered ability.

## [2026-08-26] update | tooling/sentry-mobile-rn-upgrade-filter.md
- Native Android crashes often have no `react_native_version` on the event; Discover and issue-detail payloads omit that context. Use the app git tag.

## [2026-08-26] update | tooling/sentry-mobile-rn-upgrade-filter.md
- Prove store RN from the app git tag, not a stale local checkout.
- Android native `sdk.version` can be 8.50.1 while the JS package is 8.21.0; store 8.x is expected after the RN 0.86 ship.
- `RetryableMountingLayerException` in `updateOverflowInset` is still present on 0.86.2; the pick is in 0.86.3.

## [2026-08-26] update | workflows/mobile-test-impact-from-pr-or-branch.md
- Added a mandatory confirmation gate: after generating the run plan and Slack commands, explicitly ask for approval before triggering any run.

## [2026-08-26] update | workflows/mobile-test-impact-from-pr-or-branch.md
- Simplified Slack command guidance to production usage only by removing `/run_test_dev` direction.

## [2026-08-26] update | workflows/mobile-test-impact-from-pr-or-branch.md
- Added Slack `/run_test` command generation guidance to the workflow: production vs dev command
  selection, allowed `mobile-tests-*` parameter family, quoting rules, common command templates,
  and required "Slack commands" output section.

## [2026-08-26] update | index.md
- Registered a new workflow page for PR/branch-based mobile test impact planning and bumped index
  date/page count.

## [2026-08-26] create | workflows/mobile-test-impact-from-pr-or-branch.md
- Added a workflow that takes a PR or branch diff, classifies impacted mobile surfaces, selects
  applicant/business/both scope, maps risk to targeted vs smoke vs full regression depth, and
  returns an explicit run plan with estimated counts.

## [2026-08-26] update | workflows/self-review-workflow.md
- Expanded the workflow to include the `/personal-code-review` process content: babysit + review
  passes, selector/page-object/native-picker/Instatest checks, dead-code and constants checks, PR
  description structure, and a full review checklist with the required findings table output.

## [2026-08-25] update | repos/mobile-step-definition-conventions.md
- Do not use hasattr before getattr on @element descriptors: hasattr swallows
  AttributeError and WebDriverException, so a typed except after it never runs.

## [2026-08-25] update | workflows/self-review-workflow.md
- Exception-narrowing reviews must trace helper suppression and nullable returns; guard return values
  and fix deterministic errors at their source instead of widening catches.

## [2026-08-25] update | preferences/global.md
- For driver lookup/stale/missing-element misses, catch WebDriverException rather
  than listing ElementLookupException, NoSuchElementException, or StaleElementReferenceException.

## [2026-08-25] update | preferences/global.md
- Personal code review: keep existing try/except around optional UI; narrow the
  exception type instead of deleting the block.

## [2026-08-25] update | preferences/global.md, workflows/self-review-workflow.md
- Personal code review holds a best practices mindset: passing CI or absence of bugs is not enough.

## [2026-08-25] update | tooling/asana-api-cloud-agents.md
- Task descriptions are a single field: writing plain `notes` overwrites the rendered body, so
  headers, tables, and code blocks flatten. Edit formatted descriptions via `html_notes` only.
- No revision history to recover from: `stories` record that the description changed, not its old
  body. Rebuild structure from a sibling task in the same family, then re-apply as `html_notes`.
- `html_notes` XML rules: single `<body>` root, allowed tags only, `<br/>` in table cells is
  rejected (use a newline); `<a data-asana-gid="..."/>` is enough for a task mention.

## [2026-08-25] update | tooling/sentry-mobile-rn-upgrade-filter.md
- Do not OR `sdk.version:1.6.1` into the RN-upgrade filter: that is an old Sentry SDK on
  ancient App Store builds, not `@sentry/react-native` 8.21.0.

## [2026-08-21] update | repos/mobile-step-definition-conventions.md
- Added how to pick regression coverage after moving a shared step: three independent
  failure modes (matcher/import dry-run, Gherkin-using features including optional
  captures, Python-wrapper features), plus iOS for platform-branched helpers.

## [2026-08-21] update | preferences/global.md
- Added: never silence a linter with an inline suppression comment; fix the underlying cause
  even when it means a wider refactor, and read a needed suppression as evidence of a real
  structural problem.

## [2026-08-21] update | repos/mobile-step-definition-conventions.md
- Tightened the mid-file-import gotcha: a mid-file import and any suppression protecting it
  from being hoisted are symptoms, so attack the cycle before documenting the workaround.
- Added the cycle-removal method: enumerate what each side uses from the other; cycles
  usually hinge on one generic symbol mis-homed in a feature module (widespread aliasing at
  import sites is the tell). Relocate it to the shared module, rename around sibling name
  collisions, and verify with an AST intra-package import-graph cycle check rather than a
  runtime import.

## [2026-08-21] update | repos/mobile-step-definition-conventions.md
- Verified against the pinned fork (instawork/behave-parallel, 1.2.7.dev0), not PyPI behave:
  matcher is a process-global bound at `@step` decoration time; reset happens only between
  top-level step files, not around imports; default comes from environment.py (cfparse), so
  a missing call fails silently. Invariant: set the matcher after all imports, before the
  first `@step`. Mid-file imports forced by cycles need their own restore. `register_type`
  is matcher-sensitive too. Noted the read-the-fork trap for future matcher questions.
- Added the parse->regex conversion check: `"re"` is SimplifiedRegexMatcher (`^...$`), so
  replay real feature lines against the anchored pattern; non-matching lines are usually
  owned by a longer step definition elsewhere.

## [2026-08-20] update | tooling/circleci-api-cloud-agents.md, tooling/circleci-api-job-logs.md
- Local zprofile token is `CIRCLECI_TOKEN`, not `CIRCLECI_API_TOKEN`. Grep both names, then
  send it as `Circle-Token`.

## [2026-08-20] update | workflows/asana-bug-report.md
- Transcription needs no local install: `uvx --from openai-whisper whisper` on a 16k mono wav.
- Recordings of remote-device web consoles: crop stills and trimmed clips to the device screen
  rectangle and upscale, so the ticket shows product UI instead of browser chrome and log panes.
- Pixel-level layout claims (overlap, clipping, spacing) need a tight 3–6x zoom before deciding.

## [2026-08-19] update | workflows/self-review-workflow.md
- Do not treat debugger-session Behave step timings as runtime cost when reviewing waits.

## [2026-08-19] update | tooling/sentry-mobile-rn-upgrade-filter.md
- Resolve CI short SHAs with GitHub commit + pulls APIs, not only local `git fetch origin`.
  `compare/branch...sha` with ahead>0 and behind=0 means soup on top of the branch, not in it.
- Sentry `mechanism` is not the `handled` flag: `onunhandledrejection` can still be `handled: yes`.
  Read per-issue tag distributions (and `error.handled` in Discover) instead of the latest event.
- Transitive native modules are never autolinked; they stay dormant until some package requires
  their JS entry point.

## [2026-08-19] update | preferences/documentation.md
- Fast-approved Confluence decision-page pattern: answer in one sentence, ticket link, how
  numbers were measured, full inventory tables, split meaning vs action, short delivery list,
  warn only on the exception, no PR/CI-YAML framing.

## [2026-08-19] update | workflows/qa-story-pointing.md
- Default: most tasks are agent-executed; point with a startup mindset and keep scores low.
  Score human decision, review, and residual risk, not file count or mechanical volume.
  Prefer 1–3; 5 is rare; 8+ still means split.

## [2026-08-19] update | workflows/task-definition-readiness.md
- Estimate section now points at qa-story-pointing for the agentic/startup default.

## [2026-08-19] update | preferences/global.md
- Pointing: Always assume agent-executed + startup mindset; keep scores low. Replaced the
  earlier Do/instead-of wording.

## [2026-08-17] update | workflows/asana-bug-report.md
- Added Backend SHA from health_check: curl each env's `/internal/health_check` and take `sha`.
  Table covers Staging (`admin.staging2`), QA, Demo, Production (`admin`/`app`), Local. Agents
  should fetch the SHA when the env is known instead of leaving Unknown or asking the user.
- Also noted: no whitespace between `</h2>` and section body (Asana renders a blank line).

## [2026-08-17] update | workflows/asana-bug-report.md
- Bug bodies must carry exactly the template sections, no more and no less. Removed the earlier
  "optional iOS vs Android comparison table" advice: extra sections/tables get rejected on review
  even when correct. Platform differences go in Description / Actual Result prose plus the OS and
  App version(s) Details rows.

## [2026-08-16] update | preferences/global.md
- Record that task-description sections must have an empty line between them.

## [2026-08-16] update | preferences/global.md
- Record that task-description section headers must be bold.

## [2026-08-16] update | preferences/global.md
- Record startup-oriented sizing: for agent-executed work, points represent human decision, review,
  and residual risk rather than bulk mechanical edits.

## [2026-08-16] update | workflows/task-definition-readiness.md
- Added an Estimate section rule: include story points and a one-sentence rationale.

## [2026-08-16] update | workflows/task-definition-readiness.md
- Require reading target code, configuration, dependencies, command runner, and CI before drafting
  an implementation task.

## [2026-08-16] update | workflows/task-definition-readiness.md
- Clarified that a task with a `Do` section should not repeat it as `Done when` unless a distinct
  release gate remains.

## [2026-08-16] create | workflows/task-definition-readiness.md
- Added a pre-draft evidence check and a post-draft readiness check for implementation tasks.
- Designated ordered `Do` actions as the single source of truth to avoid duplicated completion criteria.

## [2026-08-16] update | index.md
- Registered `workflows/task-definition-readiness.md`; page count 39 → 40.

## [2026-08-14] update | tooling/sentry-mobile-rn-upgrade-filter.md
- Confirmed react_native_version is context-only (not searchable). Prefer
  `environment:beta release:*-ci-*` / SHA-specific release wildcards; optional future tag.

## [2026-08-14] create | tooling/sentry-mobile-rn-upgrade-filter.md
- Recipe for filtering applicant/business Sentry during RN upgrade soup merges: prefer React
  Native Version over release/appVersion+buildNumber; scope automation/beta; verify tag vs
  context searchability; note MCP needs auth token.

## [2026-08-14] update | index.md
- Registered `tooling/sentry-mobile-rn-upgrade-filter.md`; page count 38 → 39.

## [2026-08-14] update | workflows/asana-bug-report.md
- Reviewer feedback: for toast/overlay bugs, state interactable vs stuck and what clears the UI.
  Attach device logs as `.txt` only (no console screenshots; they are not agent-searchable and
  steal the board preview).

## [2026-08-14] update | workflows/asana-bug-report.md
- Added video analysis pipeline (English): 1 fps frame slice, local Whisper transcript with
  timestamps, reason over transcript then open only relevant stills, one ticket per bug, always
  attach a trimmed clip (not the full long recording). Silent-recording fallback noted.

## [2026-08-14] update | workflows/asana-bug-report.md
- Rewrote into an end-to-end checklist from a real filing: evidence collection (env/SHA/app
  version/recording), resolve+instantiate live template, Investigate section, custom fields from
  live task, upload then embed/link attachments, template-exact html_notes (strong headings,
  Details/Screenshots tables, no p/br), recording frame sampling, and device-log caution.

## [2026-08-14] update | workflows/asana-bug-report.md
- Added attachment-in-body recipe: embed inline images with `<img data-asana-gid=...>` (works in
  table cells; Asana expands to src+thumbnail) and link non-renderable files (video) via
  get_asset?asset_id=<gid>. List attachments to get GIDs first.

## [2026-08-14] update | workflows/asana-bug-report.md
- Corrected html_notes gotcha: `<table>` IS supported via REST (template uses it for Details and
  Screenshots); only `<p>` is rejected. Added rule to match template section names/order and
  strong-wrapped headings exactly. MCP allowed-element list omits table but REST accepts it.

## [2026-08-14] update | workflows/asana-bug-report.md
- Added attachment REST recipe, frame-sampling note for recordings, and html_notes gotcha.

## [2026-08-14] update | workflows/asana-bug-report.md
- Hard rules: always Bugs board; always resolve/fetch New Bug Report via API each filing (no
  cached template GID/body). Reframed fill steps around the live instantiated task.

## [2026-08-14] update | preferences/global.md
- Added: when filing product bugs in Asana, use Bugs board and resolve live template via API each
  time (never reuse cached template body or GID).

## [2026-08-14] create | workflows/asana-bug-report.md
- Started bug-filing runbook: Bugs project + New Bug Report template IDs, Investigate default
  section, REST instantiate (MCP cannot), template body checklist. Marked living draft for
  evidence-collection recipes.

## [2026-08-14] update | index.md
- Registered `workflows/asana-bug-report.md`; page count 37 → 38.

## [2026-08-14] update | tooling/asana-api-cloud-agents.md
- Corrected PAT env var to `ASANA_ACCESS_TOKEN` (not `ASANA_API_TOKEN`). Added recipe to list
  project task templates and instantiate from a template; noted tasks do not carry source
  template GID so match by project template name.

## [2026-08-13] update | index.md
- Added `workflows/self-review-workflow.md` to the workflow catalog and bumped the page count/date.

## [2026-08-13] create | workflows/self-review-workflow.md
- Added a self-review runbook that requires running `/personal-code-review` and reporting fix items
  in a numbered evidence-backed table.

## [2026-08-13] update | tooling/mobile-rn-upgrade-locator-defect-signature.md
- Gotcha: a blank content area under correctly rendered chrome (title bar, inputs, carried-over
  state) exonerates navigation and the server round trip; check for a missing empty-state slot in
  the server-driven list before blaming the client, and treat hardcoded seed/default content as
  zero evidence that the backing search index has data in that environment.

## [2026-08-13] update | workflows/mobile-ios-localhost-automation-build.md
- Gotcha: authenticated deep links carry the harness backend URL to the app, and a container-only
  host alias is unreachable and ATS-blocked from the iOS simulator while Android is insulated by
  reverse port forwarding plus warm intent delivery. Added the recognition signature (error view,
  no backend log entry, no socket) and the three grounding checks (process launch args, persisted
  override, backend access log), plus the leaked-Appium-session and duplicate-simulator traps.

## [2026-08-12] update | repos/mobile-step-definition-conventions.md
- Gotcha: Android Hyperview buttons can be visible but `clickable=false`; wait on native
  clickable (not only Selenium enabled), put the helper in Instatest, prefer `.click()` over
  coordinate taps, and document non-default timeouts.

## [2026-08-11] create | repos/hyperview-local-mobile-sync.md
- Captured the local Hyperview source watcher command for testing TypeScript changes in the mobile
  app without publishing the package or reinstalling dependencies.

## [2026-08-11] create | repos/mds-storybook-form-field-ids.md
- Captured that MDS form_field stories reuse Django field name `field`, so every instance gets
  `id_field`; unique, representative shell ids come from explicit widget `attrs["id"]`, not from
  renaming the field (POST fixtures expect `field`).

## [2026-08-11] update | tooling/appium-native-date-picker-locators.md
- Clarified ownership: shared native picker selection belongs in the Instatest helper; screen
  objects expose only app-owned trigger IDs and should not duplicate OS picker locators or steps.

## [2026-08-11] update | preferences/global.md
- Recorded the preference to avoid XPath selectors in mobile automation, using stable
  accessibility IDs or OS-native selectors when a system control cannot expose one.

## [2026-08-11] update | tooling/appium-native-date-picker-locators.md
- Gotcha: the server-driven framework shares one bottom-sheet modal across its date and list
  picker fields on iOS, with unidentified confirm/cancel buttons whose labels are per-element
  attributes, so name/text lookups are ambiguous once two picker kinds coexist. Also recorded the
  accepted resolution shape: document the exposure beside the locator code when no such screen
  exists yet, instead of building scoping machinery for an impossible case.

## [2026-08-10] update | tooling/github-pr-gh-cli-cloud-agents.md
- Added the tree/blob comparison recipe for proving a subtree-style import contains every tracked
  source file at its declared revision, plus the source-HEAD freshness check.

## [2026-08-10] update | tooling/appium-native-date-picker-locators.md
- Corrected an earlier same-day entry that was wrong: AOSP names the spinner wheels day/month/year,
  but live page source shows the NumberPicker nodes with no resource id, even though sibling
  internal ids surface. Lesson: framework layout source is not evidence an id reaches the
  accessibility tree — dump page source first. Settled strategy is container-scoped enumeration
  plus value-shape classification, symmetric across Android and iOS.

## [2026-08-10] update | repos/mobile-step-definition-conventions.md
- Gotcha: new Hyperview/template automation ids are not on current QA until deploy; do not
  keep text/XPath fallbacks for pre-merge BrowserStack — validate id-only after QA serves
  the template.

## [2026-08-09] update | repos/mobile-native-datepicker-automation-ids.md
- Cross-repo check: hyperview date-field wrappers also do not pass IDs into DateTimePicker;
  createTestProps only tags the field shell. Confirmed same OS-locator strategy in instawork e2e.

## [2026-08-09] create | repos/mobile-native-datepicker-automation-ids.md
- Captured that native datetimepicker spinner wheels cannot get stable per-wheel app IDs on both
  Android and iOS at the app layer alone; container `testID` only; OS locators + index is the
  practical short-term path.

## [2026-08-10] update | tooling/appium-native-date-picker-locators.md
- Captured the accessibility tradeoff behind "just add ids to the wheels": the attribute that
  makes a native wheel addressable is the one the screen reader speaks, and the widget can
  overwrite it mid-interaction; the id-resource alternative instead risks saved-state
  restoration.

## [2026-08-09] update | tooling/appium-native-date-picker-locators.md
- Corrected the blanket "per-wheel ids are impossible" claim: they are unreachable only while
  the system picker renders the wheels, and swapping in an app-owned picker component is a
  real (product-level) path. Added the recognition pattern for finding that escape hatch: an
  existing screen using a JS picker library, plus a one-line re-export shim as the swap seam.

## [2026-08-09] create | tooling/appium-native-date-picker-locators.md
- Captured why OS-native date picker wheels cannot take app-owned per-wheel ids on either
  platform, the value-based wheel classification pattern, and the trap that a library `testID`
  advertised for automation may be a Detox view tag invisible to UiAutomator2.

## [2026-08-05] update | tooling/browserstack-test-management-api.md
- Added a BrowserStack project identifier quick-lookup section and documented the tested
  test-plan clone workflow (new plan + linked run + copy run test cases).
- Captured a compatibility fallback: if `update-test-cases` returns 404, use
  `PATCH /test-runs/{id}/update` with `test_run.test_cases`.

## [2026-08-03] update | workflows/mobile-browserstack-qa-docker.md
- Documented that direct BrowserStack `docker compose run` commands must mount `<repo>/.git` to
  `/workspace/.git` so step imports that call `git rev-parse --show-toplevel` do not fail.

## [2026-08-03] update | repos/mobile-step-definition-conventions.md
- Documented API-based feature-view setup for scenarios that depend on a viewed state.

## [2026-08-03] update | preferences/global.md
- Recorded the preference to keep work in the primary agent unless subagent delegation is explicitly requested.

## [2026-08-03] update | workflows/mobile-browserstack-qa-docker.md
- Added the local-backend readiness check required after recreating the backend for a BrowserStack
  localhost run.

## [2026-08-03] update | repos/mobile-step-definition-conventions.md
- Captured PR review guidance: do not extract module-level `_*` helpers used from a
  single `@step`; inline at the call site unless reused or the step body is genuinely
  hard to scan.

## [2026-07-31] update | repos/mobile-step-definition-conventions.md
- Captured the mobile assertion rule: scope with stable accessibility IDs (or partial IDs
  for dynamic segments), not XPath; add missing IDs in the backend-driven UI first.

## [2026-07-31] update | workflows/mobile-browserstack-qa-docker.md
- Captured that Android interactive BrowserStack debugging requires the scenario tag `@set_capability::interactiveDebugging=true` (commonly added temporarily to `@wip` scenarios).

## [2026-07-30] update | workflows/mobile-browserstack-qa-docker.md
- Clarified that BrowserStack runs must use the existing `BUILD_ID` from `e2e/mobile/.env`; for parallel runs, all launches reuse that same value.

## [2026-07-30] update | workflows/mobile-browserstack-qa-docker.md
- Documented parallel BrowserStack run grouping: reuse one BUILD_ID across all parallel runs and vary compose project names.

## [2026-07-29] update | workflows/mobile-ios-localhost-automation-build.md
- Added the missing local debug config export recovery for `ConfigLocal.get is not a function`.

## [2026-07-29] update | workflows/mobile-ios-localhost-automation-build.md
- Documented that the Automation configuration embeds JavaScript and does not connect to Metro, plus
  the identity and environment tradeoff when switching to Debug.

## [2026-07-28] update | WOKRSPACE-CATALOG.md
- Added `hyperview` to the routing table and created a dedicated repo entry with purpose, scope, and
  cross-repo handoff to `mobile`.

## [2026-07-28] update | workflows/mobile-ios-localhost-automation-build.md
- Added the exact Xcode setup that worked for local backend runs: `applicant_automation` scheme,
  simulator selection, `-instaworkUrl http://localhost:8080`, plus Xcode clean build folder steps.

## [2026-07-28] update | workflows/mobile-ios-localhost-automation-build.md
- Added a gotcha for `PhaseScriptExecution Bundle React Native code and images`: inspect the first
  underlying Metro/Babel `TransformError` in the full log and fix source parse errors before
  continuing with pod/linker troubleshooting.

## [2026-07-28] update | index.md
- Added the new iOS localhost automation build workflow to the index and refreshed page count/date.

## [2026-07-28] create | workflows/mobile-ios-localhost-automation-build.md
- Captured the end-to-end local iOS automation rebuild flow: pod install location, simulator build,
  artifact copy into `instawork/e2e/mobile/.../bin`, Appium 2 + xcuitest compatibility, and the
  `CpHeader RCTAppDependencyProvider.h` recovery pattern.

## [2026-07-21] update | preferences/global.md
- Recorded the preference to show full verified file paths rather than ellipsis-truncated paths.

## [2026-07-21] create | workflows/qa-story-pointing.md
- Captured Instawork's QA story-point matrix (Fibonacci 1-13, time/effort/complexity/risk per
  point) from the Confluence "QA Story Pointing" page, for future task sizing.

## [2026-07-21] update | workflows/rice-prioritization.md
- Clarified that internal RICE analyses must define one consistent opportunity unit for Reach.

## [2026-07-21] update | workflows/rice-prioritization.md
- Added standard impact and confidence scales, compact decision output, and non-comparable work handling.

## [2026-07-21] create | workflows/rice-prioritization.md
- Added an evidence-backed RICE scoring workflow for prioritizing initiatives.

## [2026-07-16] update | repos/mobile-step-definition-conventions.md
- Documented behave.ini `exclude_re` false positive: unanchored `deep_*` excludes feature files
  whose names contain `deeplink`, yielding 0 selected scenarios with exit 0.

## [2026-07-16] update | tooling/appium-mcp-browserstack-device-session.md
- Captured iOS BrowserStack XCUITest lifecycle limits: `deep_link`, `clear`, and `background` are
  unsupported via Appium MCP; use the Safari URL-bar deeplink flow (TabBarItemTitle → URL → Go) and
  terminate the app between deeplinks when background is unavailable.

## [2026-07-16] update | preferences/global.md
- Recorded the preference to wait for explicit authorization before committing, pushing, merging, or changing a pull request's state.

## [2026-07-13] update | repos/mobile-step-definition-conventions.md
- Captured a reusable mobile locator rule: Android OEM camera/gallery UIs can change control labels
  and open in landscape, so capture and confirm controls with locator unions instead of single-label
  assumptions or portrait-only geometry.

## [2026-07-07] update | repos/mobile-step-definition-conventions.md
- Captured a reusable mobile-data-setup rule: shared shift-group creation steps should accept an
  explicit worker type and set `w2_employees_only` intentionally, instead of relying on backend defaults.

## [2026-07-06] update | tooling/appium-mcp-browserstack-device-session.md
- Captured a mobile payment-row locator gotcha: live payment-list row ids may be opaque tokens
  rather than numeric invoice ids from generated-data manifests, so locate by visible row content and
  validate the target transaction inside Payment Details.

## [2026-07-06] update | tooling/instawork-automation-api.md
- Captured a v2 celery-runner invoice-payment gotcha: serializer-echoed invoice identifiers may not
  be task-ready primary keys, so resolve the invoice inside the backend runtime from the transaction
  or shift before calling payment tasks.

## [2026-07-06] update | tooling/instawork-automation-api.md
- Captured a Stage-2 mobile automation data-gate gotcha: legacy automation webhooks may exist for
  a setup action, but if the orchestrator requires `/automation/api/v2/swagger` coverage, a
  test-scoped missing v2 endpoint remains a blocker unless the run explicitly allows non-v2 helpers.

## [2026-07-06] update | workflows/mobile-automation-pipeline.md
- Captured mobile automation feature-file placement guidance: target roughly 5 scenarios per
  feature file, but keep scenarios that must run sequentially together because Instatest runs
  feature files in parallel in CI; shared mutable setup such as Constance flag updates is a
  sequential-placement signal.

## [2026-07-04] create | tooling/appium-mcp-browserstack-device-session.md
- Captured Appium MCP + BrowserStack device-session gotchas from a live run: `appium_app_lifecycle`
  `deep_link` requires the package `id` even though the schema implies it's optional; a fresh-install
  admin-login deep link lands on a one-time welcome/congrats screen before the main app; the
  applicant Profile tab's "My skills" section needs ~2 scrolls past Profile strength/Positions
  before it renders; `scroll_to_element` can falsely report an element "already visible"; skill/
  category rows expose full-label accessibility ids (name + count suffix).

## [2026-07-03] update | tooling/instawork-automation-api.md
- Corrected an over-eager Stage-2 blocker: API v2 has no Skill/PositionSkill endpoints (verified,
  zero matches in the full swagger dump), but that's expected and not a data-fit failure — catalog
  data like Skill/PositionSkill is pre-existing product config, not per-test data Stage 2 must
  create. Recognition pattern: before treating a missing automation-API endpoint as a blocker, check
  whether the entity is test-scoped data (blocker) or global/product config that already exists in
  the target environment (not a blocker — resolve live during device session instead).

## [2026-07-01] update | preferences/documentation.md
- Recorded the prose-writing preference: use `/writing-guidelines` before drafting or polishing
  emails, Slack replies, Markdown docs, PR descriptions, and similar prose.

## [2026-07-01] update | tooling/slack-mcp-bulk-export.md
- Consolidated overlapping disk-writing guidance into one preferred path: private per-window shards,
  `jq` for JSONL escaping, Python only with `-S` when unavoidable, and namespaced staging for
  parallel collectors.

## [2026-07-01] update | SCHEMA.md
- Added the reconciliation rule for wiki updates: read the whole page, merge overlapping guidance
  instead of appending competing bullets, and centralize writes from parallel workers through one
  parent agent.

## [2026-07-01] update | preferences/global.md
- Replaced unconditional full wiki preflight with tiered preflight: full lookup for substantive work,
  fast path for tiny one-step tasks, and escalation to full lookup when a task broadens or gets
  blocked.

## [2026-07-01] update | preferences/global.md
- Recorded the Drive sharing preference: never grant anyone-with-link access; restrict sharing to
  people within the organization when sharing is required.

## [2026-07-01] update | tooling/gws-slides-cli.md
- Corrected the Slides image workflow to avoid public Drive sharing. Use organization-scoped
  sharing only, and stop if an API path requires a public URL.
- Added the `createImage` gotcha: organization-readable Drive URLs are rejected, so public sharing
  must not be used as the workaround.

## [2026-07-01] update | tooling/gws-slides-cli.md
- Added the image path for Slides: upload local generated assets to Drive from the asset folder,
  make only those generated image files link-readable, and insert them with `createImage` using the
  Drive download URL.

## [2026-07-01] create | tooling/gws-slides-cli.md
- Captured the working Google Slides CLI path: inspect schemas, create a presentation, populate
  slides with `batchUpdate`, fetch notes page body IDs for speaker notes, and use `color` for
  shape fills versus `opaqueColor` for text foreground colors.

## [2026-07-01] update | preferences/documentation.md
- Refined presentation deck preference: keep visible slide text sparse, mix visual types, describe
  the intended visual for each slide, avoid overusing diagrams, and move detail into speaker notes
  when useful.

## [2026-06-30] update | tooling/cursor-agent-worker-launchd.md
- Added per-repo `--worker-dir` config guidance (discover via nested `.git` search, keep root too)
  and a bootout/bootstrap race gotcha: running both back-to-back in one command can fail with
  "Bootstrap failed: 5: Input/output error" because bootstrap fires before the label finishes
  unregistering; run them as separate calls with a `launchctl list` check in between.

## [2026-06-30] update | tooling/instawork-automation-api.md
- Added a fast manual mobile repro loop that skips the full instatest/behave cycle per iteration:
  create a worker via the automation API for a token, build the same authenticated deep link the
  suite uses, launch via `adb shell am start` (single-quote the deep link inside one command
  string, or the device's remote shell truncates the URL at the first unescaped `&`), and drive
  with a raw Appium HTTP session (no MCP tool required) instead of a full test run. Also flagged
  that tapping between bottom tab-bar items does not remount/re-fetch an already-mounted screen
  (React Navigation keeps tab screens mounted) — relaunch via the deep link to force a fresh
  fetch when testing server-side template changes, and watch the Django reloader's cache-clear
  log line to confirm an edit was actually picked up.

## [2026-06-30] update | workflows/mobile-android-login-local.md
- Added two emulator/Appium gotchas from a local RN-upgrade A/B repro run: (1) a cold-booted
  emulator can die silently within seconds with no crash report when host free memory is in the
  tens-of-MB range (`vm_stat`); quitting a memory-heavy app before retrying the cold boot fixed
  it. (2) background processes started with `&`/`disown` inside a single shell-tool call die
  once that call returns; use the tool's own persistent-background mechanism instead and verify
  liveness with a follow-up call.

## [2026-06-30] update | tooling/mobile-rn-upgrade-locator-defect-signature.md
- Added live `page_source` triage guidance from a confirmed local repro: on Android, RN `testID`
  surfaces as `content-desc`, essentially never as `resource-id` (apps with no native id assignment
  show only 1-2 system `resource-id`s in the whole dump). A dropped slash-namespaced id can vanish
  entirely rather than get a renamed 1:1 replacement; check whether the breakage is scoped to one
  component (e.g. just the tab navigator) by confirming other slash-namespaced ids elsewhere on the
  same screen still exist unchanged.

## [2026-06-30] update | tooling/mobile-rn-upgrade-locator-defect-signature.md
- Added two gotchas from a second forensic case: test_runs array order is not guaranteed
  chronological (sort by time.start), and a retry's screenshot can reveal a false pass on an
  earlier "screen loaded" step (too-loose screen selector) compounding the locator defect.

## [2026-06-30] update | tooling/mobile-rn-upgrade-locator-defect-signature.md
- Added the counter-signature for genuine product defects during forensic triage: a completely
  blank/content-less failure screenshot (vs. the target screen rendered minus one element) rules
  out a locator-only explanation. Also flagged that a generic "the X screen should be loaded"
  step can be a false pass if it only instantiates a page-object reference without asserting any
  element/text, so its "pass" doesn't prove the precondition for the next step was really met.

## [2026-06-30] create | tooling/mobile-rn-upgrade-locator-defect-signature.md
- Captured the forensic recognition pattern for classifying a failed mobile test as
  automation/locator defect (vs product defect) during RN upgrades: element visible and
  correctly rendered in the failure screenshot, the step looks the element up by an id/testID
  locator, sibling id-lookups on the same screen object pass in preceding steps, and the
  failure reproduces identically across every retry. Don't trust the raw exception text alone.

## [2026-06-30] update | tooling/browserstack-failure-triage.md
- Added the RN-upgrade locator breakage signature: assertion says element "is displayed" but the
  failure screenshot shows it rendered; confirm by diffing the failing locator's xpath against a
  same-screen sibling locator that still passes (same structural pattern, divergent trailing
  indices implies a view-tree shape shift, not a real product regression). Useful even when the
  framework's own classifier bucketed the failure as a product defect.

## [2026-06-30] create | workflows/mobile-ci-allure-failure-triage.md
- Captured the efficient triage path for mobile CI failures: read the job's custom
  `all_error_details.json` (pre-deduplicated to scenarios that failed on every retry attempt) and
  `categories.json` (product vs test defect hint) instead of raw allure-results; verify the hint
  against the actual failure screenshot; package one folder per failing scenario before fanning out
  to parallel subagents.

## [2026-06-30] update | repos/mobile-vendored-instatest.md
- Captured that instatest's `driver.page_source` -> `.xml` failure-capture is fully built but ships
  disabled in two places (`screen_capture.py` capture block, `environment.py` Allure-attachment
  block), with a vestigial unused env-flag gate. Re-enabled it locally and verified end to end on
  BrowserStack: it correctly dumped page source next to a failure screenshot. Also captured the
  forensic recipe for using it (search both `resource-id` and `content-desc`, since RN-on-Android
  apps can rely almost entirely on `content-desc`; don't assume a 1:1 id rename, check whether the
  old id's *role* still has any surviving candidate element; diff naming-convention scope to narrow
  the regression to one component family).

## [2026-06-30] update | WOKRSPACE-CATALOG.md
- Recorded an active migration: the mobile e2e suite (`.feature`/steps) is moving from `mobile`
  (`*/test/automation/`) into `instawork` (`e2e/mobile/<app>/`). Both repos receive commits right
  now and get periodically reconciled. CircleCI applicant/business app automation jobs currently
  execute from the `instawork` copy, not `mobile`. Confirmed via a real CircleCI job's checked-out
  commit having `e2e/mobile/applicant-app/features/` and via reconciliation commit messages
  (`chore(mobile-e2e): reconcile mobile automation drift...`) in both repos' git logs.

## [2026-06-30] create | tooling/circleci-api-artifact-download.md
- Captured the CircleCI artifact-download gotcha: the `url` field from
  `GET /v2/project/{slug}/job/{job}/artifacts` 302-redirects to a presigned S3/circle-artifacts.com
  URL. Plain `urllib.request.urlopen` forwards the `Circle-Token` header across the redirect and
  gets a silent 404; `curl` needs `-L` explicitly. A bare HEAD request (`-I`) also 404s even when
  a real GET with `-L` succeeds, so don't use HEAD to probe these URLs.

## [2026-06-30] update | repos/mobile-step-definition-conventions.md
- Added a forensic-triage gotcha for "X was not found on screen" assertion failures during RN
  upgrades: read the step def's locator strategy, then check whether the screenshot shows the
  component visually absent entirely (product/rendering defect) vs present-but-unaddressable
  (locator/testID defect), and cross-check against the screen's class hierarchy for whether the
  component was expected at all.

## [2026-06-30] create | tooling/cursor-agent-worker-launchd.md
- Captured two root causes for cursor agent worker failing silently under launchd: (1) macOS 15 TCC
  blocks background tasks from writing to ~/Documents — use ~/Library/Logs/ instead; (2) Node.js
  Keychain API fails in launchd context — embed CURSOR_ACCESS_TOKEN directly in plist env.
  Also captured: launchctl bootstrap hangs with Node.js XPC registration (background the call);
  exit code 78 with no log output is the combined symptom; label poisoning after crash loops
  requires a fresh label name to escape.

## [2026-06-29] update | workflows/mobile-browserstack-qa-docker.md
- Added Apple Silicon gotcha: browserstack-local==1.2.2 hardcodes x86_64 binary on Linux; ARM64
  container fails with Rosetta ELF error and tunnel never starts. v1.2.3+ detects aarch64 and
  downloads the arm64 binary. Fixed by bumping requirements_qa to 1.2.15 and rebuilding the image.

## [2026-06-29] create | tooling/docker-compose-env-precedence.md
- Captured docker compose env precedence gotcha: an exported-but-empty shell var beats the project
  .env in `${VAR:-default}` substitution, silently falling back to the default and masking a correct
  .env edit. Verify with `docker compose config | grep VAR`, `unset VAR`, then `up --force-recreate`
  (restart does not re-read env). Confirm via in-container `printenv`/derived setting, not the file.

## [2026-06-29] create | tooling/granola-mcp-bulk-export.md
- Captured Granola MCP bulk export: one wide list_meetings call yields the index (cap ~138/response),
  Me:/Them: speaker labels in transcripts, a hard ~1 transcript/2min server-side rate cap on
  get_meeting_transcript that concurrency cannot beat, and an encrypted local granola.db (no bypass).

## [2026-06-29] update | preferences/global.md
- Recorded cross-linking preference: PR and its tracking ticket must each open with a link to the
  other at the very top of the description.

## [2026-06-29] update | tooling/circleci-config-typed-parameters.md
- Added the alternative (preferred when you own the consumer) sentinel placement: normalize the
  numeric sentinel to empty inside the consumer at arg-parse time, so the config passes the raw value
  and sentinel handling sits next to the empty-means-unset logic. Single source of truth, no config
  re-validation.

## [2026-06-29] create | tooling/circleci-config-typed-parameters.md
- Captured the CircleCI typed-parameter empty/optional gotcha: integer and boolean params can't be
  empty (`default: ""` is invalid), so converting an optional string param to integer breaks any
  shell logic that treated empty as unset. Fix is a numeric sentinel (e.g. 0) with numeric checks in
  the config plus a boundary conversion back to empty before calling the consumer, so the consumer's
  contract stays untouched. Verify with `circleci config validate` and a shell matrix harness.

## [2026-06-29] update | repos/serverless-deploybot-run-test.md
- Corrected the `/run_test` parsing note after the naive splitter failed the existing behave-args test.
  The splitter must be escape-aware (`r'[^\s=]+="(?:\\.|[^"\\])*"|\S+'`) so values with spaces and inner
  `\"` stay one token; unescape via `shlex.split(value)[0]` in infer_data_type (a `[1:-1]` slice does not
  unescape). A plain `\S+="[^"]*"` regex shatters escaped-quote values into bogus invalid-parameter tokens.

## [2026-06-29] update | repos/serverless-deploybot-run-test.md
- Added the `/run_test` parameter parsing contract: quoted value means string, unquoted infers type.
  Quote normalization must cover the full Unicode Quotation_Mark set (smart/guillemet/CJK/low-9/
  fullwidth) via one `str.maketrans`, not a hand-picked class. Integer/bool CircleCI params must be unquoted.

## [2026-06-29] create | workflows/large-corpus-distillation-subagents.md
- Captured the map-reduce method for distilling a corpus too large for one context (persona/voice
  clone from ~47k Slack messages): ~1500-item shards (2300+ triggers worker `resource_exhausted`),
  one fresh worker note per shard, waves of ~7, fresh reducer synthesizes from notes not raw data.
  Shard by exact id (ts) so a later dataset refresh only re-distills the uncovered delta and reuses
  prior good notes. Verify notes by line count (0-line files happen) before launching the reducer.

## [2026-06-27] update | tooling/slack-mcp-bulk-export.md
- Added shared-dir collision gotcha: concurrent sibling subagents writing into the same `raw/` dir
  overwrite each other's generically-named helper/temp files mid-run and can truncate a shared output
  file. Namespace all temp/staging files with a unique prefix, accumulate privately, and cp/mv onto
  the canonical path as the final step. Noted cursors are deterministic (base64 "CURRENT_PAGE:N").

## [2026-06-27] update | tooling/slack-mcp-bulk-export.md
- Added per-subagent context sizing guidance: one dense month (~1100 msgs, ~55-60 pages) fills a
  whole subagent context, so split bulk pulls by month (not multi-month) and flush a durable
  per-slice file as you go. Noted that writing cleaned JSONL per page (vs re-emitting raw + parsing)
  roughly halves output-token cost.

## [2026-06-27] update | tooling/slack-mcp-bulk-export.md
- Added a "Persisting pages to disk reliably" section: per-page shell heredoc appends are flaky on
  the pyenv-shim shell (intermittent ~40s startup, one observed empty-truncation). Reliable pattern
  is Write-tool shard files per window then a single cat+dedup-by-ts merge. Also note skipping
  non-target/system authors (`From: (ID: U00)`) and empty-text messages.

## [2026-06-26] create | tooling/atlassian-confluence-cql-mcp.md
- Captured how to bulk-collect Confluence pages via the Atlassian MCP `searchConfluenceUsingCql`.
  Key gotcha: expanding `content.body.view` caps results at 50 per page regardless of `limit`, so
  paginate with the `cursor` param taken from the `next` link and url-decoded (`%3D`→`=`). Plain
  metadata-only search has no cap. Space key lives in `content._expandable.container`.

## [2026-06-26] create | tooling/slack-mcp-bulk-export.md
- Captured the low-context protocol for bulk-exporting Slack via the MCP after a collection subagent
  died with `resource_exhausted`. Keepers: use `from:<@id>` + `-in:<@id>` filters, force
  `response_format=concise` and `include_context=false`, paginate 20 at a time and stream each page to
  disk discarding raw responses, cap pages. Recognition: `resource_exhausted` here is context
  exhaustion not API quota. Gotcha: concise format gives no epoch ts and surfaces non-real years, so
  absolute dates are unreliable (ordering still holds). Stripped the persona-task specifics.

## [2026-06-26] update | AGENTS.md, wiki/SCHEMA.md, preferences/global.md
- Encoded two fixes after user feedback. (1) Proactive capture: AGENTS.md self-sustaining memory
  now mandates a per-reply reflex to judge and persist durable learnings without being asked;
  recorded the same as a standing preference in global.md. (2) Symptom vs learning: added a 4th
  capture-threshold criterion in SCHEMA ("general, not the incident": survives stripping point-in-
  time facts) and a matching "persist the learning, not the symptom" line in AGENTS.md.

## [2026-06-26] update | repos/serverless-deploybot-run-test.md
- Added a recognition pointer: `/run_test` showing Slackbot `operation_timeout` while the pipeline
  still starts is a `triggerautomation` cold start (handler finishes after Slack's 3s ack; async
  worker invoke already fired), not a bug. Kept it to the recognition shortcut plus a confirm-via-
  REPORT-line pointer; deliberately omitted incident numbers and a mitigation essay as symptom/
  point-in-time detail, not durable learning.

## [2026-06-26] create | tooling/aws-cli-cloudwatch-lambda-logs.md
- New tooling recipe: query Lambda CloudWatch logs via AWS CLI (SSO DeveloperAccess, account
  183605072238, us-west-2). macOS `date -u -j -f ... +%s`000 epoch-ms trick for filter-log-events
  windows; filter by unique substring then request id; diagnose cold starts from INIT_START +
  Init Duration on the REPORT line. Captured while investigating the deploybot operation_timeout.

## [2026-06-25] create | repos/mobile-abilities-verify-now-flow.md
- Documented the data configuration required to trigger the post-booking ABILITIES "Verify now"
  flow. Root cause: Bartender gig templates auto-apply the "Professional" grooming preset by
  default, which handles ABILITIES inline during booking confirmation and prevents the post-booking
  Verify Now bottom sheet. Fix: set `add_grooming_preset: false` when creating the shift group.
  Also: worker must be fresh (no prior ABILITIES answers). Verified in Run25 (all 21 steps passed).

## [2026-06-25] update | AGENTS.md, wiki/SCHEMA.md
- Clarified wiki memory boundary: wiki entries are process-oriented (recipes, shortcuts, gotchas,
  workflows, preferences), not business logic or feature behavior. Those live in the repos.
  Added explicit statement to both AGENTS.md (Memory boundaries) and SCHEMA.md (Purpose section).

## [2026-06-25] create | workflows/mobile-automation-pipeline.md
- New recipe page: automating a BrowserStack test case end-to-end. Pipeline: fetch Asana task +
  BS test case → tracing-feature-flows → test-plan-generation (1 scenario for corroboration) →
  mobile-automation-orchestrator Phase 1 Spec. Key shortcuts: spawn all 5 layer subagents in
  parallel after surface discovery; run gate reviewers in parallel; single-scenario override for
  test-plan-generation; "Cannot Automate" staleness gotcha.

## [2026-06-25] update | tooling/browserstack-test-management-api.md
- Added gotcha: "Automation Type: Cannot Automate" custom field is often a legacy assessment;
  evaluate automateability independently from the trace.

## [2026-06-25] create | tooling/browserstack-test-management-api.md
- Documented the Test Management API (distinct host from App Automate). Key gotcha: UI URLs use
  internal numeric ids, not PR-/TC- identifiers. Resolve the project web number to PR-#### via
  GET /api/v2/projects (match urls.self); pass the test-case URL number as the bare ?id= value
  (?id=TC-#### returns 0). MCP listTestCases needs project_identifier=PR-####, not the web number.
  Saved the Instawork project map (PR-1023/1025/1026/1027/1028). Verified fetching TC-6782.

## [2026-06-24] update | repos/mobile-step-definition-conventions.md
- Documented Android-as-default platform convention: no @android tag exists; @ios marks iOS-specific scenarios; platform comes from --device-platform at runtime, not tags.

## [2026-06-23] update | preferences/documentation.md
- Recorded preference: always put related PR links at the very top of the Asana task description.

## [2026-06-23] update | preferences/documentation.md
- Recorded preference: always put the Asana task link at the very top of the PR description
  when the work has a task.

## [2026-06-23] update | preferences/documentation.md
- Recorded preference: keep PR descriptions concise and skimmable so people read to the end;
  avoid long write-ups that go unread.

## [2026-06-23] update | repos/instawork-docker-network.md
- Corrected the fix: external:true is intentional (instawork_default is shared with the finch stack;
  Compose recommends external for cross-project named networks). The bug is the missing creation
  step, not the flag. Fix (PR #45756): keep external, add an idempotent `docker network create`
  step to the web-tests-localhost job + README. Verified the cross-project warning and the
  create-then-up flow locally.

## [2026-06-23] create | repos/instawork-docker-network.md
- web-tests-localhost CI failed: "network instawork_default declared as external, but could not be
  found". PR #45688 added external:true to docker-compose.yml + docker-compose.config-sync.yml;
  nothing creates the network on fresh envs (local passed only via a leftover network).

## [2026-06-23] create | tooling/github-pr-gh-cli-cloud-agents.md
- When the built-in PR tool returns [unauthenticated] or "PR URL must belong to the current
  repository" (multi-repo workspace), use the gh CLI instead. Active gh account has repo scope, so
  gh pr create/edit/comment work across repos with --repo. Verified creating mobile#8538 and
  editing/commenting test-automation#194.

## [2026-06-23] create | workflows/mobile-browserstack-qa-docker.md
- Runbook for applicant tests on BrowserStack against QA via Docker. Key gotcha: the
  --device-name/--device-platform/--device-version triple is all-or-nothing; omitting
  --device-version falls back to runtime.config test_device (Android8) and crashes before_all with
  "No device found", exiting 0 with all scenarios untested. Includes the LOCAL instatest verify
  command and the feature-branch checkout step. Verified @validate_pr_tests (2 scenarios) pass.

## [2026-06-23] update | preferences/global.md
- Recorded standing rule: on a tool/integration auth or access failure, consult the wiki for a
  documented fallback before improvising or declaring the capability unavailable. Hardened AGENTS.md
  "If blocked" guidance with the same failure-time reflex. Driven by a recurring miss where an MCP
  needsAuth error led to ad-hoc retrieval instead of the documented REST fallback.

## [2026-06-23] create | repos/mobile-vendored-instatest.md
- src/instatest is a git-ignored editable install of the test-automation framework;
  ripgrep/Grep skip it (use --no-ignore). Framework-level changes belong in test-automation,
  not the mobile suite/PR.

## [2026-06-23] update | repos/mobile-ios-autoAcceptAlerts-race.md
- Documented root cause of the iOS in-DOM-but-not-displayed scroll issue: instatest iOS scroll
  uses directional `mobile: scroll` (not element-targeted), Android uses UiScrollable.scrollIntoView.
  scrollToElement is used nowhere else; most iOS targets just land in-viewport. Cross-linked new page.

## [2026-06-22] update | repos/mobile-step-definition-conventions.md
- Added gotcha: step function names are not unique (e.g. multiple step_screen_should_have_text);
  a same-name module-level def shadows a top-of-file import, so a direct call hits the wrong
  function and fails on arg count. Import reused step functions with an alias.

## [2026-06-22] update | repos/mobile-step-definition-conventions.md
- Added convention: reuse steps by importing and calling the step function directly
  (not `context.execute_steps`), to preserve IDE go-to-definition and step-through
  debugging. Target keeps `@retry`; importing does not re-register the behave step.

## [2026-06-22] update | repos/mobile-step-definition-conventions.md
- Added platform-scope convention: branch only for ios/android with if/elif, no `else`
  / unsupported-platform branch, since nothing else is ever tested.
- Renamed page from mobile-step-definition-asserts to mobile-step-definition-conventions
  to cover the broader topic; updated index link.

## [2026-06-22] create | repos/mobile-step-definition-conventions.md
- Mobile Behave step definitions convention: signal outcomes with `assert <cond>, "<msg>"`,
  not bare `return` on success or `raise AssertionError` on failure.
- Polling pattern: set a boolean flag, `break`, then a single `assert flag, "<msg + last state>"`.
- Gotcha: when removing return-on-success, propagate the inner `for` break to the outer `while`.

## [2026-06-20] create | repos/infrastructure-sops-secrets.md
- SOPS version must be >= 3.8.1 for SSO auth (3.7.3 fails silently with SSOProviderInvalidToken).
- Use `sops set` or `sops secrets.yml` (not decrypt→modify→re-encrypt) to preserve existing ciphertext.
- After `sops set`, manually restore empty provider arrays (gcp_kms, azure_kv, etc.) — sops 3.10+ strips them.
- Use same sops version as the file's `version:` field to avoid metadata format diffs.
- Staging2 needs its own SSM param when adding to production; build-staging2 fails if missing.
- Staging2 uses individual `aws_ssm_parameter` resources, not the grouped map like production.

## [2026-06-20] create | tooling/circleci-api-job-logs.md
- v2 API is metadata-only. Actual step log output requires v1.1 API.
- v1.1: `GET /api/v1.1/project/github/{org}/{repo}/{job_number}` returns steps[] with output_url per action.
- output_url is a presigned S3 URL (no auth needed), expires in minutes — fetch promptly.
- v1.1 uses `github/` not `gh/` in the path (v2 uses `gh/`).
- Try circleci-mcp-server `get_build_failure_logs` tool FIRST before manual API calls.

## [2026-06-20] update | repos/serverless-deploybot-run-test.md
- Rewrote with final architecture: in_channel empty text, conversations.history for user message ts, SLACK_CIRCLECI_BOT_TOKEN for chat.postMessage.
- Key gotchas documented: response_url does not return ts, delete_original doesn't work for slash commands, oldest filter races with Lambda cold start, SLACK_DEPLOY_BOT_KEY is wrong identity.

## [2026-06-19] update | repos/mobile-git-worktree-docker.md
- Confirmed worktree fix (git rev-parse try/except) resolves the crash and exposes the secondary
  blocker: stale Docker image missing allure_hook_steps. Volume mount workaround confirmed working:
  add `mobile/src/instatest:/workspace/src/instatest:ro` to docker-compose_qa.yml volumes so
  PYTHONPATH picks up the newer instatest. These changes are worktree-only scaffolding and must
  NOT be committed to the PR branch.

## [2026-06-19] create | repos/mobile-android-cancel-button-uppercase.md
- Android renders Material dialog buttons as uppercase ("CANCEL", "CALL") even when the feature
  file or business logic uses mixed-case text. TextSelector uses case-sensitive UiSelector.text().
  Pattern: use `I tap on element with text "CANCEL" if exists` + `I tap on element with text
  "Cancel" if exists` so each platform matches its own button text. The existing step definition
  step_business_contact_dialog_shows_call_and_cancel already demonstrates this platform split.


- iOS autoAcceptAlerts race condition: Appium auto-dismisses native UIAlertController (phone
  call dialog) between sequential assertions. Wrong fix: @set_capability::autoAcceptAlerts=false
  blocks permission sheets (location/motion). Correct fix: single atomic page_source check
  verifies both elements in same snapshot. Also: scrollToElement fallback when iOS element is
  in DOM but not in viewport after swipe_to_selector_using_locator.

## [2026-06-19] create | repos/test-automation-parallel-exit-code.md
- Root cause of mobile CI "0 failed but exit code 1": hook_failures in behave-parallel run_model()
  accumulates across worker batch even when autoretry makes scenarios pass. Fix: JUnit XML override
  in instatest/core/execution/run.py. PR: https://github.com/Instawork/test-automation/pull/194

## [2026-06-19] create | repos/serverless-deploybot-run-test.md
- Documented threading architecture for /run-test Slack bot: chat.postMessage to capture ts,
  pass THREAD_TS to automationworker (all worker messages in thread), inject
  test-automation-slack-thread-ts into CircleCI params so allure_report_slack.py results
  also reply in the thread. Clickable pipeline link format, respond() empty-dict bug fix,
  test setup for module-level SSM mock.
- Serverless PR: https://github.com/Instawork/serverless/pull/927
- Mobile PR: https://github.com/Instawork/mobile/pull/8532

## [2026-06-19] create | repos/mobile-git-worktree-docker.md
- git worktree .git file is unresolvable inside Docker container; `git rev-parse --show-toplevel` crashes
  `environment.py` and `login_steps.py` at import. Fix: try/except with relative fallback. Secondary issue:
  stale Docker image lacks `allure_hook_steps`; workaround via extra volume mount of newer local instatest.

## [2026-06-20] create | tooling/browserstack-failure-triage.md
- Lean two-step triage brick: BS session grounding (logs/screenshots, reason is not diagnosis) then test-code grounding. Trimmed verbose debug section from API page.

## [2026-06-20] update | tooling/browserstack-app-automate-api-cloud-agents.md
- Verified failed-session debugging via REST: text/appium/device/network logs, debug screenshot URLs from text logs, session reason field, crashlogs 404 for non-crash failures.

## [2026-06-19] create | tooling/browserstack-app-automate-api-cloud-agents.md
- Documented BrowserStack App Automate REST API fallback for cloud agents: MCP 401 despite ready status, credentials in ~/.zprofile not auto-loaded, HTTP Basic auth, preflight/build/session/device curl examples.

## [2026-06-19] create | tooling/circleci-api-cloud-agents.md
- Documented CircleCI REST API fallback for cloud agents: MCP 401 despite ready status, PAT in ~/.zprofile not auto-loaded, Circle-Token header, preflight and pipeline/workflow curl examples.

## [2026-06-19] update | repos/mobile-git-worktree-docker.md
- Added PR hygiene rule: the worktree Docker sys.path fix is scaffolding only — revert before pushing the PR.

## [2026-06-19] create | instawork tracing + test plan + mobile automation — onsite contact call prompt
- Ran `tracing-feature-flows` skill on instawork repo: saved trace packet at `.cursor/skills/tracing-feature-flows/temp/onsite-contact-call-prompt.trace.md`.
- Ran `test-plan-generation` skill on instawork repo: saved test plan at `.cursor/skills/test-plan-generation/temp/e2e-test-plan-onsite-contact-call-prompt.md`. Top automation candidate: TC-001 Android business contact Call button present (regression guard for PR #40684 class bug).
- Ran `mobile-automation-orchestrator` full workflow (Stages 1–4): spec reused from prior run, Stage 2 regenerated fresh QA data (shift expired), Stage 3 evidence reused from prior handoff, Stage 4 implementation passed BrowserStack Android (session: https://app-automate.browserstack.com/sessions/758447bec5aaabbb4a6ce426e5c9eafc9d737874).
- PR created: https://github.com/Instawork/mobile/pull/8531 on branch `cursor/onsite-contact-call-prompt-a1a3`.
- Worktree: `instawork-all-repos/mobile-onsite-contact-callprompt`.

## [2026-06-19] update | AGENTS.md, wiki/SCHEMA.md, preferences/global.md
- Made `AGENTS.md` a lean bootstrap contract: required preflight first, clear memory boundaries, autonomous memory capture, and catalog update limits. Aligned `wiki/SCHEMA.md` with required preflight and recorded the preference that wiki and preferences must be consulted before task work and grown autonomously.

## [2026-06-19] update | AGENTS.md
- Restructured to put "REQUIRED: read before doing anything else" as the very first section. Wiki grounding is now unconditional (no qualifiers like "non-trivial"), listed as concrete file reads, and gated by a self-check. Removed all escape hatches.

## [2026-06-19] update | preferences/global.md
- Added preference: never use concrete tool/service names or specific examples in agent rules or AGENTS.md; keep rules general and principle-based to avoid introducing bias.

## [2026-06-19] update | AGENTS.md
- Added "Hard rules" section at top of wiki protocol: orient applies to any task involving tool use or an external service (not just "non-trivial" tasks), and the blocked-tool rule is now a top-level guardrail before the orient steps, not buried after step 5. Removed specific tool name example from the rule text per preference against bias-inducing examples.

## [2026-06-19] create | tooling/asana-api-cloud-agents.md
- Documented Asana REST API fallback for cloud agents: MCP needsAuth, PAT in ~/.zprofile not auto-loaded, token extraction one-liner, preflight and project/task curl examples, OAuth vs PAT distinction.

## [2026-06-19] create | preferences/global.md, preferences/documentation.md
- Added user preferences memory under wiki/preferences/. Seeded global.md and a documentation.md context example from existing Cursor user rules. Updated SCHEMA.md (Preferences section), index.md (Preferences section), and AGENTS.md (three-branch decision rule, orient loop reads preferences, preferences protocol).

## [2026-06-19] update | AGENTS.md
- Wired WOKRSPACE-CATALOG.md into the orient loop as routing step 1, added the catalog-vs-wiki decision rule, and added a catalog maintenance section. Catalog stays at root, not moved into wiki/.

## [2026-06-19] lint | wiki
- Verified index completeness, relative links, and page count after initial seeding. All clean.

## [2026-06-19] update | AGENTS.md
- Rewrote AGENTS.md as the lean wiki controller; runbooks now live under wiki/workflows/.

## [2026-06-19] create | tooling/instawork-automation-api.md
- Seeded the shared automation API tooling page from existing AGENTS.md pitfalls.

## [2026-06-19] create | workflows/mobile-android-login-local.md
- Migrated the Mobile Android login runbook out of AGENTS.md into the wiki.

## [2026-06-19] create | workflows/web-e2e-local.md
- Migrated the Web E2E runbook out of AGENTS.md into the wiki.

## [2026-06-19] create | SCHEMA.md
- Wrote the wiki rulebook: layout, page format, tag taxonomy, capture threshold, maintenance, lint.

## [2026-06-19] create | wiki initialized
- Created skeleton: SCHEMA.md, index.md, log.md, and area folders (repos/, tooling/, infra/, workflows/, _archive/).
