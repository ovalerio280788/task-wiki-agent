# AGENTS.md

## Purpose

Use this repository as Oscar's AI operating manual across multiple codebases.
This repo provides routing, preferences, and reusable workflows.
Canonical implementation rules stay in each target repository.

## File Size Budget

`AGENTS.md` is a router, not a handbook.

- Target size: 80-120 lines.
- Hard limit: 150 lines.
- When adding guidance that may increase size, move details to `playbooks/`, `task-templates/`, or `workspace/` and add only a pointer here.
- Prefer replacing or tightening existing lines over appending new sections.

## Required Read Order

1. `profile/guardrails.md`
2. `workspace/repo-paths.local.json` if present, otherwise `workspace/repo-paths.json`
3. `workspace/repos.md` when available
4. Relevant file under `playbooks/` or `task-templates/` for the task
5. `context/` file only if it is fresh

If a required file is missing, proceed conservatively and call out the missing context.
If repo path values are empty, report the missing mapping instead of guessing paths.

## Task Routing

- Backend API or Django work
  - Use repo mapped as backend in `workspace/repo-paths.local.json` or `workspace/repo-paths.json`
  - Follow canonical rules in that repo
- Mobile app or React Native work
  - Use repo mapped as mobile in `workspace/repo-paths.local.json` or `workspace/repo-paths.json`
  - Follow canonical rules in that repo
- Mobile automation, Appium, or WebDriverIO work
  - Use automation repo mapped in `workspace/repo-paths.local.json` or `workspace/repo-paths.json`
  - Follow canonical rules in that repo
- Web UI tests
  - Use web test repo mapped in `workspace/repo-paths.local.json` or `workspace/repo-paths.json`
  - Follow canonical rules in that repo
- Config synchronization or tooling setup
  - Use config repo mapped in `workspace/repo-paths.local.json` or `workspace/repo-paths.json`
  - Follow canonical rules in that repo

## Repo Path Config

- Committed default path map: `workspace/repo-paths.json`
- Local override path map: `workspace/repo-paths.local.json` (gitignored)
- Starter example: `workspace/repo-paths.local.example.json`

## Cloud vs Local Behavior

- Unattended cloud agent
  - Start from this file and keep assumptions explicit
  - Prefer small, reversible changes
  - Report blockers with concrete evidence
- Local attended session
  - Keep updates short and action-focused
  - Ask for direction only when a real decision is required

## Guardrails

- Never store or expose secrets, tokens, or `.env` values in this repo.
- Never run destructive actions without explicit user request.
- Do not edit unrelated files.
- Do not commit or push unless explicitly asked.
- Prefer existing patterns over new abstractions.
- Validate changes before claiming completion.

## Freshness Policy

- Treat `context/*` as valid only if recently updated.
- If context appears stale, state assumptions and proceed with safe defaults.
- Record uncertainty explicitly instead of guessing.

## Memory Update Required

Before marking any task complete, append concise updates:

- `memory/gotchas.md` for failures, flakes, and recurring fixes
- `memory/lessons-learned.md` for reusable patterns
- `context/2026-06-current-focus.md` for short daily status updates
- `decisions/YYYY-MM-DD-topic.md` only when a durable process or architecture decision is made

Entry format should stay short:

- Date
- Repo key
- Task type
- Issue or goal
- Action taken
- Result
- Reuse note

## Memory Dedup Rule

Before writing to `memory/*`, check for an existing related entry.

- If the existing entry already covers the same point with no new value, skip writing.
- If the topic is the same but the new version is clearer or more useful, improve the existing entry.
- Add a new entry only when there is materially new context, root cause, or fix.
- Prefer fewer high-quality entries over many similar notes.

## Response Expectations

- Be concise and direct.
- Include what changed, why, and how it was validated.
- If validation is blocked, include exact blocker and next command to run.

## Invocation Style

- If a user message starts with `Oscar,`, treat it as a direct execution request.
- Do not ask unnecessary confirmation questions when scope is clear.
- If scope is ambiguous or risky, ask one concise clarifying question before proceeding.

## Source of Truth Rule

This repo defines Oscar-specific operating guidance.
Target repositories remain the source of truth for implementation details, test rules, and code conventions.
