---
title: Test Django get_or_create conflict recovery without a thread race
updated: 2026-10-16
tags: [testing, backend, gotcha]
area: tooling
---

## Context

A unique-key race is recovered inside `QuerySet.get_or_create`: the first lookup misses, `create()` raises `IntegrityError`, and a second lookup returns the row that won. A deterministic test needs the winning row to survive that failure.

## What works

On Django 4.2 the order inside `get_or_create` is:

1. `get()`
2. `_extract_model_params()`
3. `transaction.atomic()` around `create()`
4. On `IntegrityError`, `get()` again

Insert the winning row from a patch of `_extract_model_params`, guarded to the model under test. That is after the miss and before the savepoint. Let the real `create()` raise the duplicate-key error. Assert the caller returns the winning row.

## Gotchas

An insert inside a patched `create()` or `save()` happens inside the savepoint. The failed insert rolls that row back, and the recovery lookup raises `DoesNotExist`.

A second thread needs `TransactionTestCase` and can miss the row under the test transaction. Use the savepoint boundary instead.

Constructing an unsaved model with its one-to-one parent set caches that instance on the parent. A later `hasattr(parent, related_name)` then sees the unsaved instance and skips the database. Build that instance inside the mocked call, after the parent lookup has already missed.
