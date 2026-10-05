---
title: Discard a Google Play Console release before it goes live
updated: 2026-08-31
tags: [mobile, prod, runbook, gotcha]
area: workflows
---

Stop an approved or in-review Play Store update from publishing, then discard the track release
so a new build can be submitted.

## Context

Use this when a production (or other track) release is **in review** or **ready to publish** under
managed publishing, and it must not go live. Removing it from the publish queue is not enough: the
release still exists on the track until discarded.

Does not apply to a version already served to users. That cannot be deleted; halt a staged
rollout or ship a higher `versionCode`.

Official steps: [Prepare and roll out a release](https://support.google.com/googleplay/android-developer/answer/9859348)
(discard by release state) and [Control when app changes are reviewed and published](https://support.google.com/googleplay/android-developer/answer/9859654).

## What works

1. Confirm **Managed publishing** is on. Publishing overview shows a badge like "Managed publishing
   on".
2. Open **Publishing overview** (left nav, above Release). Do not start from the Production
   dashboard.
3. If the update is under **Changes in review** or **Changes ready to publish**, click **Remove
   changes**. Confirm. Those items move to **Changes not yet sent for review**. They are not live.
4. Open the track (**Production**, open testing, etc.). Click the **version** under Latest release,
   not **Create new release**.
5. On that release summary, click **Discard release** (typically top right).
6. Then **Create new release**. A new AAB needs a **`versionCode` higher than the discarded one**.
   The old code cannot be reused.

Draft or "ready to send for review" releases can be discarded on the release summary without the
Publishing overview remove step. In review or ready to publish: remove changes first, then discard.

## Gotchas

- **Production dashboard is not discard.** Latest release, install counts, and **Create new
  release** stay visible after you unschedule publish. Policy/API warnings on that page are
  unrelated. Open the version, then discard.
- **Remove changes does not delete the release.** It only unschedules review/publish. Create new
  release stays blocked until the outstanding release is discarded.
- **Only the latest release on a track can be discarded.** If several are queued, discard newest
  first.
- **Already live:** halt a staged rollout if it is still rolling out. Users who already received
  the build keep it. Next ship needs a higher `versionCode`.
- **Do not click Publish** on Publishing overview if the unwanted build is still listed as ready
  to publish.

## Related

- [app-store-connect-unsubmit-build](app-store-connect-unsubmit-build.md)
