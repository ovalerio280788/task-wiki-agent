---
title: Asana bug report
updated: 2026-08-31
tags: [testing, tooling, runbook]
area: workflows
---

## Context

File product bugs on the **Bugs** board (`491474088351836`) from the live **New Bug Report**
task template. Create/update directly (no preview) unless the user asks to review first.

When the user provides a screen recording (especially with spoken narration), analyze the video
before filing so the ticket is accurate and the attached clip is short.

## What works

### Video analysis (before filing)

Use this whenever a recording is part of the evidence.

1. **Slice frames at 1 fps.** One still per second (AVFoundation on macOS, or `ffmpeg`). A 60s
   video → ~60 stills. Keep them for later, but do not review every frame blindly.
2. **Transcribe audio with timestamps.** Extract audio and run local Whisper (or equivalent). The
   transcript should include timestamps so spoken cues map to wall-clock time in the video. No
   local install needed — run it ephemerally, first run downloads the model:

```bash
ffmpeg -v error -y -i input.m4a -ar 16000 -ac 1 audio.wav
uvx --from openai-whisper whisper audio.wav --model small --language en \
  --output_format srt --output_dir .
```
3. **Reason over the transcript first.** Infer bug moments, navigation, and which seconds matter.
   Open only the stills around those timestamps (plus a little margin). Skip long stretches of
   unrelated setup unless the transcript points there.
4. **Decide the bug(s).** If one video shows multiple distinct bugs, file **one Asana task per
   bug**, each with its own trimmed clip and stills.
5. **Trim before attach.** Export a short clip that covers only repro steps + the failure (and a
   second or two of context). Do **not** attach the full long recording when most of it is
   unrelated setup. Reviewers dislike long videos full of steps that are not the bug.
6. **Stills for the ticket.** From the relevant window, keep 1–2 key frames (before state + failure
   state) to embed in the Screenshots table.
7. **Crop to the product UI.** When the recording is a desktop capture of a remote-device web
   console, the device screen is a small part of the frame, surrounded by browser chrome, toolbars,
   and log panes. Crop both the stills and the trimmed clip to the device screen rectangle
   (`ffmpeg -vf "crop=w:h:x:y,scale=..."`), and upscale — the raw region is small. One crop covers
   the whole clip only while the console layout holds still: hovering a console panel or opening a
   device-info flyout shifts or zooms the device rectangle mid-recording. Verify the crop on a frame
   at each end of the trim window, and pick a window inside the stable stretch.
   Measure the rectangle, do not estimate it: scan rows and columns of one frame for the longest
   contiguous light run bounded by the bezel bands (dark/coloured device frame), and take those
   bounds. Then render one cropped frame and actually look at it before attaching. A crop a few
   dozen pixels too narrow amputates the right edge (truncated labels, half-visible buttons) and
   still reads as a valid full screenshot in a thumbnail, so it survives review until someone opens
   it. Sanity-check the result against a phone aspect ratio (~0.45 w/h).
8. **Zoom before judging pixel-level layout bugs.** Overlap, clipping, and spacing complaints are
   not decidable at frame scale; crop tight around the element and upscale 3–6x first.
9. **Resample at 4 fps to judge timing.** 1 fps is enough to find the window but too coarse to prove
   ordering when the question is sub-second ("did the background land during the transition?").
   Re-extract just that window (`ffmpeg -ss <t> -t <secs> -vf "fps=4,crop=...,scale=..."`) and count
   frames between events to get the gap.
10. **Recover unreadable UI text from the source, not the pixels.** Screen-share recordings are
   low-bitrate, so text that is small, or covered by a translucent overlay, is destroyed in the
   encode; upscaling and contrast/gamma tricks only wash it out further. Grep the repo for a
   neighbouring visible string on the same screen, then read the exact label out of that template.

Long recordings (for example 20 minutes with the bug at minute 12) are fine as input; only the
trimmed segment goes on the ticket.

### End-to-end checklist

1. Collect evidence (do not invent facts; ask if missing):
   - Recording and/or screenshots (run **Video analysis** above when a recording is provided)
   - Backend env + admin/base URL (see **Backend SHA from health_check** below)
   - Backend SHA from that env's `/internal/health_check` (`{"sha": "..."}`) — fetch it; do not
     leave `Unknown` when the env is known
   - App version(s), including repro vs not-repro builds when known
   - OS, device, platform(s), repro steps, actual vs expected
   - For toast / overlay / “stuck UI” bugs: whether the app stays **interactable** vs frozen, and
     what (if anything) clears the UI (auto-timeout, focus change, navigation, etc.)
   - Optional: Sentry links, device logs (as a **`.txt` file**), Django links, test scenario id
