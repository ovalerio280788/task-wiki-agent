---
title: Mobile automation pipeline (BrowserStack → spec)
updated: 2026-06-25
tags: [testing, mobile, runbook, capability]
area: workflows
---

## Context

Turning a manual BrowserStack test case into a mobile automation spec. Applies when given a
BrowserStack Test Management URL from an Asana task and asked to automate it. Chains three skills:
`tracing-feature-flows` → `test-plan-generation` → `mobile-automation-orchestrator`.

## What works

### 1. Fetch inputs

```bash
# Asana task (extract task GID from URL)
export ASANA_API_TOKEN=$(grep '^export ASANA_API_TOKEN=' ~/.zprofile | sed 's/^export ASANA_API_TOKEN="\(.*\)"$/\1/')
curl -sS "https://app.asana.com/api/1.0/tasks/<task_gid>?opt_fields=name,notes,custom_fields" \
  -H "Authorization: Bearer $ASANA_API_TOKEN"

# BrowserStack test case (wiki: tooling/browserstack-test-management-api.md)
export BROWSERSTACK_USERNAME=$(grep '^export BROWSERSTACK_USERNAME=' ~/.zprofile | sed 's/^export BROWSERSTACK_USERNAME="\(.*\)"$/\1/')
export BROWSERSTACK_ACCESS_KEY=$(grep '^export BROWSERSTACK_ACCESS_KEY=' ~/.zprofile | sed 's/^export BROWSERSTACK_ACCESS_KEY="\(.*\)"$/\1/')
curl -sS -u "${BROWSERSTACK_USERNAME}:${BROWSERSTACK_ACCESS_KEY}" \
  "https://test-management.browserstack.com/api/v2/projects/PR-1028/test-cases?id=<numeric_id>"
```

### 2. Run tracing-feature-flows (instawork repo)

Skill: `instawork/.cursor/skills/tracing-feature-flows/SKILL.md`

**Parallel shortcut — spawn all 5 layer subagents simultaneously** once surface discovery returns:

| Subagent | Mission |
|---|---|
| Surface discovery | Run first, alone (required by skill) |
| Action mapper | Routes, form handlers, API calls |
| Server tracer | Views, serializers, services, policies |
| Data model tracer | Models, state fields, lifecycle |
| Side-effect tracer | Signals, tasks, notifications, analytics |
| Test grounding scout | Existing tests, factories, feature files |

Also run all 4 gate reviewers (surface, layer, evidence, reuse) in parallel — they are independent.

### 3. Run test-plan-generation (instawork repo)

Skill: `instawork/.cursor/skills/test-plan-generation/SKILL.md`

**Corroboration shortcut:** to validate a BrowserStack test case rather than plan full coverage,
ask for exactly 1 scenario. The skill still runs all gate reviewers but scopes coverage to one
test case. State this explicitly: "generate 1 scenario only to corroborate TC-XXXX."

Run multiple gate reviewers in parallel (input boundary + source facts, risk + technique +
coverage, test-case quality + handoff) — they are independent.

### 4. Run mobile-automation-orchestrator Phase 1 (Spec)

Skill: `mobile/.cursor/skills/mobile-automation-orchestrator/SKILL.md`

Phase 1 runs in main chat (never in a subagent). Phases 2/3/4 are delegated via Task.

Before writing the spec JSON, read existing feature files in the relevant area
(e.g. `mobile/applicant-app/test/automation/features/applicant/gigs_booking/`) to use real
screen names. The spec must not invent names that conflict with existing steps.

Default mode is `supervised` (stops at each checkpoint). Override: `orchestration_mode: autopilot`.

## Gotchas

- **Parallel layer subagents save the most time.** Sequential spawning of 5 agents is the biggest
  avoidable latency in this pipeline.
- **Gate reviewers are also independent.** Multiple can run at once, cutting review loops.
- **BrowserStack "Cannot Automate" custom field is often stale.** Evaluate automateability from
  the trace independently; don't let this field block the pipeline.
- **tracing-feature-flows: surface discovery must run first, alone.** The skill explicitly forbids
  parent searches before surface discovery returns.
- **test-plan-generation: scenario coverage gate may block on first pass.** Fix: add a "Coverage
  decisions" table documenting each path type (covered / deferred / N/A) with rationale. The gate
  accepts explicit deferral.
- Each gate has at most 2 review cycles. If still failing after 2, record residual risk and
  continue — don't loop indefinitely.

## Related

- [BrowserStack Test Management API](../tooling/browserstack-test-management-api.md) — API details, URL→id mapping
- [Asana API on cloud agents](../tooling/asana-api-cloud-agents.md) — token loading
- [Mobile BrowserStack QA Docker](mobile-browserstack-qa-docker.md) — if running the test after implementation
