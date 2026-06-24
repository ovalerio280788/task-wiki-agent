# Tools and Commands

Paths are resolved from `workspace/repo-paths.local.json` (preferred) or `workspace/repo-paths.json` (fallback).

## backend

- Primary stack: Python, Django, pytest
- Primary test command: `just pytest`
- Re-run command: `just pytest --reuse-db <target>`
- Notes: Follow canonical rules in the backend repo under `.cursor/rules/`.

## mobile

- Primary stack: React Native, TypeScript, Jest
- Unit tests: `yarn test:unit`
- Render tests: `yarn test:render`
- Targeted tests: `jest <path-to-test-file>`
- Notes: Follow canonical rules in the mobile repo under `.cursor/rules/`.
