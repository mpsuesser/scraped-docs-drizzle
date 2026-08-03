---
url: https://orm.drizzle.team/docs/cockroach/guides/toggling-a-boolean-field
title: "Toggling A Boolean Field"
description: ""
access_date: 2026-08-03T19:38:26.356Z
current_date: 2026-08-03T19:38:26.356Z
---

## SQL Toggle value

PostgreSQL

MySQL

SQLite

MSSQL

Cockroach

This guide assumes familiarity with:

- Get started with [PostgreSQL](../../get-started-postgresql.md), [MySQL](../../mysql/get-started-mysql.md), [SQLite](../../sqlite/get-started-sqlite.md), [MSSQL](../../mssql/get-started-mssql.md) and [Cockroach](../get-started-cockroach.md)
- [Update statement](../../update.md)
- [Filters](../../operators.md) and [not operator](../../operators.md#not)
- Boolean data type in [MySQL](../../mysql/column-types.md#boolean) and [SQLite](../../sqlite/column-types.md#boolean)

To toggle a column value you can use `update().set()` method like below:

```tsx
import { eq, not } from 'drizzle-orm';

const db = drizzle(...);

await db
  .update(table)
  .set({
    isActive: not(table.isActive),
  })
  .where(eq(table.id, 1));
```
```sql
update "table" set "is_active" = not "is_active" where "id" = 1;
```

Please note that there is no boolean type in MySQL and SQLite. MySQL uses tinyint(1). SQLite uses integers 0 (false) and 1 (true).
