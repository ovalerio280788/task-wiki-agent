# Workspace repo catalog

High-level routing directory for an AI agent deciding which repo to open before reading files. Each top-level folder is its own git repository. Read the routing table first, confirm with the repo entry, then open files.

The `instawork` repo is the source of truth for the staffing marketplace. Almost every other repo orbits it: it serves the data, the product APIs, and the `/automation/api/v2` surface that the test repos depend on.

Important seam: the native mobile apps are heavily backend driven. The `instawork` backend serves not only the business logic and APIs but also most of the mobile UI itself, as server rendered screen templates and design-system components (`apps/mobile/`, Django Template Language plus `hxpy`, rolled out per endpoint). So a "mobile UI" change or bug is often a backend change in `instawork`, not a `mobile` repo change. See the mobile UI disambiguation below.

## Routing table

Pick the single primary repo for the task. Disambiguations for the test and infra clusters follow below the table.

| Task or signal | Primary repo |
|---|---|
| Marketplace business logic, Django models, migrations, Celery tasks, REST endpoints | `instawork` |
| Partner/pro web dashboards, React micro-frontends | `instawork` |
| Mobile UI served by the backend: screen templates, MDS/`hxpy` components, server-driven layouts, content/copy in screens | `instawork` (`apps/mobile/`) |
| The `/automation/api/v2` test-data endpoints (the API itself) | `instawork` |
| Web browser e2e (Playwright, TypeScript) | `instawork` (`e2e/web/`) |
| Native app shell: React Native client code, navigation, native iOS/Android, Fastlane release | `mobile` |
| Mobile e2e test cases: `.feature` files, page objects, `test_data.json`, locators | `mobile` (`*/test/automation/`) |
| Mobile test engine itself: `instatest` CLI, Appium/Behave plumbing, parallel runner, reporting | `test-automation` |
| AI-assisted QA tooling: TestRail as code, API v2 data generation, Allure failure AI review | `ai-manual-tester` |
| QA/testing theory, ISTQB definitions, test design technique reference | `istqb-knownledge` |
| Ad hoc QA reporting scripts: CircleCI/TestOps exports, Slack history, GIF, bulk test businesses | `utilitarian-scripts` |
| Production AI agents/LLM features serving the app | `finch` |
| Scheduled/batch jobs, DAGs, ETL into Redshift/MySQL, MWAA variables | `airflow` |
| AWS Lambda functions, Slack bots (deploy bot, ops bot), event-driven glue, lightweight ETL | `serverless` |
| Terraform, AWS provisioning, ECS task defs, IAM, RDS, ECR repos, deploy pipeline | `infrastructure` |
| Custom/base Docker image definitions, image versions, ECR image builds | `docker-custom-images` |
| Sanitized demo/config database snapshot from production | `config-sync` |

## Disambiguation: the test cluster

Four repos touch testing. Decide by artifact, not by the word "test".

- `instawork` (`e2e/web/`): browser end to end, TypeScript Playwright, drives the running web app and seeds data via the automation API. Web flows only.
- `mobile` (`*/test/automation/`): the mobile test suite content. Scenario `.feature` files, step definitions, page objects, `test_data.json`. Bugs in mobile scenarios or locators live here. **Active migration in progress (as of 2026-06-30):** this content is being moved into `instawork` (`e2e/mobile/`); both repos receive commits and are periodically reconciled (`chore(mobile-e2e): reconcile mobile automation drift...`). CircleCI jobs that actually execute the applicant/business app suites currently run from `instawork/e2e/mobile/<app>/`, not from `mobile`. Check which copy a CI job checked out before editing scenarios/steps; do not assume `mobile` is the live copy.
- `test-automation`: the `instatest` framework only (the engine the mobile suite imports as a pip package). Bugs in the CLI, Appium/Behave plumbing, drivers, parallelism, or reporting live here, not the scenarios.
- `ai-manual-tester`: AI-driven QA productivity tooling (TestRail sync, API v2 data generation workflows, Allure report triage). Not an automated suite.
- `istqb-knownledge`: testing theory reference content, no executable code.
- `utilitarian-scripts`: personal one-off QA reporting/data scripts, not a suite and not framework code.

