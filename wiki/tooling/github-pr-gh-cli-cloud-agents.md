---
title: GitHub PRs via gh CLI when the PR tool is unauthenticated
updated: 2026-06-23
tags: [tooling, ci, capability, gotcha, command]
area: tooling
---

## Context

On cloud agents the built-in PR management tool can fail to write, with errors like
`[unauthenticated] Error` or `PR URL must belong to the current repository`. The second one shows
up in this multi-repo workspace because the tool binds to a single "current repository" and rejects
PRs in the sibling repos. When that happens, the `gh` CLI is the reliable alternative for PR
operations across any repo in the workspace.

System guidance treats `gh` as read-only, but in this environment the active `gh` account holds a
token with the `repo` scope, so `gh` write operations (create PR, edit, comment) do work. Check
before assuming.

## What works

Confirm the active account and scopes first (look for `repo` in scopes; ignore any second,
inactive/invalid account in `hosts.yml`):

```bash
gh auth status
```

Create a PR in any repo (set `--repo` explicitly; default to draft unless asked otherwise):

```bash
gh pr create --repo Instawork/<repo> --base <base> --head <branch> --draft \
  --title "<title>" --body-file /tmp/pr_body.md
```

Edit title/body (use `--body-file` for multi-line bodies to avoid shell-escaping pain):

```bash
gh pr edit <number> --repo Instawork/<repo> --body-file /tmp/pr_body.md
```

Reply to a reviewer / post a PR comment:

```bash
gh pr comment <number> --repo Instawork/<repo> --body-file /tmp/reply.md
```

Read PR data for verification:

```bash
gh pr view <number> --repo Instawork/<repo> --json title,body,reviews,state
```

Verified 2026-06-23: created mobile#8538, edited test-automation#194 and mobile#8538 bodies, and
posted a review reply on test-automation#194, all via `gh` after the PR tool returned
`unauthenticated`.

## Gotchas

- **Try the PR tool first, fall back to `gh`.** Use the dedicated PR tool by default; switch to `gh`
  only when it errors with auth or repository-scoping problems.
- **`--repo` is required** for sibling repos; otherwise `gh` infers from the cwd's git remote.
- **Two accounts in `hosts.yml`:** one may be invalid/inactive. `gh auth status` shows which is
  `Active account: true`. Only that one's scopes matter.
- **Cross-repo PR cross-links:** `Instawork/<repo>#<number>` renders as a link from either repo, so
  companion PRs can reference each other.
- **Do not change git identity or force-push** just to make `gh` work; the configured user is fine.
- **Never paste tokens** (the masked token in `gh auth status` output) into chat, commits, or wiki.

## Related

- [Asana API on cloud agents](asana-api-cloud-agents.md) — same "fall back when the integration is
  not authenticated" pattern.
- [CircleCI API on cloud agents](circleci-api-cloud-agents.md) — REST fallback for CircleCI.
