---
title: Task definition readiness
updated: 2026-08-19
tags: [capability, runbook]
area: workflows
---

## Context

Use when drafting or reviewing an implementation task before it enters a sprint.

## Before drafting

Read the relevant implementation files before drafting: the target code, its configuration,
dependency manifests, command runner, and CI path. Then check that the task has enough evidence to
define:

1. The outcome and explicit exclusions.
2. The affected code, configuration, dependencies, and CI paths.
3. Any unresolved decision, its owner, and the facts needed to decide it.
4. The commands or checks that will verify the change.

Do not encode an unverified assumption as an implementation instruction. Record the decision as
the first action instead.

## Drafting

Use these sections only when they add information:

- **Goal:** one outcome and any important boundary.
- **Current state:** verified facts that explain why the work is needed.
- **Do:** ordered, executable work. This is the task's single source of truth. Do not add a
  `Done when` section unless it contains a distinct release gate that is not already an action.
- **Out of scope:** work that could otherwise be inferred but is intentionally excluded.
- **Estimate:** story points plus one sentence explaining the scope, risk, or uncertainty behind
  them. Point using [qa-platform-story-pointing](qa-platform-story-pointing.md): agent-executed, startup-speed,
  human decision/review/residual risk, keep scores low.

Avoid repeating `Do` as separate acceptance criteria, completion criteria, or coordination notes.

## Readiness check

The task is ready to start when:

- Every action in `Do` is concrete and testable.
- The first action resolves any material unknown before dependent work starts.
- The scope does not include unrelated cleanup or speculative migrations.
- Verification is part of the executable actions.
- The description contains no duplicated requirements.
