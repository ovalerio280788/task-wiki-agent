---
title: Documentation preferences
updated: 2026-09-02
scope: documentation
---

How the user wants me to write documentation (markdown docs, READMEs, wiki pages, comments meant as
prose). Read this when the task involves writing documentation.

- **Never:** use em dashes or overuse regular dashes; they read as AI writing. — added 2026-06-19
- **Always:** write in sentence case. **Never:** camel case in prose. — added 2026-06-19
- **Always:** keep PR descriptions concise and skimmable so people read to the end. **Instead of:** long write-ups that go unread. — added 2026-06-23
- **Always:** put the Asana task link at the very top of the PR description (when the work has a task). — added 2026-06-23
- **Always:** put related PR links at the very top of the Asana task description. — added 2026-06-23
- **Do:** write Asana task descriptions as rich text (`html_notes`) with real heading elements and nested lists. No whitespace after `</h2>` (body starts on the same line); exactly one blank line (`\n\n`) between sections. **Instead of:** a newline after the heading (Asana renders that as a blank under the header), plain-text markdown (renders literal `**`), `<table>` (not supported), or `<pre>` blocks for tabular data (renders as a line-numbered code box that overflows). — updated 2026-09-02
- **Do:** write Asana *comments* as plain `text`. **Instead of:** `html_text`, which the comment API stores as literal escaped markup so the reader sees raw tags. Rich text works for task descriptions (`html_notes`), not comments. — added 2026-09-02
- **Always:** use the `/writing-guidelines` skillbefore drafting or polishing emails, Slack replies, Markdown docs, PR descriptions, and other prose. — added 2026-07-01
- **Do:** for presentation decks, keep visible slide text sparse, mix visual types, and describe the intended visual for each slide. **Instead of:** dense bullet-heavy slides or overusing diagrams; put detail in speaker notes when useful. — updated 2026-07-01
- **Always:** for a decision or spike page, open with the answer in one sentence, then the tracking ticket link. — added 2026-08-19
- **Always:** show how numbers were measured (commands, tool version, constraints) before the results. — added 2026-08-19
- **Always:** list the full inventory in tables. **Never:** hide the long tail as "N other codes". — added 2026-08-19
- **Do:** split "what it is" from "what to do" into two tables. **Instead of:** repeating the action column. — added 2026-08-19
- **Do:** keep delivery as a short numbered list and omit PR-centric framing. **Instead of:** CI YAML or "this PR" language on Confluence. — added 2026-08-19
- **Do:** mark only the real exception (warning + one clause). **Instead of:** a Yes/No column on every row. — added 2026-08-19
- **Do:** give each list item its own markdown table row, with the shared columns filled on the first row only. **Instead of:** stacking items in one cell with `<br>` or `<ul><li>`, which the table formatter flattens into one run-on line. — added 2026-08-30
