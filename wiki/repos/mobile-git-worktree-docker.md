---
title: mobile — git worktree + Docker gotcha
area: repos/mobile
updated: 2026-06-19-2
---

# mobile — git worktree + Docker gotcha

When running automation tests from a **git worktree** via `docker compose`, two Python files
crash at import time because `git rev-parse --show-toplevel` cannot resolve the worktree `.git`
file inside the container.

## Root cause

A git worktree has a `.git` **file** (not directory) with content like:

```
gitdir: /Users/.../mobile/.git/worktrees/<worktree-name>
```

Docker mounts the worktree directory as `/workspace`. Inside the container, the absolute host
path in `.git` doesn't exist, so `git rev-parse --show-toplevel` exits 128.

## Files affected

Both files call `subprocess.check_output(['git', 'rev-parse', '--show-toplevel'])` at module
load (before any try/except):

- `applicant-app/test/automation/features/applicant/environment.py` (line ~23)
- `applicant-app/test/automation/features/applicant/steps/login_steps.py` (line ~5)

## Fix

Wrap the call in `try/except subprocess.CalledProcessError` and fall back to computing the
project root relative to the file:

```python
import os
import subprocess
import sys

try:
    top_level_path = subprocess.check_output(
        ['git', 'rev-parse', '--show-toplevel'], universal_newlines=True
    ).strip()
except subprocess.CalledProcessError:
    # <file> lives at: <root>/<N levels deep>/<file>.py — go up N levels.
    top_level_path = os.path.abspath(
        os.path.join(os.path.dirname(os.path.abspath(__file__)), *(['..'] * N))
    )
if top_level_path not in sys.path:
    sys.path.insert(0, top_level_path)
```

Depth for each file:
- `environment.py` (in `applicant-app/test/automation/features/applicant/`): N = 5
- `login_steps.py` (in `applicant-app/test/automation/features/applicant/steps/`): N = 6

The git stderr output `fatal: not a git repository: (null)` still prints (subprocess stderr is
not captured), but it is harmless — the exception is caught and the fallback runs.

## Secondary blocker: stale Docker image

The Docker image built before `instatest.core.helpers.allure_hook_steps` was added will raise
`ModuleNotFoundError` at `environment.py` line ~37. This was masked by the git crash above.

**Workaround (no rebuild):** Mount the newer instatest from the main repo via an extra
docker-compose volume, because `PYTHONPATH=/workspace/src/instatest:/opt/src/instatest:...`
checks `/workspace/src/instatest` first:

```yaml
volumes:
  - .:/workspace
  - /path/to/mobile/src/instatest:/workspace/src/instatest:ro
```

**Permanent fix:** Rebuild the Docker image (requires SSH access to GitHub for the private
instatest dependency):

```bash
eval "$(ssh-agent -s)" && ssh-add ~/.ssh/id_rsa
docker compose -f docker-compose_qa.yml build --no-cache
```

## PR hygiene (when using a worktree)

Worktrees are used occasionally on request. When they are, this fix is scaffolding
to unblock local BrowserStack validation — it does not belong in the PR.
Revert before pushing:

```bash
git revert --no-commit <fix-commit-1> <fix-commit-2>
git commit -m "revert: remove worktree-only Docker sys.path fixes"
```

## Related

- [mobile-android-login-local](../workflows/mobile-android-login-local.md)