Rule of thumb: scenario or locator fails -> `mobile`; the runner/engine misbehaves for all mobile suites -> `test-automation`; a web flow fails -> `instawork/e2e/web`; you need data set up via API or a TestRail/Allure workflow -> `ai-manual-tester`.

## Disambiguation: the infra and deploy cluster

Five repos touch infrastructure. Decide by layer.

- `infrastructure`: Terraform source of truth for long-lived AWS resources and the `deploy-service.sh` other repos call. Provisioning and deploy plumbing.
- `docker-custom-images`: builds and versions the container images (base and service) pushed to ECR that the other repos consume.
- `serverless`: application-level AWS Lambda functions and bots (Serverless Framework). Function code, not the AWS account scaffolding.
- `airflow`: scheduled DAG-based batch orchestration on MWAA.
- `config-sync`: a single batch job that produces a sanitized demo database snapshot. Not general infra.

Rule of thumb: declaring/changing an AWS resource or ECS deploy -> `infrastructure`; the Dockerfile/image contents -> `docker-custom-images`; a Lambda or Slack bot -> `serverless`; a cron/DAG/ETL -> `airflow`.

## Disambiguation: AI repos

- `finch`: production AI agent platform (FastAPI, LangChain/LangGraph) that serves the `instawork` app. Bidirectionally coupled to the backend.
- `ai-manual-tester`: internal QA AI tooling, unrelated to the product runtime.

## Disambiguation: mobile UI (server-driven vs native)

The native apps are mostly backend driven, so "mobile UI" is split across two repos. Decide by what actually renders the screen.

- `instawork` (`apps/mobile/`): the source of most mobile screens. Server rendered UI templates and the mobile design system (MDS) components, built on Django Template Language plus `hxpy`, delivered to the apps at runtime and rolled out per endpoint (`hxpy_endpoint_rollout`). A screen's layout, wording, fields, validation, or which component shows usually lives here.
- `mobile`: the native client shell that hosts and renders what the backend sends, plus genuinely native concerns: navigation, native modules, device/permissions, build and release, and any screens still implemented purely in React Native.

Rule of thumb: if a screen's content, layout, or copy is wrong, suspect `instawork/apps/mobile/` first; if the app crashes, fails to render the server payload, or the issue is navigation/native/build/release, look in `mobile`. When unsure, check whether the screen has an `hxpy`/MDS template in `instawork` before editing `mobile`.

---

## Repos

### instawork
- Purpose: core Django monolith plus web frontend for the two-sided staffing marketplace. Holds the bulk of business logic, MySQL models, Celery async tasks, and all product HTTP APIs. Web frontend is hybrid: legacy Django templates plus React micro-frontends in `web_frontend/`. Critically, `apps/mobile/` also serves most of the native apps' UI as server-rendered screen templates and design-system (MDS) components (Django Template Language plus `hxpy`, rolled out per endpoint), so the backend drives mobile UI, not just data. Embeds a Playwright web e2e suite at `e2e/web/` and exposes a dedicated `/automation/api/v2/` surface for test data setup.
- Business role: central product and system of record. Owns matching, scheduling, payments, hiring, and partner/pro flows.
- Tech: Python 3.9 / Django, DRF, MySQL, Celery + Redis, OpenSearch, Mercure; TypeScript, React, single-spa, Webpack, orval (OpenAPI client gen); Playwright; Docker Compose, just, uv.
- When to look here: marketplace logic, models/migrations, REST endpoints, web dashboards, micro-frontends, Celery tasks, the automation API, web e2e, and most native mobile screens (server-rendered UI templates and MDS components in `apps/mobile/`: layout, copy, fields, validation, which component renders). Also now the live target for the mobile e2e suite mid-migration (`e2e/mobile/applicant-app/`, `e2e/mobile/business-app/`, `e2e/mobile/shared/`) — CircleCI jobs run scenarios from here. NOT for the native client shell, navigation, native modules, or app build/release (that is `mobile`), infra (`infrastructure`), DAGs (`airflow`), Lambdas (`serverless`).
- Cross-repo: serves the mobile UI to the `mobile` apps as server-rendered `apps/mobile/` templates/components (the backend drives mobile UI; a mobile screen bug often lives here). Serves `/automation/api/v2/` (`backend/automation_api_v2_urls.py`) consumed by `mobile` Instatest, `instawork/e2e/web`, and `ai-manual-tester` data gen. Generates typed frontend clients via `orval.config.js` from per-app `swagger.json`. Integrates `config-sync` via `docker-compose.config-sync.yml`. Consumes `finch` AI through `lib/ml_client/services/finch_agent.py` (`iw_finch_client`) and exposes an `IsFinchWebhook`. Its mobile design system is also synced by `serverless` `iwds` and scraped by `utilitarian-scripts` `mds-stories`. Deployed by `infrastructure`; runs on base images from `docker-custom-images`.

