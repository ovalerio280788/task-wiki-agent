# Session Close

Use this at the end of a work session to keep `ai_oscar_profile` current.

## Checklist

1. Append one concise entry to `memory/lessons-learned.md`.
2. If there was a failure or flaky behavior, append one concise entry to `memory/gotchas.md`.
3. Append a short status update to `context/2026-06-current-focus.md`.
4. If a durable decision was made, add `decisions/YYYY-MM-DD-topic.md`.
5. Keep all entries factual, short, and free of secrets.
6. Apply memory dedup:
   - Skip if an existing entry already covers the same point.
   - Improve existing entry when the new version is clearer or more useful.
   - Add a new entry only for materially new context, root cause, or fix.

## Task-Type Notes

- Automation maintenance: prioritize `memory/gotchas.md`.
- Creating new skills: prioritize `memory/lessons-learned.md`.
- Reviewing automation runs: update `context/` and add recurring failures to `gotchas.md`.
