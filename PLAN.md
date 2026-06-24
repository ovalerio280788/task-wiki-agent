# AI Oscar Profile Plan

## Goal

Build `ai_oscar_profile` as a personal AI operating manual that works for:
- Unattended Cursor Cloud Agents
- Local attended Cursor sessions

This repo is a routing and memory layer. It must not duplicate canonical rules from other repos.

## Principles

- Keep `AGENTS.md` short and directive.
- Link to canonical rules in each target repo instead of copying them.
- Separate stable guidance from time-sensitive context.
- Never store secrets, tokens, or `.env` values.
- Date volatile notes and expire stale context.

## Target Structure

```text
ai_oscar_profile/
  AGENTS.md
  README.md

  profile/
    preferences.md
    guardrails.md
    communication.md

  workspace/
    repos.md
    tools.md
    environments.md

  playbooks/
    cloud-agent-bootstrap.md
    mobile-test-triage.md
    backend-debugging.md
    pr-review.md

  task-templates/
    cloud-agent-task.md
    bug-investigation.md
    test-fix.md
    pr-review.md

  memory/
    gotchas.md
    lessons-learned.md
    glossary.md

  decisions/
    YYYY-MM-DD-topic.md

  context/
    YYYY-MM-current-focus.md

  .cursor/
    rules/
      oscar-profile.mdc
    skills/
```

## AGENTS.md Scope

`AGENTS.md` is the entry point for agents and should stay around 80-120 lines.

It should include:
- Purpose of this repo
- Read order for files
- Routing table by task type (backend, mobile, automation, config)
- Cloud vs local expectations
- Freshness policy for context files
- Output/reporting expectations

It should not include:
- Long tutorials
- Duplicated repo-specific rules
- Sensitive data

## File Responsibilities

- `profile/*`: Stable user preferences and boundaries.
- `workspace/*`: Repo map, tooling map, environment assumptions.
- `playbooks/*`: Repeatable cross-repo workflows.
- `task-templates/*`: Prompt templates for unattended runs.
- `memory/*`: Durable lessons and recurring issues.
- `decisions/*`: ADR-style durable decisions.
- `context/*`: Time-sensitive priorities and focus areas.

## Integration Approach

- Local attended: Add `ai_oscar_profile` to multi-root workspace.
- Cloud unattended: In task prompt, require reading `AGENTS.md` first.
- Repo-specific tasks: Always defer to canonical rules in target repo.

## Risks and Mitigations

- Drift from duplicated rules
  - Mitigation: Only link to canonical rules in source repos.
- Stale context driving wrong actions
  - Mitigation: Date files and add freshness checks.
- Entry point bloat
  - Mitigation: Keep `AGENTS.md` as a router, not a handbook.
- Secret leakage
  - Mitigation: Add strict no-secret policy in `guardrails.md`.

## Implementation Phases

### Phase 1: Bootstrap
- Create `AGENTS.md`
- Create `workspace/repos.md`
- Create `profile/guardrails.md`
- Create `playbooks/cloud-agent-bootstrap.md`

### Phase 2: High-ROI Playbooks
- Add `playbooks/mobile-test-triage.md`
- Add `playbooks/backend-debugging.md`
- Add `playbooks/pr-review.md`

### Phase 3: Templates and Memory
- Add `task-templates/*`
- Add `memory/gotchas.md`
- Add `memory/lessons-learned.md`
- Add `memory/glossary.md`

### Phase 4: Maintenance Controls
- Add `context/YYYY-MM-current-focus.md`
- Add `decisions` template and first entries
- Add `.cursor/rules/oscar-profile.mdc` with pointer to `AGENTS.md`

## Done Criteria

- Agents can route to correct repo and docs without manual correction.
- Cloud task prompts can stay short because `AGENTS.md` bootstraps context.
- No duplicated canonical rules from other repos.
- No secrets or private values in this repo.
- Playbooks are actionable and validated in at least one real task each.
