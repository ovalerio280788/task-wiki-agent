---
title: MDS storybook form field automation IDs
updated: 2026-08-11
tags: [gotcha, recipe]
area: repos/instawork
---

## Context

MDS Viewer form-field stories (`apps.mobile.templates.mobile.components` / `form_field`) render
several Django forms on one screen. Each form usually names its field `field`, so Django
`auto_id` yields the same accessibility id (`id_field`) on every instance.

## What works

- Form wrappers are already unique: `form-{field_index}-{form_index}`.
- Submit buttons are already unique: `submit-{form_index}` (from `hxpy_stories.py` / `stories.xml`).
- For unique, state-based field shell IDs, set explicit widget attrs in the widget's `stories()`:

```python
create_form(attrs={"id": "date-field-default"}),
create_form(attrs={"id": "date-field-value-set"}, initial=...),
```

- Labels follow the widget id when set (`label_{widget.attrs.id}` in `form_field` hxpy/DTL).
- Story source for DateField: `apps/mobile/forms/widgets/date_field.py`.

## Gotchas

- Changing the Django field *name* breaks story POST assertions that expect `{"field": ...}`.
  Prefer overriding `attrs["id"]`, not renaming the field.
- Single-form stories use a representative id such as `date-field-min-max`, rather than a
  positional suffix.
- Other multi-form widgets (PickerField, PasswordField, etc.) still share `id_field` until their
  stories get the same treatment.

## Related

- [appium-native-date-picker-locators](../tooling/appium-native-date-picker-locators.md)
- [mobile-native-datepicker-automation-ids](mobile-native-datepicker-automation-ids.md)
