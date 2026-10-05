---
title: Shallow fetch makes local master look unrelated
updated: 2026-09-08
tags: [tooling, git, gotcha]
area: tooling
---

## Context

`git merge master` (or rebase onto `master`) fails with `fatal: refusing to merge unrelated histories` even though the branch was cut from this repo's `master`.

## What works

1. Confirm the clone was shallow-poisoned:

```bash
git rev-parse --is-shallow-repository
git rev-list --count master
git merge-base HEAD master
```

If the repo is shallow, `master` has count `1`, and `merge-base` is empty, Git only has the tip of `master` as a root commit. The feature branch still has the full graph, so the two tips have no shared ancestor **in this clone**.

2. Restore history, then merge (or skip the merge if the remote branch already merged `master`):

```bash
git fetch --unshallow origin
git merge origin/master
```

Do **not** pass `--allow-unrelated-histories`. That would create a junk merge of two roots.

## Gotchas

- `git fetch origin master --depth=1` in a previously full clone writes `.git/shallow` and replaces `origin/master` with a single-commit island. Later `git checkout master` / `git merge master` uses that island.
- `git fetch` with no deepen does not undo this. `origin/master` can still say "up to date" while ancestry is one commit.
- Never use `--depth` against a default branch on a developer clone. Use it only in a throwaway directory.

## Related

- [github-pr-gh-cli-cloud-agents](github-pr-gh-cli-cloud-agents.md)