2. Auth: `ASANA_ACCESS_TOKEN` (see [asana-api-cloud-agents](../tooling/asana-api-cloud-agents.md)).
3. Resolve the live template (never reuse a cached template GID or body):

```bash
curl -sS \
  "https://app.asana.com/api/1.0/task_templates?project=491474088351836&opt_fields=gid,name" \
  -H "Authorization: Bearer $ASANA_ACCESS_TOKEN" -H "Accept: application/json"
```

   Pick **New Bug Report**. Optionally instantiate a throwaway task first and read its
   `html_notes` if you need the current skeleton (template detail endpoint often omits body).
4. Instantiate (MCP `create_tasks` cannot use templates):

```bash
curl -sS -X POST \
  "https://app.asana.com/api/1.0/task_templates/<template_gid>/instantiateTask" \
  -H "Authorization: Bearer $ASANA_ACCESS_TOKEN" \
  -H "Accept: application/json" -H "Content-Type: application/json" \
  -d '{"data":{"name":"<concise bug title>"}}'
```

   Poll the job until `status=succeeded`; take `new_task.gid`.
5. Move to **Investigate** (`1200317047462310`) if it landed elsewhere (common: top of Bugs).
6. Read custom fields from the new task; set **Platform** (multi-enum), **Source** (for example
   Manually), leave **Type=Bug** from the template. Resolve option GIDs from the live task, not
   memory.
7. Upload attachments (**trimmed** video + key stills + optional **device log `.txt`**), then list
   GIDs:

```bash
curl -sS -X POST "https://app.asana.com/api/1.0/attachments" \
  -H "Authorization: Bearer $ASANA_ACCESS_TOKEN" \
  -F "parent=<task_gid>" -F "file=@/path/to/file;filename=evidence.ext"

curl -sS "https://app.asana.com/api/1.0/tasks/<task_gid>/attachments?opt_fields=name,permanent_url" \
  -H "Authorization: Bearer $ASANA_ACCESS_TOKEN" -H "Accept: application/json"
```

8. PUT `html_notes` that **match the live template exactly**:
   - Delete the yellow PLEASE READ intro.
   - Keep section order and heading style from the template: strong-wrapped
     `Description` / `Steps to Reproduce` / `Actual Result` / `Expected Result`; plain `<h2>` for
     `Details` and `Screenshots`.
   - Fill body text; use `<ul>`/`<ol>` for steps.
   - Keep **Details** and **Screenshots** as `<table>`s. Drop unused Details rows; do not convert
     tables into bullet lists.
   - Ship **exactly** the template sections, no more and no less: `Description`, `Steps to
     Reproduce`, `Actual Result`, `Expected Result`, `Details`, `Screenshots`. Never invent extra
     sections or extra tables (for example an iOS vs Android comparison table); platform
     differences belong in Description / Actual Result prose and the `OS` and `App version(s)`
     Details rows.
   - Video: link the **trimmed** clip →
     `<a href="https://app.asana.com/app/asana/-/get_asset?asset_id=<gid>">name.ext</a>`
   - Screenshots: embed **product UI** stills in table cells →
     `<img data-asana-gid="<attachment_gid>"/>`
   - Device logs: attach a **plain-text `.txt`** file and link it in Details. Do **not** attach or
     embed a screenshot of the console (not searchable for agents, and it becomes the board
     preview instead of the real bug still).
9. Prefer REST for template list/instantiate/attachments/`html_notes` with tables. MCP is fine for
   simple field updates when ready.
10. Return the task permalink to the user.

### Replacing evidence on an already-filed ticket

Once a ticket is live, humans edit it. Never re-author the body from your own draft.

1. GET `html_notes` immediately before the write, not from an earlier fetch in the session.
2. Regex-swap only the attachment references (video `<a>` href, `<img>` gid) in that fetched body,
   assert the old gids are gone and the new ones present, then PUT it back.
3. Upload the corrected file first, repoint the notes, then DELETE the superseded attachment, so the
   clipped or wrong version cannot become the board preview.

