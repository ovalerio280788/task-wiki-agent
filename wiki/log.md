# Wiki log

> Append-only chronological record of captured learnings. Newest entries at the top.
> Format per entry: `## [YYYY-MM-DD] <action> | <page>` where action is one of: create, update, archive, lint.
> When adding an entry, insert it directly below this header, above the previous newest entry.
> Rotate when this file exceeds ~500 entries or at year end: rename to `log-YYYY.md` and start fresh.

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
