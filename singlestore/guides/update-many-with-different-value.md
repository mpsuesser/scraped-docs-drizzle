---
url: https://orm.drizzle.team/docs/singlestore/guides/update-many-with-different-value
title: "Update Many With Different Value"
description: ""
access_date: 2026-08-03T18:54:19.022Z
current_date: 2026-08-03T18:54:19.022Z
---

## Update many with different values for each row

PostgreSQL

MySQL

SQLite

MSSQL

Cockroach

This guide assumes familiarity with:

- Get started with [PostgreSQL](../../get-started-postgresql.md), [MySQL](../../mysql/get-started-mysql.md), [SQLite](../../sqlite/get-started-sqlite.md), [MSSQL](../../mssql/get-started-mssql.md) and [Cockroach](../../cockroach/get-started-cockroach.md)
- [Update statement](../../update.md)
- [Filters](../../operators.md) and [sql operator](../../sql.md)

To implement update many with different values for each row within 1 request you can use `sql` operator with `case` statement and `.update().set()` methods like this:

```ts
import { SQL, inArray, sql } from 'drizzle-orm';
import { users } from './schema';

const db = drizzle(...);

const inputs = [
  {
    id: 1,
    city: 'New York',
  },
  {
    id: 2,
    city: 'Los Angeles',
  },
  {
    id: 3,
    city: 'Chicago',
  },
];

// You have to be sure that inputs array is not empty
if (inputs.length === 0) {
  return;
}

const sqlChunks: SQL[] = [];
const ids: number[] = [];

sqlChunks.push(sql\`(case\`);

for (const input of inputs) {
  sqlChunks.push(sql\`when ${users.id} = ${input.id} then ${input.city}\`);
  ids.push(input.id);
}

sqlChunks.push(sql\`end)\`);

const finalSql: SQL = sql.join(sqlChunks, sql.raw(' '));

await db.update(users).set({ city: finalSql }).where(inArray(users.id, ids));
```
```sql
update users set "city" = 
  (case when id = 1 then 'New York' when id = 2 then 'Los Angeles' when id = 3 then 'Chicago' end)
where id in (1, 2, 3)
```
