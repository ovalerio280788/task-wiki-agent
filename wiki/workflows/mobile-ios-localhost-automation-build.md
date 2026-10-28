---
title: Mobile iOS automation app build against localhost
updated: 2026-10-22
tags: [testing, mobile, local, runbook, command, gotcha]
area: workflows
---

Build and run the applicant iOS automation app so local automation can launch it and point to the
host backend on `localhost:8080`.

## Context

Use this when local iOS automation fails before scenario execution and you need a clean simulator
`.app` artifact for the Instawork e2e runner at:

`/Users/ovalerio/workspace/be_instawork/instawork/e2e/mobile/applicant-app/bin/applicant-automation.app`

## What works

1) Ensure local e2e env values are set:

```bash
# instawork/e2e/mobile/.env
INSTAWORK_URL_OVERRIDE=http://localhost:8080
DEVICE_LOCAL_IOS_NAME="iPhone 16"
DEVICE_LOCAL_IOS_VERSION="18.0"
```

2) Install iOS pods from the correct directory:

```bash
cd "/Users/ovalerio/workspace/mb_instawork/mobile/applicant-app/ios"
bundle exec pod install
```

3) Build the simulator automation app:

```bash
cd "/Users/ovalerio/workspace/mb_instawork/mobile/applicant-app"
rm -rf build/derived_data

xcodebuild \
  -workspace ios/applicant.xcworkspace \
  -scheme applicant_automation \
  -configuration Automation \
  -sdk iphonesimulator \
  -destination "platform=iOS Simulator,name=iPhone 16,OS=18.0" \
  -derivedDataPath build/derived_data \
  clean build
```

4) Copy the built app to the e2e bin path used by the run task:

```bash
rm -rf "/Users/ovalerio/workspace/be_instawork/instawork/e2e/mobile/applicant-app/bin/applicant-automation.app"
cp -R \
  "/Users/ovalerio/workspace/mb_instawork/mobile/applicant-app/build/derived_data/Build/Products/Automation-iphonesimulator/applicant.app" \
  "/Users/ovalerio/workspace/be_instawork/instawork/e2e/mobile/applicant-app/bin/applicant-automation.app"
```

5) Keep Appium on major version 2, with an Appium-2-compatible iOS driver:

```bash
appium --version
appium driver list --installed
```

Known good combo:
- `appium` `2.12.0`
- `xcuitest` `9.10.5`

6) Xcode run configuration that worked for local backend testing:

- Open `/Users/ovalerio/workspace/mb_instawork/mobile/applicant-app/ios/applicant.xcworkspace`.
- Select scheme `applicant_automation`.
- Select an iOS simulator device (recent working setup: `iPhone 17 (iOS 26.0)`).
- In the scheme run arguments, keep or add:
  `-instaworkUrl http://localhost:8080`
- Run from Xcode.

7) Clean build folder from Xcode when switching branches, pods, or simulator targets:

- In Xcode, open the `Product` menu.
- Hold `Option` to reveal `Clean Build Folder...`.
- Run `Clean Build Folder...`, then build again.
- If issues persist, clear local derived data path and rebuild:

```bash
cd "/Users/ovalerio/workspace/mb_instawork/mobile/applicant-app"
rm -rf build/derived_data
```

## Business app variant

Same recipe from `mobile/business-app`: workspace `ios/business.xcworkspace`, scheme
`business_automation`, configuration `Automation`. Product lands at
`build/derived_data/Build/Products/Automation-iphonesimulator/business.app`; copy it to
`instawork/e2e/mobile/business-app/bin/business-automation.app`. The fastlane `ios build
simulator:true` lane needs match signing secrets, so plain `xcodebuild` is the local path.
Bundle id is `com.instawork.business.automation` (dots, per the Fastfile `BUNDLE_IDS`).

## Device beta IPA

For a signed device build (TestFlight-equivalent, backend staging2) run `yarn build:ios:beta` from
the app directory (`applicant-app` or `business-app`). The lane pulls ad-hoc profiles via match
from the secrets repo, so it needs working GitHub SSH, the App Store API key json in `fastlane/`,
`bundle check` green, Pods in sync, and `node_modules` present. Output is
`bin/<app>-beta.ipa`, versioned `<package version>-<user>-<sha>`; ~12 min on a warm machine. Tail
the log for `Successfully exported and signed the ipa file` rather than relying on the fastlane
update nag at the end.

## Gotchas

