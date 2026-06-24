# Guardrails

- Never store or expose secrets, tokens, or `.env` values.
- Never run destructive actions unless explicitly requested.
- Do not commit or push unless explicitly requested.
- Do not edit unrelated files.
- Prefer existing repository patterns over new abstractions.
- Validate changes before claiming completion.
- If context is missing, state assumptions instead of guessing.
