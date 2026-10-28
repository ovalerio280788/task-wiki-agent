# Wiki index

> Catalog of every wiki page, grouped by area, one line each.
> Read this first to find relevant pages and to avoid creating duplicates.
> Last updated: 2026-10-28 | Total pages: 102

## Preferences
<!-- how the user wants me to behave; read global.md every orient, context files when relevant -->
- [global](preferences/global.md) — behavioral preferences that apply everywhere. Always read at orient.
- [documentation](preferences/documentation.md) — sentence case; decision/spike pages: answer first, full tables, short delivery list.

## Workflows
<!-- multi-step runbooks -->
- [web-e2e-local](workflows/web-e2e-local.md) — run Playwright web e2e against the local backend on :8080.
- [mobile-android-login-local](workflows/mobile-android-login-local.md) — run applicant Android login via Instatest/Docker against local :8080.
- [mobile-android-resource-id-locator-fix](workflows/mobile-android-resource-id-locator-fix.md) — one applicant scenario on local Android emulator; classify resource-id migration, fix locators like instawork#47197, re-run.
- [mobile-ios-localhost-automation-build](workflows/mobile-ios-localhost-automation-build.md) — rebuild applicant iOS automation simulator app, copy artifact to instawork e2e bin, and run against localhost; also signed device beta IPA via `yarn build:ios:beta`.
- [mobile-browserstack-qa-docker](workflows/mobile-browserstack-qa-docker.md) — BrowserStack fast start: reuse `BS_APP_ID_*` from `.env` (no upload), one build id per feature per OS, at most 10 parallel scenarios; `--threads` splits by feature.
- [refresh-browserstack-env-app-ids](workflows/refresh-browserstack-env-app-ids.md) — four subagents fetch the latest CI `bs://` ids; the parent writes `BS_APP_ID_*` in `instawork/e2e/mobile/.env` and shows previous vs new.
- [mobile-automation-pipeline](workflows/mobile-automation-pipeline.md) — BrowserStack test case → spec: trace → test-plan → orchestrator; parallel subagent shortcuts.
- [browserstack-release-feature-testing](workflows/browserstack-release-feature-testing.md) — changelog → one draft case per note under `<version> Release - Feature Testing` in the project's Release Feature Testing folder.
- [large-corpus-distillation-subagents](workflows/large-corpus-distillation-subagents.md) — map-reduce a huge message corpus: ~1500-item shards, one worker note each, waves of 7, fresh reducer over notes; reuse notes on dataset growth via id-based delta.
- [mobile-ci-allure-failure-triage](workflows/mobile-ci-allure-failure-triage.md) — use the job's custom `all_error_details.json`/`categories.json` (pre-deduplicated, retry-aware) instead of raw allure-results; one subagent per failing scenario.
- [web-ui-smoke-failure-triage](workflows/web-ui-smoke-failure-triage.md) — master deploy `web-ui-smoke-tests`: tests API + zip screenshots; `@flaky` still runs under p0/p1 smoke.
- [rice-prioritization](workflows/rice-prioritization.md) — compare initiatives with evidence-backed RICE scoring.
- [qa-platform-story-pointing](workflows/qa-platform-story-pointing.md) — Fibonacci QA story-point matrix (1–13) mapping points to time/effort/complexity/risk; split tasks above 8.
- [task-definition-readiness](workflows/task-definition-readiness.md) — evidence-based pre-draft and post-draft checks for implementation tasks.
- [self-review-workflow](workflows/self-review-workflow.md) — run personal code review and present fix items as a numbered evidence-backed table; `lint-e2e` unused-step scan is HEAD vs GitHub `master`.
- [mobile-test-impact-from-pr-or-branch](workflows/mobile-test-impact-from-pr-or-branch.md) — analyze PR/branch changed files, map mobile blast radius, and output a targeted vs smoke vs full-regression run plan.
- [ruff-noqa-cleanup-pipeline](workflows/ruff-noqa-cleanup-pipeline.md) — cloud-agent queue: one code set per tree; peel `# noqa`, fix; ignore the old autofix Prompt.
- [xpath-to-id-asana-task-format](workflows/xpath-to-id-asana-task-format.md) — board card shape: Summary/Steps/Locator map/Done when; field name Candidate id; optional screenshot context only.
- [xpath-to-id-app-target-resolution](workflows/xpath-to-id-app-target-resolution.md) — verify Candidate id against live screen owner (Hyperview Django `auto_id` vs RN `ids.ts`), not from an OR xpath.
- [asana-bug-report](workflows/asana-bug-report.md) — Bugs board filing: video→1fps+Whisper→trim clip, live template, health_check SHA per env, evidence, attachments.
- [qa-platform-pre-grooming](workflows/qa-platform-pre-grooming.md) — prep Planning "Needs Triage" before grooming: confirm quarter umbrellas, overlay last 1–2 weeks of meetings, propose keep/defer/split/drop read-only.
- [qa-platform-sprint-kickoff](workflows/qa-platform-sprint-kickoff.md) — dated markdown packet plus canvas; kickoff canvas is inventory (stats, carry/new); mid-sprint Monday is week two (last week plus this week's focus).
- [qa-platform-quarter-close-out](workflows/qa-platform-quarter-close-out.md) — end-of-quarter mission canvas: leaf-task % per milestone, committed-outcome as the headline, carry/pause/close; skip blockers and new-ideas when they are not real.
- [integration-branch](workflows/integration-branch.md) — collector PR only: merge other PRs into `integration/<topic>`, `integration` label, description lists `#` PRs.
- [play-console-discard-release](workflows/play-console-discard-release.md) — managed publishing: remove in-review/ready-to-publish changes, then discard the track release; Production dashboard is not discard.
- [app-store-connect-unsubmit-build](workflows/app-store-connect-unsubmit-build.md) — withdraw from review to Developer Rejected, then minus the Build; marketing version cannot be deleted.
- [instawork-local-hiring-opensearch-setup](workflows/instawork-local-hiring-opensearch-setup.md) — Jobs tab empty locally: OpenSearch sits behind the `full` compose profile; index + pipelines + sync, then `upsert_all_jobs()` to actually make jobs searchable.
## Tooling
<!-- cross-cutting tools: aws-cli, docker, just, uv, gh, appium, playwright -->
- [instawork-cursor-backend-debugging](tooling/instawork-cursor-backend-debugging.md) — attach Cursor to Django on port 5678 or pytest on 5700; Docker `/app` path mapping and reload gotcha.
- [django-get-or-create-conflict-test](tooling/django-get-or-create-conflict-test.md) — a conflicting row inserted inside `create()` is rolled back before the recovery lookup; insert after the miss and before the savepoint.
- [allure-testops-screenshots](tooling/allure-testops-screenshots.md) — TestOps failure artifacts by API: find the result in a launch, `/execution` step tree, retry summary, API timeline, screenshot; token name gotcha.
- [redshift-allure-ops](tooling/redshift-allure-ops.md) — Allure warehouse tables are schema `allure_ops` on cluster `instawork-dw`, database `instawork`; `scenarios.id` is the TestOps test result id.
- [redshift-copy-load-errors](tooling/redshift-copy-load-errors.md) — COPY `XX000` "check stl_load_errors" hides the bad row; only the loading user or a superuser can see it. The Finch plugin user sees none.
- [ruff-enable-excluded-tree](tooling/ruff-enable-excluded-tree.md) — split lint-gate vs format PRs; keep `extend-exclude` until format lands; parse errors cannot be ignored.
- [github-pr-gh-cli-cloud-agents](tooling/github-pr-gh-cli-cloud-agents.md) — use the gh CLI for PR create/edit/comment when the PR tool returns unauthenticated or repo-scoping errors.
- [git-shallow-fetch-unrelated-histories](tooling/git-shallow-fetch-unrelated-histories.md) — `fetch --depth=1` of master in a full clone makes merge refuse unrelated histories; unshallow, never allow-unrelated.
- [git-blame-phantom-file-add](tooling/git-blame-phantom-file-add.md) — blame and line-log stop on a commit that records a file as added while the parent diff is empty; blame the parent, and walk past tag-line rewrites.
- [asana-api-cloud-agents](tooling/asana-api-cloud-agents.md) — Asana REST API fallback when MCP auth is unavailable or errors; also the only way to list board sections and place tasks in a column.
- [circleci-api-cloud-agents](tooling/circleci-api-cloud-agents.md) — CircleCI REST API fallback when MCP returns 401 on cloud agents.
- [circleci-api-job-logs](tooling/circleci-api-job-logs.md) — how to fetch actual step log output (v1.1 API); v2 API is metadata-only.
- [circleci-api-artifact-download](tooling/circleci-api-artifact-download.md) — bulk-download all job artifacts; artifact urls 302-redirect, must use `curl -sSL` not bare urllib.
- [circleci-config-typed-parameters](tooling/circleci-config-typed-parameters.md) — integer/boolean params can't be empty; use a numeric sentinel + boundary conversion to keep optional semantics.
- [browserstack-app-automate-api-cloud-agents](tooling/browserstack-app-automate-api-cloud-agents.md) — BrowserStack App Automate REST API fallback when MCP returns 401 on cloud agents.
- [captain-tooling-shared-business](tooling/captain-tooling-shared-business.md) — captain scenarios get-or-create one business, so parallel runs adopt and then cancel each other's shifts.
- [appium-webview-context-chromedriver](tooling/appium-webview-context-chromedriver.md) — a listed WEBVIEW context is not attached until Chromedriver matches that WebView's Chrome; fix the server when the step passes elsewhere, native click is a fallback.
- [appium-mcp-browserstack-device-session](tooling/appium-mcp-browserstack-device-session.md) — Appium MCP + BrowserStack session gotchas: deep_link needs explicit package id, post-login welcome-popup close, My skills scroll depth, skill row locator shape.
- [browserstack-test-management-api](tooling/browserstack-test-management-api.md) — read/edit Test Management test cases; UI URL number to PR-/TC- id mapping gotcha.
- [browserstack-failure-triage](tooling/browserstack-failure-triage.md) — two-step BS session grounding then test-code grounding for failed App Automate sessions; RN-upgrade locator breakage signature.
- [instawork-automation-api](tooling/instawork-automation-api.md) — preflight + gotchas for the local /automation/api/v2 surface.
- [aws-cli-cloudwatch-lambda-logs](tooling/aws-cli-cloudwatch-lambda-logs.md) — query Lambda CloudWatch logs via AWS CLI; epoch-ms window trick; cold-start diagnosis from REPORT line.
- [slack-mcp-bulk-export](tooling/slack-mcp-bulk-export.md) — stream Slack MCP search to disk (concise + no context, cap pages) to avoid context exhaustion; concise-format dates are unreliable.
- [atlassian-confluence-cql-mcp](tooling/atlassian-confluence-cql-mcp.md) — bulk Confluence collection via MCP CQL; body.view expand caps at 50/page, paginate with the url-decoded cursor.
- [granola-mcp-bulk-export](tooling/granola-mcp-bulk-export.md) — Granola meeting index + transcripts via MCP; `list_meetings` is this week, last week, or last 30 days only; ~1 transcript/2min hard rate cap.
- [gws-slides-cli](tooling/gws-slides-cli.md) — create Google Slides with `gws`; batchUpdate schema, speaker notes, shape color gotcha.
- [hyperview-button-enabled-attribute](tooling/hyperview-button-enabled-attribute.md) — native `enabled` flips only on a disabled Hyperview touchable (behavior/action/href plus `disabled="true"`); label stays enabled; missing action defaults to push.
- [docker-compose-env-precedence](tooling/docker-compose-env-precedence.md) — empty exported shell var beats .env in `${VAR:-}` substitution; verify with `docker compose config`, unset then `up --force-recreate`.
- [cursor-canvas-authoring](tooling/cursor-canvas-authoring.md) — kickoff canvas: openFile button for packet; contributor Cards + UsageBar (not topLeftLabel prose); Stat key gotcha. Every canvas table starts with a `#` column.
- [python-local-https-certifi](tooling/python-local-https-certifi.md) — stdlib `urllib` on macOS framework Python fails CA verification; pass `ssl.create_default_context(cafile=certifi.where())`.
- [cursor-agent-worker-launchd](tooling/cursor-agent-worker-launchd.md) — cursor agent worker LaunchAgent: TCC blocks ~/Documents log path, Keychain inaccessible from launchd (embed token), bootstrap hangs with Node.js, exit 78 diagnosis.
- [appium-native-date-picker-locators](tooling/appium-native-date-picker-locators.md) — why OS-native date picker wheels can't take app-owned per-wheel ids; value-based wheel classification, and the Detox-vs-Appium `testID` trap.
- [third-party-compose-sdk-locators](tooling/third-party-compose-sdk-locators.md) — vendor SDK screens render as a Compose subtree with no `resource-id`; no app-owned id is possible, locate by content-desc/name; first casualty when `id=` flips to resourceId.
- [appium-native-camera-locators](tooling/appium-native-camera-locators.md) — OS camera controls: one Matches `IdSelector` alternating Android resource id and iOS name replaces the platform branch; `id=` is resource id, not content-desc.
- [mobile-rn-upgrade-locator-defect-signature](tooling/mobile-rn-upgrade-locator-defect-signature.md) — forensic signature for classifying a failed mobile test as automation/locator defect during RN upgrades: element visible in screenshot + id-based lookup fails + sibling id-lookups pass + reproducible every retry.
- [sentry-mobile-rn-upgrade-filter](tooling/sentry-mobile-rn-upgrade-filter.md) — filter Sentry for RN upgrade integration builds: prefer React Native Version over release/app version; automation/beta env scope; tag vs context searchability.
- [mobile-datadog-sdk-metric-check](tooling/mobile-datadog-sdk-metric-check.md) — manual Datadog SDK bump check: beta/release build, sampling 0 skips init, cancelled tti is dropped, pass is a new-SDK session plus a tti-/metric- log.
- [android-rn-native-crash-repro](tooling/android-rn-native-crash-repro.md) — reproduce Fabric teardown races: force destruction with "Don't keep activities" + process kill, match device ABI class, logcat tombstone tag gotcha.
- [ios-accessible-hides-child-testids](tooling/ios-accessible-hides-child-testids.md) — child `testID` visible on Android but missing on iOS: XCUITest prunes the subtree of an accessibility element; `accessible={false}` on the touchable restores one cross-platform locator pair.
- [ios-visible-false-under-transparent-overlay](tooling/ios-visible-false-under-transparent-overlay.md) — iOS `visible=false` while the node is on screen: a later absolute transparent sibling wins the hit test; Android `displayed` stays true. A root toast top sheet is the inverse: the covered control stays displayed, the click succeeds, and the toast consumes it. An unclosed bottom sheet produces both signatures at once, one per platform.
- [ios-native-tabbar-liquid-glass-locators](tooling/ios-native-tabbar-liquid-glass-locators.md) — iOS 26 native tabs are Buttons with title-case `name`; Hyperview tabs are Other with id `name`; match both, do not require TabBar ancestor.
- [ios-safari-deeplink-address-bar](tooling/ios-safari-deeplink-address-bar.md) — deep-link `TabBarItemTitle` is Safari's collapsed address button; an already-expanded field never receives the URL.
- [staging-hyperview-picker-repro](tooling/staging-hyperview-picker-repro.md) — staging2 select-wheel without OpenSearch: my-dynamic-form / storybook PickerField, or Django shell throwaway artform + intro URL.
- [hyperview-navigator-event-behaviors](tooling/hyperview-navigator-event-behaviors.md) — navigator behaviors only run on load or on-event; a behavior has no id, so append the follow-up onto the navigator.
- [hyperview-stale-press-handler](tooling/hyperview-stale-press-handler.md) — first press after a document update fires the previous element's href; behavior cache refreshes one commit late, label looks right, a wait never helps but a re-render does.

## Infra
<!-- ECS, RDS, ECR, networking, terraform envs -->
- [celery-worker-capacity](infra/celery-worker-capacity.md) — deployed Celery `--concurrency` and ECS `desired_count` live in `Instawork/infrastructure` task definitions, not the app repo.

## Repos
<!-- repo-specific learnings, grouped by repo -->
- [airflow-allure-launch-without-scenarios](repos/airflow-allure-launch-without-scenarios.md) — launch row commits before scenario fetch; an empty leaf or a failed scenario call is never retried.
- [mobile-git-worktree-docker](repos/mobile-git-worktree-docker.md) — git worktree + Docker crash: git rev-parse fails inside container, fix and stale-image workaround.
- [mobile-ios-autoAcceptAlerts-race](repos/mobile-ios-autoAcceptAlerts-race.md) — iOS autoAcceptAlerts race condition when asserting native dialogs; atomic page_source fix and scrollToElement fallback.
- [serverless-deploybot-run-test](repos/serverless-deploybot-run-test.md) — deploybot run-test threading architecture, Slack token identities, response_url gotchas, conversations.history approach.
- [infrastructure-sops-secrets](repos/infrastructure-sops-secrets.md) — SOPS + Terraform convention for SSM parameters: sops version, sops set, empty arrays gotcha, staging2 pattern.
- [test-automation-parallel-exit-code](repos/test-automation-parallel-exit-code.md) — parallel runner exits 1 despite "0 failed" summary: hook_failures + autoretry interaction, fix via JUnit override.
- [mobile-step-definition-conventions](repos/mobile-step-definition-conventions.md) — step definitions signal outcomes with assert (not return/raise), flag + assert after poll loops, branch only for ios/android, inline single-use logic instead of one-off `_*` helpers; unused steps via `behave --dry-run -f steps.usage`; no text fallback for undeployed Hyperview ids; Android clickable-wait belongs in Instatest.
- [mobile-automation-waits](repos/mobile-automation-waits.md) — replace fixed waits with real wait primitives; do not wait on copy; a clipped last tab can accept a click without selecting; loader-gone is racy and not universal; `screen should be loaded` often asserts nothing.
- [mobile-step-retry-retrying](repos/mobile-step-retry-retrying.md) — `retrying` `@retry` re-runs the whole step; `@step` must sit above it; `wait_fixed` is milliseconds and is skipped if omitted; scenario retry is already `TEST_GLOBAL_RETRIES`.
- [mobile-native-datepicker-automation-ids](repos/mobile-native-datepicker-automation-ids.md) — `@react-native-community/datetimepicker` spinner wheels cannot get per-wheel app IDs without fork/custom picker; use OS locators + index.
- [mds-storybook-form-field-ids](repos/mds-storybook-form-field-ids.md) — MDS form_field stories share Django name `field` so auto_id duplicates; set `attrs.id` to `id_field-N` for unique automation ids.
- [hyperview-local-mobile-sync](repos/hyperview-local-mobile-sync.md) — test local Hyperview source changes in mobile without publishing; Yarn 4 cannot run `yarn sync` against the Classic lockfile.
- [mobile-abilities-verify-now-flow](repos/mobile-abilities-verify-now-flow.md) — data config required to trigger the post-booking ABILITIES "Verify now" flow; Professional preset gotcha.
- [mobile-pre-booking-hold-review](repos/mobile-pre-booking-hold-review.md) — seed unbooked far-out gig + first-company worker; in-app Book starts HOLD_GIG_DETAILS_REVIEW and disabled Continue.
- [mobile-vendored-instatest](repos/mobile-vendored-instatest.md) — src/instatest is a git-ignored editable install (search with --no-ignore); framework changes belong in test-automation; iOS directional scroll vs Android scrollIntoView.
- [e2e-constance-parallel-race](repos/e2e-constance-parallel-race.md) — `@set_constance` writes a global backend key with no scoping; races within a run *and* across per-platform launches on the shared env. Detect by overlapping launch windows + HAR bodies; fix with per-entity test data, not serialisation. A shared catalog delete is the same race: accept the latest live row per user.
- [instawork-docker-network](repos/instawork-docker-network.md) — instawork_default is external (shared with finch); fresh envs/CI must `docker network create` it first (web-tests-localhost).
- [instawork-docker-uv-data-stale](repos/instawork-docker-uv-data-stale.md) — ModuleNotFoundError for a package already in requirements.txt: `uv_data` hides image site-packages; `just pip-sync` refreshes it.
- [instawork-instatest-unit-tests](repos/instawork-instatest-unit-tests.md) — run `lib/instatest` pytest files via the `automation` compose service; no CI job gates them, so check `master` before blaming a branch.
- [instawork-mobile-e2e-shared-pythonpath](repos/instawork-mobile-e2e-shared-pythonpath.md) — compose PYTHONPATH now matches CI (`/workspace:/workspace/lib/instatest`); older trees are instatest-only and need environment.py's sys.path bootstrap first.
- [mobile-e2e-endpoint-coverage](repos/mobile-e2e-endpoint-coverage.md) — a renamed scenario stales its endpoint-coverage YAML; update the label and rename the file with `build_coverage_file_name`. Both coverage CI jobs fail on the same validator.
- [instawork-prod-only-settings-local-500](repos/instawork-prod-only-settings-local-500.md) — view 500s locally on `None` service-URL setting: prod-injected, unset in every local config; short-circuit the constance gate upstream (and check that branch's fallback) or set a dummy URL.
- [instawork-worker-app-shell-sendbird-500](repos/instawork-worker-app-shell-sendbird-500.md) — generic Hyperview 500 on GET /worker_app/app after login: match x-request-id in Sentry; uncaught Sendbird session-token fetch. An empty token after a worker burst is a Sendbird body with no token. Another token request does not create a missing user.
- [instawork-local-sendbird-user-id-collision](repos/instawork-local-sendbird-user-id-collision.md) — a new local worker can open another person's Sendbird chats because the shared app keys users by the small local id.
- [hyperview-footer-safe-area](repos/hyperview-footer-safe-area.md) — triage a CTA hidden behind the Android nav bar: grep the on-screen string, follow the footer include, look for the missing bottom `safe_area`, diff against the working sibling footer.
- [hyperview-optional-field-label](repos/hyperview-optional-field-label.md) — "(optional)" on a Hyperview field heading comes from the label template when `required` is false, not from the Python `label` string.
- [hyperview-tab-replace-dismisses-welcome](repos/hyperview-tab-replace-dismisses-welcome.md) — tab REPLACE `setParams` paints into the focused welcome modal; navigate root `tabs-route` with nested screen and url.
