# Cloud Agent Bootstrap

## Steps

1. Read `AGENTS.md`.
2. Read `profile/guardrails.md`.
3. Resolve repo paths from `workspace/repo-paths.local.json` or `workspace/repo-paths.json`.
4. Read `workspace/repos.md` and `workspace/tools.md`.
5. Load only task-relevant playbooks or templates.

## Execution Defaults

- Make small, reversible changes.
- Validate with the smallest relevant command first.
- Report blockers with command output context.
- If a required mapping is missing, stop and report the missing key.
