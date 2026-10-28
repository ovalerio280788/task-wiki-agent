---
title: Sentry filters for mobile RN upgrade builds
updated: 2026-10-13
tags: [mobile, tooling, gotcha]
area: tooling
---

## Context

When triage-ing an RN upgrade integration branch (soup merges → many app versions / build numbers), pick a filter that stays constant across those builds.

Applicant Sentry project is DSN path `/1549809`; business is `/1549805`. Environments map to `BUILD_FLAVOR` (`automation`, `beta`, `debug`, `release`).

Release string shape (from `core-mobile` error-reports init):

`{bundleId}@{appVersion}+{buildNumber}`

Automation/beta also change `bundleId` via Android `applicationIdSuffix` (and iOS equivalents), so `release:` alone is a poor “this upgrade” key.

## What works

1. **Do not filter Issues by Contexts → React Native → React Native Version.** The SDK stores it under `contexts.react_native_context.react_native_version` (display-only). Sentry contexts are not searchable; only tags are. Queries like `react_native_version:0.86.2` return nothing.
2. **Best filter for this RN upgrade:** `sdk.version:8.21.0` (`@sentry/react-native` bumped 6.15.1 → 8.21.0 on the integration branch). Searchable built-in field (not a custom tag). Works across soup SHAs and marketing versions. Scope with `environment:beta` / `automation`.
3. **Also useful — CI `release` string:** beta/automation CI builds look like
   `com.instaworkmobile.beta@{appVersion}-ci-{shortSha}+{buildNumber}`
   - all CI soup builds: `environment:beta release:*-ci-*`
   - one SHA tip: `release:*ci-1a994edaf*`
4. **Do not use React Native Version from Contexts** — display-only under `contexts.react_native_context.react_native_version`.
5. **To make RN version filterable later:** `Sentry.setTag('react_native_version', …)`.
6. Agent access: Sentry MCP (`plugin-sentry-sentry`) must be authenticated; use issue URL with `get_sentry_resource`.
7. **Read per-issue tag distributions, not just the latest event.** `get_issue_tag_values` (via `execute_sentry_tool`) gives counts per tag value across the whole group — use it for `handled`, `release`, `environment` before drawing conclusions. Cross-check aggregates with `search_events` on `dataset=errors` and fields like `error.handled`.
8. **Before blaming the branch under test, prove the build came from it.** Take the short SHA from the CI `release` string and resolve it with `gh api repos/<org>/<repo>/commits/<sha>` plus `.../commits/<sha>/pulls`. A SHA missing from a local `git fetch origin` is often still on GitHub on a stacked/closed branch. Then `compare/<branch-under-test>...<sha>`: `ahead>0, behind=0` means soup **on top of** the branch, not a commit **in** it. Also check whether the crashing package exists in that branch's `package.json`.
9. **Prove store RN version from the app git tag, not a local checkout.** For applicant `X.Y.Z`, read `applicant-app/package.json` (and root `resolutions`) at `applicant-app/vX.Y.Z`. A workspace clone can still be on the pre-upgrade RN line.
10. **Do not expect `react_native_version` on native Android crashes.** Discover field `contexts.react_native_context.react_native_version` is empty, and issue-detail payloads often omit the React Native context card entirely when `event.origin` is `android` (Java uncaught handler). That version is JS-SDK context; if the UI card is missing, Sentry never received it. Use the app git tag for the release instead.
11. **After the store ship, isolate new issues with `firstRelease:`** the first Play/App Store string, e.g. `firstRelease:com.instaworkmobile@2.242.0+874` + `environment:release`. That is “never seen before this build,” not “caused by this build.” Still check disk, Firebase init, and other non-RN stacks before blaming New Arch.
12. **On a later store hotfix, do not treat `firstRelease:` of that hotfix as “where RN landed.”** RN 0.86.2 shipped on the first 2.242.x store build; a later 2.242.x can be 0.86.3 (or other app changes) with new ANR fingerprints that Sentry never grouped before. Always: (a) read `react-native` from `applicant-app/package.json` at each git tag, (b) compare `firstRelease:` of the *first* RN-ship build (Fabric `getViewState` / ShadowTree) against the hotfix, (c) also query `release:` of the hotfix **without** `firstRelease` — pre-existing Background ANR groups usually dominate volume.
13. **AppExitInfo ANRs with Chromium/WebView init, `syscall` wait, or an empty stack are not Fabric signatures.** A New-Arch ANR shows `FabricUIManager` / `MountItemDispatcher` / Reanimated `NodesManager` on the main thread. Confirm with `get_issue_tag_values` on `release` for the known Fabric issues, not only the hotfix’s new fingerprints.
14. **A later RN patch that kills one Fabric crash class does not close the others.** Query the hotfix `release:` for `error.type:SIGSEGV` (and known native titles) even when `firstRelease:` of the hotfix is mostly ANRs. The original New Arch issue id keeps accruing events; `firstRelease:` of the hotfix will not show it.
15. **`ShadowTreeRegistry::enumerate` SIGSEGV splits by caller.** RuntimeScheduler, `findNextFocusableElement`, and EventQueue flush become different issue ids. Sum by title / `error.type:SIGSEGV` on that release, not one fingerprint.
16. **The events-list Title is that event's top symbol, not the shared cause.** Aggregate `stack.function` on the issue before splitting reasons. `abort` (SIGABRT through `LogMessage` / `art::Runtime::Abort`) and a SIGSEGV titled `android::IdGenerator::IdGenerator` are one crash when every stack still contains `android::MotionEvent::getRawAxisValue`. That frame is the OS fatal check while reading a motion event. The stacks are system-only, so the screen in breadcrumbs is not the cause. The same function can also land in an older mixed `abort` issue; count the function on the release, not one issue id.