### mobile
- Purpose: Instawork's native apps as a Yarn workspace monorepo: `applicant-app` (pro/worker), `business-app` (partner), shared `core-mobile`. This is the native client shell; much of the actual screen UI is server-rendered by the `instawork` backend (`apps/mobile/`) and hosted here, so this repo leans toward navigation, native modules, device concerns, and build/release rather than owning all screen markup. Embeds a full Python Behave + Appium e2e suite ("Instatest") under `*/test/automation` plus shared QA utils. App code is TypeScript/React Native; the embedded automation is Python.
- Business role: the shipped iOS and Android apps that let workers book shifts and businesses staff them. The embedded suite validates those apps on device before release.
- Tech: React Native 0.77, TypeScript, Redux, CocoaPods/Android, Yarn 4, Fastlane; automation in Python, Behave, Appium, BrowserStack, Allure.
- When to look here: the native client shell, navigation, native modules/device concerns, native build/release issues, screens still implemented purely in React Native, and the mobile e2e suite content (`.feature` files, page objects, Instatest runs). NOT for server-driven screen UI/templates/copy/layout (that is `instawork` `apps/mobile/`), NOT the backend logic/APIs (`instawork`), NOT the web Playwright suite (`instawork/e2e/web`), NOT the Instatest framework source (`test-automation`).
- Cross-repo: renders mobile UI served by the `instawork` backend (`apps/mobile/` server-rendered templates/MDS components via `IW_DOMAIN`/`INSTAWORK_URL_OVERRIDE`, e.g. webviews and `/mobile/storybook`), so the app is largely backend driven for screens, not just data. Depends on `test-automation` directly via `requirements_qa.in` (`-e git+ssh://...test-automation.git@master#egg=instatest`); a vendored copy sits at `src/instatest/`. Talks to the `instawork` backend at `localhost:8080` via `/automation/api/v2` (`.env.example`, `shared_qa_utils/apiv2/api_client.py`). Its automation images and requirements are pulled by `docker-custom-images` (`instawork-qa-automation`). Mobile e2e is triggered by `serverless` deploybot.

### test-automation
- Purpose: the `instatest` framework, a standalone pip-installable BDD mobile test runner (Behave + Appium). Provides the `instatest run` CLI, parallel execution, driver/device management (local, BrowserStack, Sauce Labs), Allure/JUnit reporting, and page-object abstractions. It is the engine, not a test suite.
- Business role: centralizes the reusable mobile automation engine so app repos do not reimplement Appium plumbing, parallelism, and reporting.
- Tech: Python 3.8, forked `behave-parallel`, Appium-Python-Client, allure-behave, boto3, pymysql; Drone CI.
- When to look here: changes to the engine itself (CLI, Behave hooks, drivers, selectors, parallel execution, reporting) or framework bugs shared across mobile suites. NOT for actual test cases (those are in `mobile`), NOT the web Playwright suite, NOT `ai-manual-tester`.
- Cross-repo: consumed by `mobile` as a pip dependency and invoked via `python -m instatest run`. Backend coupling is generic: it reads a `backend_domain` from the consumer's `test_data.json`, so the real `instawork` URLs are configured in `mobile`, not here.

