---
title: Local worker ids inherit chats from the shared Sendbird app
updated: 2026-10-15
tags: [backend, mobile, local, demo, gotcha]
area: repos/instawork
---

## Context

A newly created local worker opens Messages and sees old conversations. The test created the worker in this run. The session token is new too.

## What works

Sendbird identity is the numeric user id in `POST /users/{user_profile.id}/token`. The session token only authenticates that id. Channels belong to the Sendbird user.

Local MySQL ids are small. The local backend uses the shared Sendbird app, where those same ids already belong to real users. Compare the new worker id with `GET /users/{id}` and `GET /users/{id}/my_group_channels` on that app. An old `created_at` and existing channels mean the list is inherited.

Deactivating the local worker does not delete the Sendbird user.

## Gotchas

Do not treat a fresh `SendbirdSessionToken` row as proof the inbox is empty. The row stores the access token, not the channel list.

When the empty-inbox copy is missing, use the screenshot. Gray placeholder rows under the Messages chrome mean the list is still loading. Named conversations mean that Sendbird user already had channels.

## Related

- [Worker app shell 500](./instawork-worker-app-shell-sendbird-500.md)
