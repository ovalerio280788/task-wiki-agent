---
title: Unsubmit or swap an App Store Connect build before it goes live
updated: 2026-09-14
tags: [mobile, prod, runbook, gotcha]
area: workflows
---

Stop an iOS App Store version from shipping, then attach a different binary. Apple does not
discard a marketing version the way Play Console discards a track release.

## Context

Use this when a version is waiting for review, in review, pending release, or already
**Developer Rejected**, and the attached build must not go live. Live **Ready for Distribution**
versions stay as they are.

Official: [Remove a submission from review](https://developer.apple.com/help/app-store-connect/manage-submissions-to-app-review/remove-a-submission-from-review/)
and [App and submission statuses](https://developer.apple.com/help/app-store-connect/reference/app-information/app-and-submission-statuses/).

Play Store analog: [play-console-discard-release](play-console-discard-release.md).

## What works

**If status is Waiting for Review, In Review, Pending Developer Release, or Pending Apple Release**

1. Open the iOS version in the sidebar (not the live Ready for Distribution version).
2. At the top banner, click **remove this version from review**, then **Remove**. Status becomes
   **Developer Rejected**. Review restarts from zero if you submit again.
3. Alternate path: sidebar **App Review** → open the submission → **Cancel Submission**. Same
   Developer Rejected outcome for the app version.

**If status is already Developer Rejected (or Prepare for Submission)**

You are already off the review queue. Do not click **Add for Review** until the new binary is
attached.

1. On that version page, scroll to **Build**.
2. Hover the selected build and click the red minus to remove it.
3. Choose a new build (higher `CFBundleVersion` / build number). The marketing version string can
   stay the same.
4. Save, then **Add for Review**.

## Gotchas

- **You cannot delete the version row.** 2.x.y in the sidebar stays. Swap the build, or later add
  a newer marketing version. Do not create a second iOS version just to "cancel" this one.
- **Developer Rejected is not Apple Rejected.** You pulled it; metadata and screenshots are
  editable again. **Rejected** means App Review refused it.
- **Ready for Review** can block the minus on the version page. Remove the item from **App
  Review** (hover the version, red minus / cancel submission) first, then change the build.
- **Do not click Add for Review** on a Developer Rejected page if you still need to swap the
  binary. That resubmits the current attached build.
- **Live version is untouched.** The green Ready for Distribution row remains what users have
  until you actually ship the replacement.
- **Pause exists only for a phased release.** The on-page radios can look disabled; open **Edit**,
  choose **Pause the phased release**, then **Save** (up to 30 days, then it resumes unless you
  ship a newer version). A version already at 100% cannot be paused; people who already installed
  keep it. To stop further installs you ship a new version (or remove from sale, which takes the
  whole app down). Manual App Store updates still get this version while paused.
- New IPA still needs a **higher build number** than any upload already in App Store Connect.
  Reusing a build number fails even if that binary never shipped.

## Related

- [play-console-discard-release](play-console-discard-release.md)
