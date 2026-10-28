---
title: Git blame stops on a phantom file add
updated: 2026-10-06
tags: [tooling, gotcha, command]
area: tooling
---

## Context

`git blame` and `git log -L` attribute every line of a file to one commit, so it looks like the whole file was created there. Use this when you need the real edit that introduced a line, especially a tag on a line that was rewritten later.

## What works

Check the suspect commit against its parent before trusting blame:

```sh
git diff <parent> <commit> -- path
git log -1 --name-status <commit> -- path
```

If the diff is empty and name-status says `A`, the file was already in the parent tree. Blame that parent, not HEAD:

```sh
git blame -L <line>,<line> <parent> -- path
```

`git blame --ignore-rev <commit>` does not cross this boundary. History walks that start at HEAD (`git log -L`, `git log -S` without `--full-history`) stop at the phantom add too. Walk `git log <parent> -- path` for earlier history.

If the goal is "when was this token added" and the current line was only rewritten (a neighboring token removed or reordered), blame names that rewrite. Blame the same line on the parent of the rewrite and repeat until the token appears in a commit that did not already have it.

## Gotchas

An empty diff versus the parent plus status `A` is the signature. The commit is a real ancestor; the file content did not change.

## Related

- [Shallow fetch unrelated histories](git-shallow-fetch-unrelated-histories.md)
