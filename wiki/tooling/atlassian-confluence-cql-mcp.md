---
title: Confluence CQL search via Atlassian MCP
updated: 2026-06-26
tags: [tooling, capability, gotcha]
area: tooling
---

## Context

Bulk-collecting Confluence pages (for example everything created by one user) through the
Atlassian MCP server `plugin-atlassian-atlassian`, tool `searchConfluenceUsingCql`.

## What works

- Resolve identity first: `getAccessibleAtlassianResources` returns the `cloudId`, and
  `atlassianUserInfo` returns the current `account_id`. Pass the accountId straight into CQL.
- Filter created pages with CQL: `type=page AND creator = "<accountId>" ORDER BY created DESC`.
- Get page bodies in the same search by setting `expand` to `content.body.view,content.history`.
  `body.view` is rendered HTML, easy to strip to plain text. A plain search with no expand returns
  metadata only but the full result set in one call.
- The result file ends with `size`, `totalSize`, and a `_links.next` cursor URL.

## Gotchas

- Expanding bodies caps each page at 50 results regardless of the `limit` you pass. With more than
  50 matches you must paginate.
- Paginate by passing the `cursor` param, taking its value from the `next` link and URL-decoding it
  (`%3D` to `=`, `%2C` to `,`). Repeat until `next` is absent. A metadata-only search has no such
  cap, so use it to confirm `totalSize` up front.
- The space key is not a direct field; pull it from `content._expandable.container`
  (`/rest/api/space/<KEY>`), and the human name from `resultGlobalContainer.title`.
