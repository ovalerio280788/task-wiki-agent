# Repo Map

Paths are resolved from:
- `workspace/repo-paths.local.json` (preferred, machine-specific)
- `workspace/repo-paths.json` (fallback)

Do not hardcode absolute paths in this file.

## backend

- Path key: `backend`
- Role: Backend application (Django, APIs, backend automation)
- Canonical rules: This repo's own `.cursor/rules/*`

## mobile

- Path key: `mobile`
- Role: Mobile applications and related automation workflows
- Canonical rules: This repo's own `.cursor/rules/*`