- Before building, check `node_modules` and Pods are current for the checked-out branch:
  a missing `node_modules/@amplitude/.../*-Bridging-Header.h` means run `yarn` first; a missing
  `Pods/hermes-engine/destroot/.../hermesvm.xcframework` means rerun `pod install`. `pod install`
  can need `--repo-update` after a Datadog bump; jsDelivr CDN errors during that update are
  non-blocking when `Podfile.lock` still matches `Pods/Manifest.lock`.
- Prebuilt RN core: the `[RNCore] Replace React Native Core` script phase fails with
  `tar: .../ReactNativeCore-artifacts/reactnative-core-<ver>-{debug,release}.tar.gz: No such file`
  when Pods only has the previous RN version's tarballs. `pod install` does not refetch them. The
  `<ver>` is the `-r` argument in that Pods script phase. Download both from Maven into
  `Pods/ReactNativeCore-artifacts/`:
  `https://repo1.maven.org/maven2/com/facebook/react/react-native-artifacts/<ver>/react-native-artifacts-<ver>-reactnative-core-{debug,release}.tar.gz`
  and name them `reactnative-core-<ver>-{debug,release}.tar.gz`. Hermes is the same failure in
  `[Hermes] Replace Hermes`: version is that script's `-r` value, not the React Native version.
  Maven path `com/facebook/hermes/hermes-ios/<ver>/hermes-ios-<ver>-hermes-ios-{debug,release}.tar.gz`,
  saved as `Pods/hermes-engine-artifacts/hermes-ios-<ver>-{debug,release}.tar.gz`. Either replace
  script deletes the extracted engine when `Pods/.last_build_configuration` (Hermes) or
  `React-Core-prebuilt/.last_build_configuration` is not the configuration being built.
- `xcodebuild ... | rg "error:"` misses script-phase failures. Grep for
  `The following build commands failed` and read the `PhaseScriptExecution` block above it.
- `bundle exec pod install` from `applicant-app/` root fails with:
  `No Podfile found`. Run it from `applicant-app/ios`.
- `xcodebuild` failure on `CpHeader ... RCTAppDependencyProvider.h` indicates stale/missing generated
  iOS build artifacts. Re-run pod install in `ios/` and rebuild.
- `PhaseScriptExecution Bundle React Native code and images` can be a wrapper error only. Inspect the
  full build log for the first `TransformError`/Babel parse error in app code (for example a stray
  plain-text line accidentally pasted into a `.tsx` file), fix that source error first, then rebuild.
- `Pod install complete` plus warnings about OpenTelemetry license file or CocoaPods master specs
  repo are non-blocking for this flow.
- Localhost routing for automation is provided at app launch time by runtime argument
  (`-instaworkUrl` from `INSTAWORK_URL_OVERRIDE`), not only by build flavor.
- The `applicant_automation` scheme runs the `Automation` build configuration, which does not define
  `DEBUG=1`. `AppDelegate` therefore loads the embedded `main.jsbundle` and ignores Metro. JavaScript
  changes require an incremental app rebuild. Switching the scheme to `Debug` connects to Metro, but
  also changes the bundle identifier and config file, so it is not a drop-in automation artifact.
- The container-only host alias used as the harness backend URL does not resolve on the Mac, so the
  iOS simulator (which shares the host network stack) cannot reach it either, and the app's ATS
  exception list usually covers only loopback names. Authenticated deep links embed the harness
  backend URL as a query param that the app treats as a backend override, so on iOS that param can
  repoint the app at an unreachable host. Android is insulated twice over: preflight sets up a
  reverse port forward for the loopback URL, and the deep link is delivered to the already-running
  app by intent, so the override is never re-read. Symptom to recognise: server-driven screens show
  the generic error view while the backend log shows no request at all and the device log shows no
  socket — an ATS refusal never reaches the network.
- Verify what the app is really pointed at before theorising: the simulator app process command line
  shows the launch argument, the app's key-value store file shows the persisted override, and the
  backend access log shows whether any request arrived.
- A killed containerised run leaves its Appium session alive on the simulator. Check the server's
  session list and delete stale ones; two concurrent sessions with different launch arguments on one
  simulator make results non-deterministic. Same for two booted simulators sharing a device name
  across runtimes.
- A Debug build fails during startup with `ConfigLocal.get is not a function` when
  `applicant-app/src/config/config.local.ts` has the template's `get()` export commented out. Restore
  an empty `export const get = (): Partial<Config> => ({});` before using Metro.

## Related

- [mobile-android-login-local](./mobile-android-login-local.md)
- [mobile-browserstack-qa-docker](./mobile-browserstack-qa-docker.md)
