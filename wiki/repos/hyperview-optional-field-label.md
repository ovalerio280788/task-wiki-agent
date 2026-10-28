---
title: Hyperview optional field label
updated: 2026-10-07
tags: [backend, mobile, testing, gotcha]
area: repos/instawork
---

## Context

A mobile test looks for copy like "Positions (optional)", and the Python field `label` no longer contains that suffix. The heading can still render it.

## What works

The suffix is appended in the label template when the field is not required, not by the label string.

- Default: `apps/mobile/templates/mobile/components/form_field/index.xml` and the same check in `apps/mobile/templates/mobile/components/form_field/hxpy.py`.
- A widget `label_template_name` replaces that default. Search that template for `{% trans "optional" %}` before blaming the `label=` argument.

## Gotchas

Rewriting `label=_("Foo (optional)")` to `label=_("Foo")` leaves "(optional)" on screen while the template still has `{% if not field.field.required %} ({% trans "optional" %})`.
