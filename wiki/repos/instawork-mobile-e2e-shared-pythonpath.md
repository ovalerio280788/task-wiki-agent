---
title: instawork — mobile e2e `shared` PYTHONPATH
updated: 2026-09-04
tags: [testing, mobile, docker, local, gotcha]
area: repos/instawork
---

# instawork — mobile e2e `shared` PYTHONPATH

## Context

`e2e/mobile/shared/` is a sibling of `applicant-app/` and `business-app/`. Local Docker runs `cd` into the app dir, so `shared` is not on `sys.path` unless something adds the e2e/mobile root.

## What works

**CI** prepends the e2e/mobile root:

```bash
export PYTHONPATH="/root/project/e2e/mobile${PYTHONPATH:+:$PYTHONPATH}"
```

See `.circleci/scripts/e2e/mobile/run-instatest.sh`.

**Local Docker compose** (`e2e/mobile/docker-compose_qa.yml`) now matches CI:

```yaml
PYTHONPATH=/workspace:/workspace/lib/instatest
```

Older trees set only `/workspace/lib/instatest`; verify the service `environment` before assuming
`shared` is unimportable.

The suite compensates in `features/*/environment.py` by inserting the e2e/mobile root (`parents[3]`) into `sys.path`. That insert must run **before** any `from shared...` import.

Business `environment.py` still does the insert before other imports. Applicant `environment.py` had the insert moved below third-party imports (isort), so anything those imports pull in cannot yet see `shared`.

## Gotchas

- `ModuleNotFoundError: No module named 'shared'` on the VSCode docker debug launch is not a missing local setup step. CI has the path; local compose does not.
- Do not `from shared...` in modules imported at the top of applicant `environment.py` (for example `testrail.*`) unless `/workspace` is on `PYTHONPATH` or the sys.path insert has already run.
- Matching CI locally: set compose `PYTHONPATH` to `/workspace:/workspace/lib/instatest`.

## Related

- [mobile android login local](../workflows/mobile-android-login-local.md)
- [mobile vendored instatest](mobile-vendored-instatest.md)