### ai-manual-tester
- Purpose: QA tooling monorepo of AI-assisted utilities for manual and exploratory testing. Manages TestRail cases as code (`testrail_as_code/`), generates test data through the Instawork automation API via Cursor workflows (`apiv2_data_gen/`), and reviews Allure failure reports with custom Cursor agents/skills (`allure-report/`).
- Business role: speeds up QA work: TestRail sync, fast test data setup, and AI triage of nightly automation failures so bugs route to the right owner.
- Tech: Python 3.11 (pandas, requests, PyYAML), uv/Makefile, Cursor MCP (swagger-mcp), Allure HTML, curl-based API workflows.
- When to look here: TestRail case sync/export, generating test data via API v2 workflows, or AI analysis of Allure reports. NOT for product code, the actual automated suites (`test-automation`/`mobile`), or backend API implementation (`instawork`). No app runtime or CI here.
- Cross-repo: targets the `instawork` automation API (`apiv2_data_gen/.cursor/rules/data_gen_main.md` points swagger MCP at `/automation/api/v2/swagger` and curls `/workers`, `/shift-groups`). Consumes Allure artifacts produced by `mobile`/`test-automation` runs. External: TestRail. No code-level imports of siblings.

### finch
- Purpose: Python/FastAPI service hosting Instawork's production AI agents (LangChain/LangGraph), with a React playground (`web/`), Dramatiq workers, and an OpenAPI-generated client (`client/`).
- Business role: the internal AI agent platform that adds LLM features to the product.
- Tech: Python, FastAPI, LangChain/LangGraph, Dramatiq, React.
- When to look here: AI agent definitions, LLM orchestration, prompt/agent endpoints, the agent playground. NOT for non-AI backend logic (`instawork`), NOT QA AI tooling (`ai-manual-tester`).
- Cross-repo: bidirectionally coupled to `instawork`. Finch calls the Django backend over HTTP (`IW_BACKEND_ENDPOINT`, basic auth) and shares the `instawork_default` docker network; `instawork` consumes Finch via `lib/ml_client/services/finch_agent.py` and `iw_finch_client`. Invoked over HTTP by `airflow` (`sfdc_contact_retention_sync`) and `serverless` devopsbot (`https://finch.instawork.com/...`). Provisioned by `infrastructure` (`live/production/services/finch/`).

### airflow
- Purpose: Instawork's Apache Airflow DAG bag (~25 DAGs under `dags/`) for scheduled data pipelines, ML sync jobs, monitoring/alerting, and ops automation. Runs on AWS MWAA; deploy is `aws s3 sync` to `s3://instawork-airflow-dags` via CircleCI on merge to main. Includes scripts to sync MWAA variables/connections/pools from sops-encrypted secrets.
- Business role: centralizes scheduled/batch orchestration (ETL to Redshift/MySQL, ML loading, Salesforce/TestRail/Allure syncs, DB maintenance, alerting) outside the app request path.
- Tech: Python 3.11, Apache Airflow 3.0.6 (`apache-airflow[amazon]`), AWS MWAA, boto3, pandas/pyarrow, dbt-cloud; ruff, mypy, pytest; CircleCI.
- When to look here: a scheduled/cron job, a DAG, batch ETL, MWAA variables/connections/pools, or job-level OpsGenie/Slack alerting. NOT for the web/API backend (`instawork`), realtime Lambdas (`serverless`), the AI platform (`finch`), mobile, or Terraform (`infrastructure`).
- Cross-repo: orchestrates the deployed `config-sync` ECS task (`dags/config_sync/dag.py`). Syncs `test-automation` status via Allure TestOps and TestRail (`dags/testrail_automation_sync/`, `dags/allure_test_ops_analytics/`). Calls `finch` over HTTP (`sfdc_contact_retention_sync/llm_helper.py`). MWAA infra and image args are owned by `infrastructure` (`services/airflow/main.tf`). Integration is via deployed services, APIs, S3, and Redshift, not code imports.

