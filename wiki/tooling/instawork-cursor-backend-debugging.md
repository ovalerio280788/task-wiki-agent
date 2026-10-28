---
title: Debug the Instawork backend from Cursor
updated: 2026-10-15
tags: [backend, local, docker, config]
area: tooling
---

## Context

The Instawork Docker setup already exposes debug ports and wraps Django and pytest with `debugpy`
when `DEBUG_VSCODE=1`. The source is mounted at `/app` in the container.

`.vscode/launch.json` already exists and is gitignored, so a normal search misses it. Its current
configurations attach only to mobile automation containers on ports `15678`–`15685`. Add the Django
configurations below to that file.

## What works

Add these entries to the existing `.vscode/launch.json` `configurations` array:

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Attach Django backend",
      "type": "debugpy",
      "request": "attach",
      "connect": { "host": "localhost", "port": 5678 },
      "pathMappings": [
        {
          "localRoot": "${workspaceFolder}",
          "remoteRoot": "/app"
        }
      ],
      "django": true,
      "justMyCode": true
    },
    {
      "name": "Attach backend pytest",
      "type": "debugpy",
      "request": "attach",
      "connect": { "host": "localhost", "port": 5700 },
      "pathMappings": [
        {
          "localRoot": "${workspaceFolder}",
          "remoteRoot": "/app"
        }
      ],
      "justMyCode": true
    }
  ]
}
```

For request debugging, recreate the backend with debug mode and one stable process:

```bash
DEBUG_VSCODE=1 DISABLE_WATCH=1 docker compose up -d --force-recreate backend
```

Set a breakpoint, choose **Attach Django backend** in Cursor's Run and Debug panel, and trigger the
request.

For a test, start it first; it waits for the debugger on port 5700:

```bash
DEBUG_VSCODE=1 just pytest -n0 path/to/test_file.py
```

Then choose **Attach backend pytest**.

## Gotchas

- A hollow breakpoint, or `unable to find translation` listing only `e2e/mobile` paths, means the
  mobile automation session is the one attached. Stop that session and attach the Django
  configuration. A mobile run pointed at a demo host never executes the local Django process.
- Disable Django auto-reload while debugging requests; otherwise the executing child process can
  differ from the attached process.
- Django requests use port 5678; pytest and Kafka consumer debugging use port 5700.
- Celery defaults to port 5678 too, so do not debug Django and Celery on that port simultaneously
  unless one is assigned a different `DEBUG_PORT`.
