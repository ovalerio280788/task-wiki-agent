---
title: Read the real reason behind a Redshift COPY failure
updated: 2026-10-15
tags: [tooling, data, gotcha, prod]
area: tooling
---

## Context

An Airflow task that `COPY`s into Redshift fails with `ProgrammingError` and SQLSTATE `XX000`: `Load into table '<name>' failed. Check 'stl_load_errors' system table for details.` The Python stack is the client raising that message. The bad row is not in the task log.

## What works

Query as the database user that ran the `COPY`, or as a superuser. A regular user only sees their own load errors.

```sql
SELECT starttime, filename, line_number, colname, err_code, err_reason, raw_field_value
FROM stl_load_errors
ORDER BY starttime DESC
LIMIT 20;
```

If you cannot see those rows, download the source file from S3 and check it locally against `information_schema.columns`. Varchar length is in bytes, so compare `len(value.encode())`. Also check that integers fit their type and that timestamps match `TIMEFORMAT`. Compare with the last file that loaded to find the new offending row.

`sys_load_error_detail` is the same data without the raw field. Useful columns: `start_time`, `file_name`, `line_number`, `column_name`, `column_type`, `column_length`, `error_code`, `error_message`.

## Gotchas

- The Finch production Redshift plugin returns zero rows for these views when Airflow ran the load. That is visibility, not an empty error table.
- One bad row aborts the whole `COPY` when `MAXERROR` is left at the default of 0. Retries repeat the same rejection.

## Related

- [redshift allure ops](redshift-allure-ops.md)
