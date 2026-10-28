---
title: Worker app shell 500 after login is often an uncaught Sendbird token fetch
updated: 2026-10-15
tags: [backend, mobile, qa, gotcha]
area: repos/instawork
---

## Context

A mobile test lands on the generic Hyperview error screen ("Uh oh, something went wrong") immediately after a successful phone login. The network log shows `GET /worker_app/app` 500, and the response body is `mobile/errors/500.xml`. The tabs never render, so a later step such as "my work navigation tab" is the reported failure.

## What works

Take `x-request-id` from that response and search Sentry errors with `request_id:"Root=..."`. Environment tag for QA is `QA`.

The authenticated shell builds context in `SendbirdSessionTokenMixin` before rendering `worker/app/index.xml`. That calls `get_or_create_session_token`, which posts to Sendbird `/users/{id}/token` with a 1 second timeout. A response with no token returns `""` and the shell still renders. `SendbirdConnectionError` and `SendbirdTimeoutError` are uncaught, so Django's 500 handler replaces the shell.

The ongoing group is [BACKEND-68TM](https://instawork.sentry.io/issues/BACKEND-68TM). A missing Sentry event for the request id means this path is not the cause.

The same screen on `GET /biz_app/app` right after partner login is often a different group: [BACKEND-68W9](https://instawork.sentry.io/issues/BACKEND-68W9), an `IntegrityError` inserting a duplicate `sendbird_sendbirdsessiontoken` row. Match the request id before treating every `/app` 500 as the connection error.

After business login, the mobile client starts the app-shell request and device registration independently. Both pass through `WebConfigurationMiddleware`, which fetches a Sendbird token for every authenticated company user, including API requests that do not use `js_config`. Their token inserts can race.

## Gotchas

The same URL with `?logged-out` returns 200. That request never fetches a token. A 500 only on the authenticated call is the signal.

A burst of new workers that passes, then fails together on Messages, is a different path. The shell token POST returned a body with no `token` in a few hundred milliseconds. That is not the 1 second timeout. `build()` maps any such body to `""`, including a rate-limit or user-not-found body, and only a timeout or connection error is treated as transient. `build()` logs that body with the Sendbird status, code, and message. Asking for the token again does not create the user, so a refresh loop stays empty and adds more token posts against the shared app.

A `SendbirdConnectionError` on `task_update_user_profile_in_sendbird` is a different call (`PUT /users/{id}`).

When that log is present, HTTP 400 with Sendbird code `400201` and message `"User" not found` means the token POST ran and the user id does not exist in the Sendbird app. That is not a timeout and not rate-limit code `500910`. Another token request stays empty until something creates the user.

Partner web config also fetches a token in `WebConfigurationMiddleware`, and only when the user has a company. A pro app shell failure is the mixin on `AppView`, not that middleware.

## Related

- [BrowserStack failure triage](../tooling/browserstack-failure-triage.md)
- [Local Sendbird user id collision](./instawork-local-sendbird-user-id-collision.md)
