# Wiki index

> Catalog of every wiki page, grouped by area, one line each.
> Read this first to find relevant pages and to avoid creating duplicates.
> Last updated: 2026-07-01 | Total pages: 29

## Preferences
<!-- how the user wants me to behave; read global.md every orient, context files when relevant -->
- [global](preferences/global.md) — behavioral preferences that apply everywhere. Always read at orient.
- [documentation](preferences/documentation.md) — how to write docs (sentence case, no em dashes).

## Workflows
<!-- multi-step runbooks -->
- [web-e2e-local](workflows/web-e2e-local.md) — run Playwright web e2e against the local backend on :8080.
- [mobile-android-login-local](workflows/mobile-android-login-local.md) — run applicant Android login via Instatest/Docker against local :8080.
- [mobile-browserstack-qa-docker](workflows/mobile-browserstack-qa-docker.md) — run applicant tests on BrowserStack against QA via Docker; feature-branch instatest + device-triple gotcha.
- [mobile-automation-pipeline](workflows/mobile-automation-pipeline.md) — BrowserStack test case → spec: trace → test-plan → orchestrator; parallel subagent shortcuts.
- [large-corpus-distillation-subagents](workflows/large-corpus-distillation-subagents.md) — map-reduce a huge message corpus: ~1500-item shards, one worker note each, waves of 7, fresh reducer over notes; reuse notes on dataset growth via id-based delta.
- [mobile-ci-allure-failure-triage](workflows/mobile-ci-allure-failure-triage.md) — use the job's custom `all_error_details.json`/`categories.json` (pre-deduplicated, retry-aware) instead of raw allure-results; one subagent per failing scenario.
## Tooling
<!-- cross-cutting tools: aws-cli, docker, just, uv, gh, appium, playwright -->
- [github-pr-gh-cli-cloud-agents](tooling/github-pr-gh-cli-cloud-agents.md) — use the gh CLI for PR create/edit/comment when the PR tool returns unauthenticated or repo-scoping errors.
- [asana-api-cloud-agents](tooling/asana-api-cloud-agents.md) — Asana REST API fallback when MCP auth is unavailable on cloud agents.
- [circleci-api-cloud-agents](tooling/circleci-api-cloud-agents.md) — CircleCI REST API fallback when MCP returns 401 on cloud agents.
- [circleci-api-job-logs](tooling/circleci-api-job-logs.md) — how to fetch actual step log output (v1.1 API); v2 API is metadata-only.
- [circleci-api-artifact-download](tooling/circleci-api-artifact-download.md) — bulk-download all job artifacts; artifact urls 302-redirect, must use `curl -sSL` not bare urllib.
- [circleci-config-typed-parameters](tooling/circleci-config-typed-parameters.md) — integer/boolean params can't be empty; use a numeric sentinel + boundary conversion to keep optional semantics.
- [browserstack-app-automate-api-cloud-agents](tooling/browserstack-app-automate-api-cloud-agents.md) — BrowserStack App Automate REST API fallback when MCP returns 401 on cloud agents.
- [browserstack-test-management-api](tooling/browserstack-test-management-api.md) — read/edit Test Management test cases; UI URL number to PR-/TC- id mapping gotcha.
- [browserstack-failure-triage](tooling/browserstack-failure-triage.md) — two-step BS session grounding then test-code grounding for failed App Automate sessions; RN-upgrade locator breakage signature.
- [instawork-automation-api](tooling/instawork-automation-api.md) — preflight + gotchas for the local /automation/api/v2 surface.
- [aws-cli-cloudwatch-lambda-logs](tooling/aws-cli-cloudwatch-lambda-logs.md) — query Lambda CloudWatch logs via AWS CLI; epoch-ms window trick; cold-start diagnosis from REPORT line.
- [slack-mcp-bulk-export](tooling/slack-mcp-bulk-export.md) — stream Slack MCP search to disk (concise + no context, cap pages) to avoid context exhaustion; concise-format dates are unreliable.
- [atlassian-confluence-cql-mcp](tooling/atlassian-confluence-cql-mcp.md) — bulk Confluence collection via MCP CQL; body.view expand caps at 50/page, paginate with the url-decoded cursor.
- [granola-mcp-bulk-export](tooling/granola-mcp-bulk-export.md) — Granola meeting index + transcripts via MCP; Me:/Them: labels, ~1 transcript/2min hard rate cap, encrypted local cache (no bypass).
- [gws-slides-cli](tooling/gws-slides-cli.md) — create Google Slides with `gws`; batchUpdate schema, speaker notes, shape color gotcha.
- [docker-compose-env-precedence](tooling/docker-compose-env-precedence.md) — empty exported shell var beats .env in `${VAR:-}` substitution; verify with `docker compose config`, unset then `up --force-recreate`.
- [cursor-agent-worker-launchd](tooling/cursor-agent-worker-launchd.md) — cursor agent worker LaunchAgent: TCC blocks ~/Documents log path, Keychain inaccessible from launchd (embed token), bootstrap hangs with Node.js, exit 78 diagnosis.
- [mobile-rn-upgrade-locator-defect-signature](tooling/mobile-rn-upgrade-locator-defect-signature.md) — forensic signature for classifying a failed mobile test as automation/locator defect during RN upgrades: element visible in screenshot + id-based lookup fails + sibling id-lookups pass + reproducible every retry.

## Infra
<!-- ECS, RDS, ECR, networking, terraform envs -->

## Repos
<!-- repo-specific learnings, grouped by repo -->
- [mobile-git-worktree-docker](repos/mobile-git-worktree-docker.md) — git worktree + Docker crash: git rev-parse fails inside container, fix and stale-image workaround.
- [mobile-ios-autoAcceptAlerts-race](repos/mobile-ios-autoAcceptAlerts-race.md) — iOS autoAcceptAlerts race condition when asserting native dialogs; atomic page_source fix and scrollToElement fallback.
- [serverless-deploybot-run-test](repos/serverless-deploybot-run-test.md) — deploybot run-test threading architecture, Slack token identities, response_url gotchas, conversations.history approach.
- [infrastructure-sops-secrets](repos/infrastructure-sops-secrets.md) — SOPS + Terraform convention for SSM parameters: sops version, sops set, empty arrays gotcha, staging2 pattern.
- [test-automation-parallel-exit-code](repos/test-automation-parallel-exit-code.md) — parallel runner exits 1 despite "0 failed" summary: hook_failures + autoretry interaction, fix via JUnit override.
- [mobile-step-definition-conventions](repos/mobile-step-definition-conventions.md) — step definitions signal outcomes with assert (not return/raise), flag + assert after poll loops, and branch only for ios/android (no unsupported-platform else).
- [mobile-abilities-verify-now-flow](repos/mobile-abilities-verify-now-flow.md) — data config required to trigger the post-booking ABILITIES "Verify now" flow; Professional preset gotcha.
- [mobile-vendored-instatest](repos/mobile-vendored-instatest.md) — src/instatest is a git-ignored editable install (search with --no-ignore); framework changes belong in test-automation; iOS directional scroll vs Android scrollIntoView.
- [instawork-docker-network](repos/instawork-docker-network.md) — instawork_default is external (shared with finch); fresh envs/CI must `docker network create` it first (web-tests-localhost).