### serverless
- Purpose: monorepo of independent AWS Lambda services (Serverless Framework v3). Each `services/` subfolder is a standalone deployable: deploy/automation triggers (`deploybot`), OpsGenie alert bot (`devopsbot`), data ETL (`DataPipeline`, `EventCollectorDataSink`, `RedshiftDecrypt`), design-system sync (`iwds`), and utilities.
- Business role: centralizes event-driven, scheduled, and Slack-triggered glue code that does not belong in the main backend, run cheaply as Lambdas.
- Tech: Python 3.9 and Node.js, Serverless Framework 3.38, AWS Lambda/API Gateway/S3/Redshift/SSM, CircleCI, Sentry, Datadog, Slack API.
- When to look here: Lambda functions, Slack slash-command bots, scheduled cron jobs, lightweight ETL into S3/Redshift. NOT for Terraform/long-lived infra (`infrastructure`), NOT DAG batch orchestration (`airflow`), NOT app logic (`instawork`).
- Cross-repo: deploybot triggers the `instawork` CircleCI pipeline and health-checks `app.instawork.com`; triggerautomation triggers `mobile`/`test-automation` e2e CircleCI runs. devopsbot calls `finch` (`https://finch.instawork.com/devops-bot/run`). Reuses IAM/Redshift resources defined in `infrastructure`. `iwds` bridges Figma, `mobile`, and web design-system sources. Coupling is via HTTP, CI triggers, and shared AWS SSM/IAM, not imports.

### infrastructure
- Purpose: Terraform monorepo provisioning all Instawork AWS infrastructure across environments (`live/production`, `staging2`, `qa`, `demo`, etc.). Defines core AWS primitives (VPC, ECS, Aurora/RDS, ECR, S3, IAM, MSK, OpenSearch, SageMaker, Lambda) and per-service stacks under `live/<env>/services/`. Holds `scripts/deploy-service.sh` that other repos call to ship image tags to ECS.
- Business role: single source of truth for cloud infrastructure; provides a paved-road framework for new internal apps.
- Tech: Terraform (HCL), AWS, CircleCI, shell; references external `Instawork/infrastructure-modules`.
- When to look here: AWS resource changes (VPC, IAM, ECR, ECS task defs, RDS, MSK, S3, DNS, SSM secrets), creating an ECR repo or service stack, environment provisioning, or deploy-pipeline plumbing. NOT for app/business logic, Dockerfile authoring (`docker-custom-images`), or Lambda function code (`serverless`).
- Cross-repo: central deploy hub. Deploys `instawork` (`services/django_services/main.tf`), `config-sync` (full `services/config-sync/` stack), `finch`, `airflow`, and defines `serverless-*` ECR repos. `deploy-service.sh` is "called by other repos, like instawork, to deploy a new version of a service." Declares base-image ECR repos that `docker-custom-images` builds into (relationship indirect, no explicit pointer found).

### docker-custom-images
- Purpose: monorepo of ~25 independent custom Docker image definitions (nginx, opensearch, instawork-base, kafka/zookeeper/schema-registry, instawork-qa-automation, proxysql, deepnote, dgraph, python base images). Each folder has a Dockerfile, build.sh, and version file. CircleCI builds changed images and pushes to AWS ECR.
- Business role: centralizes building, versioning, and publishing the base and service images the rest of the stack consumes, so app/infra repos pull pinned prebuilt images.
- Tech: Dockerfiles plus Bash; CircleCI; AWS CLI/ECR; hadolint.
- When to look here: a custom/base Docker image, its OS packages, image tags, ECR pushes, or the image build pipeline. NOT for application source, business logic, test suites, or infra provisioning (`infrastructure`).
- Cross-repo: `instawork-base` is the base image for the `instawork` Python backend. `instawork-qa-automation` fetches `mobile`'s `test/automation/requirements.txt` at build time and builds for `applicant-app`/`business-app`. `proxysql-cluster-deploy` clones and `terraform apply`s `infrastructure`. Kafka/zookeeper/schema-registry/mercure/opensearch images are consumed by `instawork` via ECR tags.