Asana rewrites `<img data-asana-gid="…"/>` into full `asanausercontent.com` markup with an expiring
`src`. That is normal; the gid attribute is the durable handle, so match on it and re-emit the bare
`data-asana-gid` form.

### Backend SHA from health_check

Every Instawork backend env exposes a public JSON health endpoint. When filing a bug, resolve the
env from the repro context (user, app settings footer, recording, or prior ticket), then fetch the
SHA yourself and put it in the Details `Backend SHA` row. Do not ask the user for the SHA when the
env is known.

```bash
curl -sS "<env_base>/internal/health_check"
# → {"healthy": true, "sha": "<40-char git sha>", ...}
```

| Backend (Details row) | Health-check URL |
| --- | --- |
| Staging | `https://admin.staging2.instawork.com/internal/health_check` |
| QA | `https://qa.instawork.com/internal/health_check` |
| Demo | `https://demo.instawork.com/internal/health_check` |
| Production | `https://admin.instawork.com/internal/health_check` (same SHA as `https://app.instawork.com/internal/health_check`) |
| Local | `http://localhost:8080/internal/health_check` |

Pattern for other hosts: `{scheme}://{host}/internal/health_check`. No auth. Take only the
`sha` field for the ticket. If the env is a named demo slice (`demo-<id>.instawork.com`), hit that
host's `/internal/health_check` the same way. If the fetch fails, write `Unknown` and note the
failed URL — do not invent a SHA.

### `html_notes` rules that matter

Allowed in practice via REST: `body`, `h1`/`h2`, `ul`/`ol`/`li`, `strong`/`em`, `a`, `table` /
`tr`/`td`, `img`, `hr`, `pre` (prefer outside cramped table cells).

Do **not** use `<p>` or `<br/>` (400 XML is invalid / flaky). Prefer no whitespace between a
closing `</h2>` and the section body — a newline there renders as a blank line under the heading
in Asana. Newlines *before* the next `<h2>` are fine (Asana already separates sections).
MCP docs omit `table`, but REST accepts it; use REST when writing template-shaped notes.

### Stable board pointers

| Thing | Value |
| --- | --- |
| Bugs project | `491474088351836` |
| Default triage section | Investigate `1200317047462310` |

Template GID is **not** stable for agents: always list and match by name.

## Gotchas

- Instantiated tasks have no `created_from_template` field; re-list templates every filing.
- Bare MCP create + pasted notes drifts from the template; always instantiate.
- Token env var is `ASANA_ACCESS_TOKEN`, not `ASANA_API_TOKEN`.
- Matching the template means structure and markup, not only section titles. Lists for Details /
  Screenshots are wrong when the template uses tables.
- Nested fancy markup inside table cells (heavy `<pre>`, `<br/>`) can fail; keep cells simple:
  text + links + `<img data-asana-gid>`.
- Attaching a long untrimmed recording with unrelated steps burns reviewer time; always trim to the
  bug window first.
- Reviewers count sections and tables against the template. A helpful-looking extra section or
  table gets rejected even when its content is correct; put that content inside an existing
  section instead.
- A newline right after `</h2>` becomes a visible blank line under the heading in Asana. Keep
  heading and body flush: `</h2>The text starts here` or `</h2><ol>...`.
- Leaving Backend SHA as `Unknown` when Staging/QA/Demo/Prod is known is avoidable: curl the
  env's `/internal/health_check` and read `sha`.
- Do not open every 1 fps still by default. Use the timestamped transcript to choose which frames
  matter.
- Silent recordings still get 1 fps sampling; skip Whisper and scan frames around user-described
  moments instead.
- Device logs belong in a `.txt` attachment (or pasted searchable text), never as a console
  screenshot. Screenshots of logs cannot be used well for agent investigation and hijack the
  task board preview away from the bug UI.
- Device console noise (keyboard `RemoteTextInput`, TCP `CLOSED`) is fine to attach as text, but
  do not overclaim root cause unless it clearly maps to the symptom.

## Related

- [asana-api-cloud-agents](../tooling/asana-api-cloud-agents.md) — REST auth and template list
- [qa-platform-story-pointing](qa-platform-story-pointing.md) — size QA follow-up after the bug is filed
- [hyperview-footer-safe-area](../repos/hyperview-footer-safe-area.md) — localize bottom-overlap UI
  bugs in the template before filing
