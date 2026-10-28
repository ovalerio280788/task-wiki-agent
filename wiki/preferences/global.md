---
title: Global preferences
updated: 2026-10-27
scope: global
---

How the user wants me to behave everywhere, regardless of repo or task. Read this during required
preflight for every task. These are living: when the user states a new preference that conflicts
with one here, replace it and log the change.

- **Always:** reply concise, avoid filler and repetition. — added 2026-06-19
- **Do:** prefer simple solutions. **Instead of:** reaching for complex ones, unless the problem genuinely requires an advanced solution. — added 2026-06-19
- **Always:** understand existing code and architecture first to avoid duplicating logic (keep code DRY). — added 2026-06-19
- **Do:** be skeptical, double check, and research. **Instead of:** assuming the user or I am right. — added 2026-06-19
- **Never:** use concrete tool names, service names, or specific examples in agent rules, guidelines, or AGENTS.md. They introduce bias into future agent behavior. Keep rules general and principle-based. — added 2026-06-19
- **Do:** use the lightest wiki preflight that still protects the work: full preflight for substantive tasks, fast-path preflight for tiny one-step tasks, and full preflight if the task broadens or gets blocked. **Instead of:** paying full wiki overhead when no wiki knowledge can plausibly help. - updated 2026-07-01
- **Always:** before replying to any request, reason about whether the work surfaced something durable worth persisting, and if so capture it in the same turn without being asked. Persist the method or recognition pattern, not the incident symptom. Stay silent when nothing qualifies. — updated 2026-06-26
- **Do:** grow wiki and preferences autonomously when durable, reusable knowledge appears. **Instead of:** waiting for explicit requests to maintain them. — added 2026-06-19
- **Always:** on a tool or integration auth/access failure, consult the wiki for a documented fallback before improvising an alternative or declaring the capability unavailable. — added 2026-06-23
- **Always:** when a PR and its tracking ticket both exist, cross-link them at the very top of each description. The PR opens with a link to the ticket; the ticket opens with a link to the PR. — added 2026-06-29
- **Always:** create Asana tasks directly without showing a preview or asking for confirmation first. — added 2026-06-29
- **Never:** share Drive files with anyone or anyone-with-link access. **Do:** restrict sharing to people within the organization when sharing is required. — added 2026-07-01
- **Never:** create commits, push branches, merge changes, or change PR state without the user's explicit request. Implementing or editing files is not permission to commit; leave work in the working tree until they ask. — updated 2026-08-28
- **Never:** abbreviate file paths with ellipses in examples or output. **Do:** show the full verified path or `Unknown`. — added 2026-07-21
- **Never:** silence a linter with an inline suppression comment. **Do:** fix the underlying cause, even when that means a wider refactor; treat a needed suppression as evidence of a real structural problem. — added 2026-08-21
- **Never:** run tests before the user approves the final solution. — added 2026-07-31
- **Do:** run work in the primary agent. **Instead of:** delegating to a subagent unless explicitly requested. — added 2026-08-03
- **Always:** when filing product bugs in Asana, use the Bugs board and resolve the live New Bug Report template via API each time (never reuse a cached template body or GID). — added 2026-08-14
- **Do:** limit a locator/test fix to cases proven broken by a report or run, and let the next run surface the rest. **Instead of:** pre-emptively changing cases that only match the defect pattern by inference. — added 2026-09-11
- **Never:** use XPath selectors in mobile automation. **Do:** add or use stable accessibility identifiers; use native platform selectors only when OS controls cannot expose app-owned IDs. — added 2026-08-11
- **Always:** construct programmatic mobile id lookups with `IdSelector`. **Instead of:** `Selector.by_id`. `@element(id=)` stays for static locators. — added 2026-09-24
- **Do:** locate MDS/Hyperview controls with the full namespaced id (`{namespace}/…`). **Instead of:** `partial_id` / Contains on the suffix. Namespace is the view’s `namespace`, not a Cursor workspace. — added 2026-09-08
- **Always:** when pointing, assume the work is agent-executed and use a startup mindset: keep scores low; point human decision, review, and residual risk, not bulk mechanical edits or file count. — updated 2026-08-19
- **Always:** when running personal code review, hold a best practices mindset; passing CI or absence of bugs is not enough. — added 2026-08-25
- **Do:** keep an existing `try`/`except` around optional UI and narrow the exception type. **Instead of:** deleting the block because a helper already returns `None`. — added 2026-08-25
- **Do:** catch `WebDriverException` for driver lookup/stale/missing-element misses. **Instead of:** listing `ElementLookupException`, `NoSuchElementException`, or `StaleElementReferenceException`. — added 2026-08-25
- **Always:** format task-description section headers in bold. — added 2026-08-16
- **Always:** separate task-description sections with an empty line. — added 2026-08-16
- **Never:** send Slack messages unless the user explicitly asks. Filing a ticket, finishing a report, or closing a thread is not permission to post. — added 2026-08-31
- **Never:** include how a test was invoked (command, driver, device, tags, backend) unless the user asks for the invocation. Scenario name and pass/fail is enough. — added 2026-09-11
- **Never:** label a table cell, column, card, or stat with a bare identifier code (milestone `Mn`, ticket `P4.x`, a lint code). **Do:** show the readable title so a reader knows what the work is about without a lookup. — added 2026-09-07
- **Do:** when triaging a store app version, only flag issues where that version has more events than prior versions, and treat 1–4 unique users on the version under review as not critical. — added 2026-09-14
- **Always:** the first column of a canvas table is `#`, with 1-based row numbers. — added 2026-10-27
