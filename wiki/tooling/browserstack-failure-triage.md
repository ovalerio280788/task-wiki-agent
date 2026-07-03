---
title: BrowserStack failure triage
updated: 2026-06-30
tags: [tooling, mobile, workflow, gotcha]
area: tooling
---

## Context

Building brick for debugging App Automate failures. Step 1 is BrowserStack-only grounding.
Step 2 (after) is test-code grounding. API access: [browserstack-app-automate-api-cloud-agents](browserstack-app-automate-api-cloud-agents.md).

## Step 1: Session grounding (BrowserStack only)

Do not read test code yet. Use session `name` only to identify which test ran.

### Session JSON — read carefully

| Field | Use for diagnosis? | Meaning |
|-------|-------------------|---------|
| `status` | No | Client-marked (`passed`/`failed`) via REST API or executor |
| `reason` | **No** | Free-text label the client attached when marking failed. Not BS root cause |
| `browserstack_status` | Yes | Infra state: `done`, `running`, `timeout`, `error` |
| `name` | Yes | Test/session identifier for later code lookup |

### Source priority

1. **Text logs** (`/builds/{id}/sessions/{id}/logs`) — grep `RESPONSE` lines with `"error"`; note timestamps
2. **Screenshots** — grep text logs for `browserstack-debug-screenshots` URLs; correlate by timestamp (needs `browserstack.debug` cap)
3. **Appium logs** (`/appiumlogs`) — server-side stack traces
4. **Device logs** (`/devicelogs`) — `FATAL`, `AndroidRuntime`, ANR
5. **Network logs** (`/networklogs`) — HAR; filter `response.status >= 400`
6. **Crash logs** (`/crashlogs`) — 404 means no native crash

### Step 1 output (keep brief)

- Last Appium error + timestamp from text logs
- What the user likely saw (from screenshots at that timestamp)
- Infra vs app: `browserstack_status` + device/crash/network findings
- Hypothesis in plain language (no test code yet)

## Step 2: Test code grounding (after step 1)

Only after step 1 is written. Use `session.name` (often includes scenario id and platform) to find:

- Feature file / scenario in `mobile/` or `test-automation/`
- Step definitions for elements and flows seen in logs
- Page objects for locators that failed (`help-button`, xpath in text logs, etc.)

Reconcile BS evidence with code: which step was running, what it expected, does the screenshot match that expectation.

## Gotchas

- **`reason` is not the failure explanation.** Never narrate screenshots from `reason` alone.
- **Two statuses:** client `status` vs `browserstack_status` measure different things.
- **Last screenshot ≠ failure moment.** Match screenshot timestamps to text log errors.
- **No crash ≠ no failure.** Element errors and UI hangs won't appear in `/crashlogs`.
- **RN-upgrade locator breakage signature.** When a step asserts an element "is displayed" but the
  failure screenshot clearly shows that element rendered on screen, suspect a brittle positional
  locator (deep xpath indexed by sibling position, no testID/accessibility id) that no longer
  matches after a React Native upgrade shifts the native view tree. Confirm by comparing the failing
  locator's xpath against a sibling locator on the same screen that still passes — if they share the
  same structural pattern but diverge only in trailing index numbers, that's strong evidence of a
  tree-shape shift rather than a real product regression, even if the test framework's own
  classifier (e.g. Allure) bucketed it as a product defect.

## Related

- [BrowserStack App Automate API on cloud agents](browserstack-app-automate-api-cloud-agents.md) — auth, endpoints, curl examples.
