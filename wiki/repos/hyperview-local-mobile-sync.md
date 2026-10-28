---
title: Hyperview local mobile sync
updated: 2026-10-22
tags: [mobile, local, command, capability]
area: repos/hyperview
---

## Context

Use this to test uncommitted Hyperview TypeScript changes in the local mobile app without
publishing a package or reinstalling mobile dependencies.

## What works

After the mobile repository dependencies have been installed, run this from the Hyperview
repository and leave it running:

```sh
yarn sync ../../mb_instawork/mobile
```

The watcher copies Hyperview `src/` changes into the installed mobile package and directs the
package to load the copied TypeScript source. Restart Metro with a cleared cache, then reload an
existing debug app. A native rebuild is not required for TypeScript-only changes.

## Gotchas

- Before debugging why a synced change "did not apply", prove which JS the running app holds.
  An automation/release build embeds `main.jsbundle` and never talks to Metro — no dev menu, so
  Cmd+D does nothing and reloading cannot help. Compare a marker string (a new testID, a new
  module path) between the bundle Metro serves and the one inside the installed app:

  ```sh
  curl -s "http://localhost:8081/index.bundle?platform=ios&dev=true&minify=false" -o /tmp/b.js
  grep -c "<marker>" /tmp/b.js
  grep -c "<marker>" "$(xcrun simctl get_app_container booted <bundle-id>)/main.jsbundle"
  ```

  Different counts mean the app is stale, not the sync. `xcrun simctl listapps booted` reveals
  which flavor is installed; an automation bundle id is the tell. The served bundle also shows
  the transpiled module, so you can read back the exact prop you changed.

- Current `master` is a Yarn 4 workspace (root `packageManager: yarn@4.x`, workspaces `hyperview`
  and `demo`; library sources under `hyperview/src`). A checkout whose `node_modules` predates that
  move fails jest with `Preset @react-native/jest-preset not found`. Run
  `yarn install --immutable` at the repo root (lockfile untouched), then run unit tests from the
  `hyperview/` workspace: `cd hyperview && yarn jest src/elements/<el>/index.test.tsx`.
  The Yarn Classic notes below apply only to older checkouts.
- Older checkouts ship a Yarn Classic (`yarn.lock` v1) lockfile and no `packageManager` field, so it
  inherits whatever corepack's **global** default Yarn is. Working in a Yarn 4 repo can flip that
  default, after which `yarn` here fails with
  `hyperview@workspace:.: This package doesn't seem to be present in your lockfile` — a command
  that worked for months breaks with no change to the checkout. Confirm with `yarn --version`
  against `ls ~/.cache/node/corepack/v1/yarn`: cached 1.x entries next to an active 4.x default is
  the signature. Switching Node versions does not help; corepack shims every version.
  This is a toolchain-resolution failure, not a corrupt install — do **not** delete `node_modules`.
  Do not run `yarn install` under Yarn 4; that would rewrite the lockfile.
  If `node_modules/.bin/ts-node` is already present, skip Yarn. The sync script
  always joins the dest path onto the Hyperview repo, so pass a **relative** dest
  or it will look for `hyperview/<absolute-path>/node_modules/hyperview`:

  ```sh
  cd /Users/ovalerio/workspace/task-wiki-agent/hyperview && ./node_modules/.bin/ts-node ./scripts/sync ../../mb_instawork/mobile
  ```

  Otherwise install with Yarn 1, then sync the same way:
  `npx --yes yarn@1.22.22 install`.
- A `developmentDebug` install can die at `:app:fullstoryDevelopmentDebug` with
  `java.io.IOException: No space left on device`. That is disk full (often <1GB free after
  CMake + `app/build`), not Fullstory config. Check `df -h`. Safe reclaim for a retry:
  `applicant-app/android/app/build`, `applicant-app/android/app/.cxx`, and
  `~/Library/Android/sdk/.temp`. Do not delete `node_modules`.
- `yarn run:android` (`developmentDebug`) is a **native** first install. Metro does not skip it.
  A CMake/ninja abort with `clang-19` frames under `sdk/ndk/28...` while compiling autolinked
  modules (`react_codegen_RNCSlider`, `rnpicker`, …) is an NDK crash, not an app JS error.
  There is no debug APK until this succeeds. Retry after deleting
  `applicant-app/android/app/.cxx`; do not delete `node_modules`. Do not switch to a second
  Metro — keep the existing `--reset-cache` server.
