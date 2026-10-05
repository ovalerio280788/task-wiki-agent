---
title: Integration branches
updated: 2026-08-31
tags: [tooling, ci, runbook]
area: workflows
---

## Context

Use an integration branch when several related PRs should land on `master` together. The integration PR is only a collector: it does not carry unique feature work. Other PRs are the source of truth; the integration branch only merges them.

## What works

**Branch.** Cut from current `master` as `integration/<topic>` (example: `integration/react-native-update`). Open one PR: `integration/<topic>` → `master`.

**What may land.** Only commits that come from merging other PRs into the integration branch. Merge the source PR (or its head branch) into `integration/<topic>`. Do not commit unique work onto the integration branch, and do not cherry-pick or push patches that never existed as their own PR.

**Stacked work.** Source PRs can retarget their base from `master` to the integration branch while they wait. The integration PR still grows only by merging those PRs, not by duplicating their commits by hand.

**Label.** Add the `integration` label on the integration PR.

**Description.** Keep it short. After any Asana links, use this shape so GitHub autolinks the PRs (`#` plus the number):

```markdown
Integrate the following PRs / commits:

- #8509
- #8519 short note if the title is not enough
- 14952dcb3cdb98449b0e11937379ae253490ad2f only if a needed commit has no PR
```

Title the integration PR like any other conventional-commit PR (what the combined landing does), not the branch slug.

## Gotchas

- Cherry-picking SHAs onto the integration branch to "move" work out of another PR is not the same as merging that PR. Prefer merge so history and review stay attached to the source PRs.
- Listing PRs as full URLs without `#1234` does not give the same in-GitHub autolink sidebar as `#` references.
- Missing the `integration` label makes the PR look like a normal feature PR even when the description lists many PRs.

## Related

- [github-pr-gh-cli-cloud-agents](../tooling/github-pr-gh-cli-cloud-agents.md)
