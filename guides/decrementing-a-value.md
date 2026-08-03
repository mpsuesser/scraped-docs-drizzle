---
url: https://orm.drizzle.team/docs/guides/decrementing-a-value
title: "Decrementing A Value"
description: ""
access_date: 2026-08-03T18:54:19.022Z
current_date: 2026-08-03T18:54:19.022Z
---

## SQL Decrement value

PostgreSQL

MySQL

SQLite

MSSQL

Cockroach

This guide assumes familiarity with:

- Get started with [PostgreSQL](../get-started-postgresql.md), [MySQL](../mysql/get-started-mysql.md), [SQLite](../sqlite/get-started-sqlite.md), [MSSQL](../mssql/get-started-mssql.md) and [Cockroach](../cockroach/get-started-cockroach.md)
- [Update statement](../update.md)
- [Filters](../operators.md) and [sql operator](../sql.md)

To decrement a column value you can use `update().set()` method like below:

```ts
import { eq, sql } from 'drizzle-orm';

const db = drizzle(...)
  
await db
  .update(table)
  .set({
    counter: sql\`${table.counter} - 1\`,
  })
  .where(eq(table.id, 1));
```
```sql
update "table" set "counter" = "counter" - 1 where "id" = 1;
```

Drizzle has simple and flexible API, which lets you easily create custom solutions. This is how you do custom decrement function:

```ts
import { AnyColumn } from 'drizzle-orm';

const decrement = (column: AnyColumn, value = 1) => {
  return sql\`${column} - ${value}\`;
};

await db
  .update(table)
  .set({
    counter1: decrement(table.counter1),
    counter2: decrement(table.counter2, 10),
  })
  .where(eq(table.id, 1));
```