17. **iOS `EXC_BAD_ACCESS: _endSuspendingUpdates` with `UIPickerView hitTest` is one crash across many screens.** Compare the last tracking event plus `RCTFabricModalHostViewController` presented; do not split issues because the user was in artform vs education vs self-identification. The native stack is touch hit-test into picker cells (`objc_msgSend` → `_updateVisibleCellsNow:` → `UIPickerView hitTest:withEvent:`). Different Hyperview forms that open an RN Fabric picker modal share the same fingerprint.

18. **Derive the repro cadence by counting `ui.lifecycle` breadcrumbs, not by reading the stack.** For
   a UIKit use-after-free the stack names the victim, never the accumulating precondition. Pull
   breadcrumbs for 2–3 events and count repeats of the same `screen` with `beingPresented: true`,
   plus the gaps between them and the gap to the crash. A repeated present/dismiss cycle on one
   screen is a different repro from one long-lived presentation, and the difference decides whether
   a manual attempt can ever hit it.
19. **Bound the OS floor from the `os` tag distribution before calling a repro attempt failed.** A
   crash class that only exists above a given point release will never fire on an older device, and
   `get_issue_tag_values` on `os` gives that floor directly.
20. **Check whether internal build flavors appear in `environment` / `release` before designing a
   repro.** A crash present on `beta` / adhoc bundle ids is reproducible in-house; one confined to
   `release` on the store bundle id may depend on release-configuration builds.

21. **Before accepting "cannot reproduce", map the crashing `dist` back to a git tag and diff deps
   against the local checkout.** Two builds with the same marketing version can carry different RN
   patch releases, and the release string does not show it. Resolve `dist` → `applicant-app/vX.Y.Z`
   → `package.json`. A local branch cut one build earlier can sit on an RN patch that has no field
   exposure at all, which makes a manual repro attempt meaningless.
22. **A release with near-zero distribution is not a clean control.** Check `count()` and
   `count_unique(user)` for that release before reading "no events on it" as "not affected"; a build
   superseded within days can show a single-digit event count across the whole project.
23. **Join Amplitude product `userId` to Sentry `user.id`.** Same integer. Amplitude activity after
   the crash timestamp is a relaunch, not the dying process; cut the session at the Sentry
   `timestamp` and read breadcrumbs for the wheel. Amplitude has no picker-open event.

## Gotchas

- `sdk.version` is the Sentry SDK version, not React Native. On Android native crashes it is the Java SDK (`sentry.java.android.react-native`), which can be `8.50.1` while the JS package is `@sentry/react-native` `8.21.0`. `sdk.version:8.21.0` therefore misses Play Store events; use `sdk.version:8.*` or the app `release:` string for store triage.
- **Do not OR in `sdk.version:1.6.1`.** That is an old Sentry JS/native SDK on ancient App Store builds (e.g. `2.120.5`), not `@sentry/react-native` `8.21.0`. It pulls 6-year-old issues into the stream.
- Pre-upgrade store builds used `sdk.version:6.15.1`. After the RN 0.86 store ship, `environment:release` events are on the 8.x SDK. Do not treat 8.x as “beta only.”
- **`RetryableMountingLayerException` from `updateOverflowInset` → `getViewState` is RN Fabric, not app JS.** The exception name is not a retry: batch mount items are not retried and the crash is fatal. RN 0.86.2 still uses throwing `getViewState` on that path; the null-safe pick landed on `0.86-stable` after 0.86.2 (in 0.86.3). Confirm the yarn RN patch actually contains that method change before assuming the app already has it.
- **Logcat `ReactNoCrashSoftException: Unable to find viewState … for deleteView` is not that crash.** Same missing-tag race, different path: Fabric already swallows `deleteView` as a soft log (`SurfaceMountingManager:MissingViewState`). The fatal issue is `RetryableMountingLayerException` from `updateOverflowInset` with `Surface stopped: false`. Soft deleteView lines on beta do not prove PRO-APP-26ZT.
- `hermes:true` is a real tag but is true on old RN too — useless for isolating 0.86.2.
- Fabric / turbo_module in the React Native context card are also non-searchable.
- Long-lived issues (e.g. `PRO-APP-9CY`) mix many RN / release eras; always constrain by `sdk.version` or `release` + environment when triage-ing an upgrade branch.
- **`mechanism` is not the `handled` flag.** `onunhandledrejection` events are still reported with `handled: yes`; only the `handled` tag / `error.handled` field is authoritative. Never infer handled state from the mechanism name.
- Event/user counts shown under a filter are filter-scoped; issue **Age** is lifetime. Report both explicitly so a decades-old issue is not mistaken for new.
- A missing autolinked native module can lie dormant: RN CLI only autolinks packages declared in the app's own `package.json`, so a transitive native dependency is never linked. It only crashes once some code actually `require`s its JS entry point, which can be a package added much later.

## Related

- [browserstack-failure-triage](browserstack-failure-triage.md)
- [mobile-rn-upgrade-locator-defect-signature](mobile-rn-upgrade-locator-defect-signature.md)
