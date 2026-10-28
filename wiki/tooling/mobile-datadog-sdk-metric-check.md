---
title: Mobile Datadog SDK metric check
updated: 2026-10-06
tags: [testing, mobile, tooling, runbook]
area: tooling
---

## Context

Manual check that a mobile Datadog SDK bump still uploads RUM and custom performance metrics.
The dev note is usually one line. The upload path has gates that look like a failed SDK.

## What works

1. Use a `beta` or `release` build. A Metro debug build turns off automatic RUM (resources, taps, errors, native views). The Datadog `env` attribute is that build flavor.
2. Custom metrics are Datadog logs, message prefix `tti-` or `metric-`. Startup `tti-*` is sent only when the first screen finishes with no tap, background, offline, error, or 60s timeout. A cancelled startup report is dropped on purpose.
3. The client skips SDK init when the app-config `datadog` block is missing, or when session or resource-tracing sampling is `0`. Both rates must be non-zero.
4. A device id listed in `DATADOG_RUM_FULLY_TRACKED_SESSION_DEVICE_IDS` forces both rates to 100 for that `X-Device-Id`. Use this when the live sample rate would hide a single session.
5. Background the app, then wait a couple of minutes before judging the explorer. Filter RUM by application, app version, and `env`. Pass requires a session on the new SDK, a view, a first-party `instawork.com` resource, and one `tti-*` or `metric-*` log. After login, the session user id should match the account.
6. Staging is enough. Client token and application id come from shared settings, so staging and production land in the same RUM application. `env` is the build flavor (`beta`, `release`, `debug`), not the backend name. Whether a backend returns a config is that environment's Constance (`DATADOG_*_APP_RUM_ENABLED` and both sampling rates). Defaults are off and `0`.
7. Repeat on iOS and Android. Native SDK packages ship separately from the JS package.

## Gotchas

- Fatal crashes are no longer copied into Logs after the 3.0 migration. A missing crash log is expected.
- Resource volume can drop versus 2.x because traces follow the RUM session sample. Judge the sampled session, not the old percentage.
- The service catalog splits the mobile apps by bundle id / flavor (`beta`, `debug`, `automation`, or unsuffixed). There is no mobile service named staging. `staging2` shows up as backend hostnames the app calls.
- A beta build's version shows up as `env:beta`. iOS uses the unsuffixed bundle id; Android uses the `.beta` service. Filter by the store version.
- APM Traces does not contain these sessions. `env:beta` there has no matching services. Open RUM Explorer.

## Related

- [Sentry mobile RN upgrade filter](sentry-mobile-rn-upgrade-filter.md)
