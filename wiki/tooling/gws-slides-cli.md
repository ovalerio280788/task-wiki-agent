---
title: Google Slides creation with gws
updated: 2026-07-01
tags: [tooling, command, gotcha]
area: tooling
---

## Context

Use this when creating or updating Google Slides with the local `gws` command.

## What works

Inspect the method schemas before calling the API:

```bash
gws schema slides.presentations.create
gws schema slides.presentations.batchUpdate
gws schema slides.presentations.get
```

Create the presentation:

```bash
gws slides presentations create --json '{"title":"deck title"}'
```

Apply slide edits with `batchUpdate`:

```bash
gws slides presentations batchUpdate \
  --params '{"presentationId":"presentation_id"}' \
  --json '{"requests":[]}'
```

To populate speaker notes, fetch the presentation after creating slides. Each slide has a `slideProperties.notesPage`; insert text into the notes page element whose placeholder type is `BODY`.

To stage a local image for Slides, upload it to Drive without public sharing:

```bash
gws drive files create \
  --upload image.png \
  --json '{"name":"image.png","mimeType":"image/png"}'
```

If the file needs to be shared, use an organization-scoped permission only:

```bash
gws drive permissions create \
  --params '{"fileId":"file_id"}' \
  --json '{"type":"domain","role":"reader","domain":"org-domain.example"}'
```

Do not use anyone-with-link access. If a Slides API image insertion path requires a public URL, stop and use a placeholder or a manual organization-safe insertion path instead.


## Gotchas

Shape fill colors and text colors use different object shapes:

```json
{
  "shapeBackgroundFill": {
    "solidFill": {
      "color": {
        "rgbColor": {
          "red": 1,
          "green": 1,
          "blue": 1
        }
      }
    }
  }
}
```

Text foreground colors use `opaqueColor` instead:

```json
{
  "foregroundColor": {
    "opaqueColor": {
      "rgbColor": {
        "red": 0.1,
        "green": 0.2,
        "blue": 0.3
      }
    }
  }
}
```

If shape fills use `opaqueColor`, `batchUpdate` fails schema validation before applying any changes.

`gws drive files create --upload` validates that the upload path is under the current working directory. Run the upload command from the generated asset folder, or copy the asset into the workspace before uploading.

Slides API `createImage` rejects Drive download URLs that are only organization-readable. If it asks for a publicly accessible image, do not grant anyone-with-link access. Use a placeholder, manual Slides upload, or another organization-safe path.
