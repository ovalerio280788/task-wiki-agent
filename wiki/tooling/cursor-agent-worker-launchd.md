---
title: Cursor agent worker LaunchAgent setup and troubleshooting
updated: 2026-06-30
tags: [tooling, gotcha, config]
area: tooling
---

## Restart/reload gotcha

`launchctl bootout` and `launchctl bootstrap` back-to-back in one command (with `&` to avoid the
Node.js check-in hang) can race: bootstrap fires before bootout has fully unregistered the label,
failing with `Bootstrap failed: 5: Input/output error`. Fix: run bootout alone first, confirm with
`launchctl list | grep <label>` (should disappear), then run bootstrap as a separate call.

## Context

The `cursor agent worker start` command must run persistently at login so cloud agents can dispatch
to this machine. It is wired as a macOS LaunchAgent. This page covers the two root causes that
silently break it and the working plist configuration.

## What works

Production plist at `~/Library/LaunchAgents/com.cursor.agent-worker-root.plist`:

```xml
<key>ProgramArguments</key>
<array>
    <string>/Users/antonyfuentes/.local/bin/agent</string>
    <string>worker</string>
    <string>start</string>
    <string>--name</string>
    <string>antony-macbook</string>
    <string>--worker-dir</string>
    <string>/Users/antonyfuentes/Documents/workspace/instawork-all-repos</string>
    <string>--worker-dir</string>
    <string>/Users/antonyfuentes/Documents/workspace/instawork-all-repos/&lt;each-repo&gt;</string>
    <!-- one --worker-dir pair per repo, plus the root; see "Per-repo worker dirs" below -->
</array>
<key>RunAtLoad</key><true/>
<key>KeepAlive</key><true/>
<key>ThrottleInterval</key><integer>30</integer>
<key>StandardOutPath</key>
<string>/Users/antonyfuentes/Library/Logs/cursor-agent/worker.log</string>
<key>StandardErrorPath</key>
<string>/Users/antonyfuentes/Library/Logs/cursor-agent/worker.log</string>
<key>EnvironmentVariables</key>
<dict>
    <key>PATH</key>
    <string>/Users/antonyfuentes/.local/bin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin</string>
    <key>HOME</key>
    <string>/Users/antonyfuentes</string>
    <key>CURSOR_ACCESS_TOKEN</key>
    <string><token from: security find-generic-password -s cursor-access-token -w></string>
</dict>
```

Load it (run in background to avoid hanging):

```bash
launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/com.cursor.agent-worker-root.plist &
sleep 5
launchctl list | grep cursor
```

Verify running:

```bash
launchctl list | grep cursor   # should show a PID (not "-") with exit 0
tail -5 ~/Library/Logs/cursor-agent/worker.log
```

When the token changes (re-login to Cursor):

```bash
TOKEN=$(security find-generic-password -s "cursor-access-token" -w)
# Update CURSOR_ACCESS_TOKEN value in the plist, then:
launchctl bootout gui/$(id -u)/com.cursor.agent-worker-root
launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/com.cursor.agent-worker-root.plist &
```

## Per-repo worker dirs

Pass one `--worker-dir` per actual repo, not just the workspace root, so cloud agents can be
dispatched to any individual repo. Discover the repo list by finding `.git` dirs (top level plus
nested, e.g. a repo vendored inside another repo):

```bash
find /path/to/workspace -mindepth 2 -maxdepth 4 -type d -name ".git" -not -path "*/node_modules/*"
```

Keep the root `--worker-dir` too (cross-repo tasks still need it). Re-derive this list periodically;
new repos added to the workspace won't automatically get a worker-dir entry.

## Gotchas

**macOS TCC blocks launchd from writing to `~/Documents`.**
On macOS 15, background tasks (LaunchAgents) cannot write to the user's `~/Documents` folder
without explicit TCC consent. Pointing `StandardOutPath`/`StandardErrorPath` there causes the
agent to exit immediately with `EX_CONFIG` (code 78) before writing anything. The log file is
never created, leaving no trace of the failure.
Fix: use `~/Library/Logs/<app>/` instead — that path is always writable by background tasks.

**Keychain is inaccessible from launchd context.**
The agent reads `cursor-access-token` from the macOS Keychain at startup. In a launchd session
(no interactive Keychain prompt possible), Node.js Keychain API calls fail silently, causing
`EX_CONFIG` (code 78). The `security` CLI has the same restriction in this context.
Fix: embed `CURSOR_ACCESS_TOKEN` as an `EnvironmentVariables` entry in the plist. The token is
a long-lived JWT (expire ~1 year after issue). Refresh it from a terminal session when it changes.

**`launchctl bootstrap` / `launchctl load` hangs when the service uses Node.js.**
The agent (Node.js) registers Mach/XPC services at startup, which causes `launchctl bootstrap`
to wait indefinitely for the process to "check in." This is not a failure — the service IS
loading — but the calling shell blocks. Always background the bootstrap call:
`launchctl bootstrap ... &` and poll with `launchctl list` instead.

**Exit code 78 (`EX_CONFIG`) with no log output is the combined symptom of both TCC and Keychain
failures.** The service registers in `launchctl list` with `-` PID and exit 78, and no log is
ever written. This is the first thing to check when the worker stops auto-starting.

**Label poisoning after repeated crash loops.**
If a launchd label has crashed many times, it enters a throttled state that persists across
bootout/bootstrap cycles. Symptom: even after updating the plist, the label immediately shows
exit 78 with no log output and `runs` incrementing without any new log lines. Fix: use a new
label name (e.g. append `-v2`), load it cleanly, confirm it runs, then retire the old label.

## Related

- [github-pr-gh-cli-cloud-agents](github-pr-gh-cli-cloud-agents.md)