### config-sync
- Purpose: standalone Python batch job that takes the latest production RDS snapshot of the Instawork MySQL DB, copies only configuration tables into a clean DB, sanitizes it (admin/dev users, experiment traffic forcing, constance overrides, PII stripping), validates foreign keys, then snapshots and shares the result to S3/SSM. Entry point `src/main.py`.
- Business role: produces a safe, lightweight config/demo database snapshot from production so dev and demo/QA environments get realistic configuration data without real user PII.
- Tech: Python 3.9, pymysql, boto3, sentry-sdk; mysqldump/mysql CLI; Docker on AWS Fargate ECS; CircleCI plus Terraform.
- When to look here: config/demo snapshot generation, the synced configuration table list (`src/config_table_names.py`), constance overrides for demo (`src/constance_overrides.py`), experiment forcing, PII stripping, or the ConfigSync ECS task. NOT for the app itself, schema/migration authoring, runtime feature flags, or general infra.
- Cross-repo: operates directly on the `instawork` backend MySQL database (queries `instawork.internal_migrationsha`, `USE instawork;`; all synced tables are Instawork Django tables). Deployed via `infrastructure` Terraform (`live/production/services/config-sync`, cloned in CircleCI). Its ECS run is orchestrated by the `airflow` `config_sync` DAG. Integrated locally by `instawork` via `docker-compose.config-sync.yml`.

### istqb-knownledge
- Purpose: static QA knowledge base. Four markdown summaries (Foundation, Advanced Test Analyst, Test Management, Test Automation), the source syllabus PDF, and the extraction prompt. Written as terse directive bullets for machine consumption.
- Business role: encodes formal QA/testing best practices as reference material so AI agents doing QA work ground decisions in ISTQB standards.
- Tech: content only (markdown plus one PDF), no code.
- When to look here: QA testing theory, ISTQB terminology, test design techniques, coverage/risk concepts, canonical definitions. NOT for executable tests, frameworks, runnable code, or product-specific anything.
- Cross-repo: none. Personal repo (`antonyfuentesdev`). Conceptually adjacent to `ai-manual-tester` and `test-automation` as shared reference, but zero technical coupling.

### utilitarian-scripts
- Purpose: personal grab-bag of small, independent Python utility scripts, each in its own folder with its own requirements. Automate one-off QA/productivity chores: Allure/TestOps summaries, CircleCI pipeline/schedule exports, Slack history export, storybook label scraping, bulk test business creation, video-to-GIF. No shared package or entry point.
- Business role: lets the QA lead script repetitive reporting and test-data tasks without polluting the product or test-automation repos. A private toolbox, not a deployed service.
- Tech: Python only (requests, jinja2, faker, moviepy/Pillow, python-dotenv); per-script deps.
- When to look here: ad hoc QA/reporting scripts (Allure/TestOps summaries, CircleCI exports, Slack exports, storybook scraping, GIF generation). NOT for product code, the backend, the actual suites, CI config, or anything deployed/imported by another repo.
- Cross-repo: external coupling only. `business-copy/main.py` uses `instawork` Django ORM models (run inside the backend shell); `mds-stories` hits the local backend storybook at `0.0.0.0:8080`; CircleCI scripts target the Instawork org and reference a `test-automation` param; TestOps reporting hits `instawork.testops.cloud`. No in-workspace imports. Personal repo (`antonyfuentesdev`).

---

## Local-only utility folders (not git repos)

- `cursor-stats`: small local script folder (`fetch_skills.sh`, `data/`, `.env`) for pulling Cursor usage/skills stats. Not a repository.
- `cursor-agent-logs`: a single local log file (`agent-worker.log`). Not a repository.
