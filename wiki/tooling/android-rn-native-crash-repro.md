---
title: Android RN native crash repro
updated: 2026-08-27
tags: [mobile, testing, gotcha]
area: tooling
---

## Context

Trying to reproduce a native (C++) React Native New Architecture / Fabric crash on a real or rented
Android device, starting from a Sentry native stack. These are teardown/use-after-free races, so a
correct tap sequence alone usually fails and the session looks like a false negative.

## What works

- **Force the teardown, do not hope for it.** The race needs the OS to actually destroy the surface.
  Turn on **Don't keep activities** so every activity is destroyed the moment the app leaves the
  foreground, and kill the process (`adb shell am force-stop <pkg>`) before any cold-start variant
  of the steps.

  Enable it with ADB when the session has a shell (fastest on a rented device):

  ```bash
  adb shell settings put global always_finish_activities 1
  adb shell settings get global always_finish_activities   # expect 1
  ```

  Or from the device UI (Samsung One UI, for example Galaxy S25):

  1. Settings → About phone → Software information → tap **Build number** seven times
     (skip if Developer options is already listed).
  2. Settings → Developer options.
  3. Under **Apps**, turn on **Don't keep activities**.

  Leave it on only for the repro session. It destroys every app's activity on Home, not just the
  one under test.

- **Keep amplifiers out of the user-facing repro steps.** A developer-options toggle is a lab
  instrument: it makes a real lifecycle transition deterministic, it does not invent a code path.
  Use it to prove the path is unsafe, then write the ticket's Steps to Reproduce as the plain user
  path and record the amplifier separately as "how to hit it reliably". Never derive frequency or
  severity from how easily the amplified setup crashes — that comes from the event and user counts.

- **Closest faithful alternative without dev options:** `adb shell am kill <pkg>` kills a
  backgrounded process the same way the low-memory killer does to real users, unlike `force-stop`,
  which also sets the stopped flag and is closer to a manual force-stop. Pair it with a low-RAM
  device and other apps open to create genuine memory pressure.
- **Treat device class as part of the repro.** Low-RAM and 32-bit ABI devices reclaim and tear down
  far more aggressively than a warm flagship. Match the ABI and OS major of the reporting device
  before concluding "not reproducible".
- **Measure the timing instead of eyeballing it.** The repro is the sub-second ordering (the
  background must land while the surface is still mounting), not the list of screens. Verify the gap
  from the screen recording, resampled around the moment of interest — see the video analysis steps
  in [asana-bug-report](../workflows/asana-bug-report.md).
- **Repeat.** A race needs many attempts under the same preconditions. One clean pass is not
  evidence that the path is safe.

## Gotchas

- **A logcat filter on the C++ symbol from the Sentry frame catches nothing.** Those symbols come
  from the tombstone, and Android emits native SIGSEGV output under the `libc` / `DEBUG` tags. Filter
  on `Fatal signal|libc|DEBUG` or run unfiltered; a quiet symbol-filtered pane proves nothing.
- **"The app survived" is a weak negative.** With a warm process, no forced activity destruction and
  a flagship device, the crash conditions were never created, so the attempt says nothing about the
  code path.
- Repro steps inferred from breadcrumbs give the screen order and rough gaps, never the exact
  preconditions (process state, memory pressure). Reconstruct the preconditions explicitly and state
  them as part of the steps.

## Related

- [sentry-mobile-rn-upgrade-filter](sentry-mobile-rn-upgrade-filter.md) — isolate RN-upgrade issues first
- [asana-bug-report](../workflows/asana-bug-report.md) — recording analysis and frame timing
