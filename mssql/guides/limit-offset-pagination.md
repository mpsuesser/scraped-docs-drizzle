---
url: https://orm.drizzle.team/docs/mssql/guides/limit-offset-pagination
title: "Limit Offset Pagination"
description: ""
access_date: 2026-08-03T19:08:17.353Z
current_date: 2026-08-03T19:08:17.353Z
---

## SQL Limit/Offset pagination

PostgreSQL

MySQL

SQLite

MSSQL

Cockroach

This guide assumes familiarity with:

- Get started with [PostgreSQL](../../get-started-postgresql.md), [MySQL](../../mysql/get-started-mysql.md), [SQLite](../../sqlite/get-started-sqlite.md), [MSSQL](../get-started-mssql.md) and [Cockroach](../../cockroach/get-started-cockroach.md)
- [Select statement](../../select.md) with [order by clause](../../select.md#order-by) and [limit & offset clauses](../../select.md#limit--offset)
- [Relational queries](../../rqb.md) with [order by clause](../../rqb.md#order-by) and [limit & offset clauses](../../rqb.md#limit--offset)
- [Dynamic query building](../../dynamic-query-building.md)

This guide demonstrates how to implement `limit/offset` pagination in Drizzle:

### PostgreSQL, MySQL, SQLite and Cockroach

To implement a PostgreSQL, MySQL, SQLite and Cockroach SQL Limit/Offset pagination (skip to [MSSQL](../../guides/limit-offset-pagination.md#mssql)) with Drizzle you can use following example:

index.ts

schema.ts

```ts
import { asc } from 'drizzle-orm';
import { users } from './schema';

const db = drizzle(...);

await db
  .select()
  .from(users)
  .orderBy(asc(users.id)) // order by is mandatory
  .limit(4) // the number of rows to return
  .offset(4); // the number of rows to skip
```
```sql
select * from users order by id asc limit 4 offset 4;
```
```ts
// 5-8 rows returned
[
  {
    id: 5,
    firstName: 'Beth',
    lastName: 'Davis',
    createdAt: 2024-03-11T20:51:46.787Z
  },
  {
    id: 6,
    firstName: 'Charlie',
    lastName: 'Miller',
    createdAt: 2024-03-11T21:15:46.787Z
  },
  {
    id: 7,
    firstName: 'Clara',
    lastName: 'Wilson',
    createdAt: 2024-03-11T21:33:46.787Z
  },
  {
    id: 8,
    firstName: 'David',
    lastName: 'Moore',
    createdAt: 2024-03-11T21:45:46.787Z
  }
]
```

Limit is the number of rows to return `(page size)` and offset is the number of rows to skip `((page number - 1) * page size)`. For consistent pagination, ensure ordering by a unique column. Otherwise, the results can be inconsistent.

If you need to order by a non-unique column, you should also append a unique column to the ordering.

This is how you can implement `limit/offset` pagination with 2 columns:

```ts
const getUsers = async (page = 1, pageSize = 3) => {
  await db
    .select()
    .from(users)
    .orderBy(asc(users.firstName), asc(users.id)) // order by first_name (non-unique), id (pk)
    .limit(pageSize) 
    .offset((page - 1) * pageSize);
}

await getUsers();
```

Drizzle has useful relational queries API, that lets you easily implement `limit/offset` pagination:

```ts
import * as schema from './db/schema';

const db = drizzle({ schema });

const getUsers = async (page = 1, pageSize = 3) => {
  await db.query.users.findMany({
    orderBy: (users, { asc }) => asc(users.id),
    limit: pageSize,
    offset: (page - 1) * pageSize,
  });
};

await getUsers();
```

Drizzle has simple and flexible API, which lets you easily create custom solutions. This is how you can create custom function for pagination using `.$dynamic()` function:

```ts
import { SQL, asc } from 'drizzle-orm';
import { PgColumn, PgSelect } from 'drizzle-orm/pg-core';

function withPagination<T extends PgSelect>(
  qb: T,
  orderByColumn: PgColumn | SQL | SQL.Aliased,
  page = 1,
  pageSize = 3,
) {
  return qb
    .orderBy(orderByColumn)
    .limit(pageSize)
    .offset((page - 1) * pageSize);
}

const query = db.select().from(users); // query that you want to execute with pagination

await withPagination(query.$dynamic(), asc(users.id));
```

You can improve performance of `limit/offset` pagination by using `deferred join` technique. This method performs the pagination on a subset of the data instead of the entire table.

To implement it you can do like this:

```ts
const getUsers = async (page = 1, pageSize = 10) => {
   const sq = db
    .select({ id: users.id })
    .from(users)
    .orderBy(users.id)
    .limit(pageSize)
    .offset((page - 1) * pageSize)
    .as('subquery');

   await db.select().from(users).innerJoin(sq, eq(users.id, sq.id)).orderBy(users.id);
};
```

**Benefits** of `limit/offset` pagination: it’s simple to implement and pages are easily reachable, which means that you can navigate to any page without having to save the state of the previous pages.

**Drawbacks** of `limit/offset` pagination: degradation in query performance with increasing offset because database has to scan all rows before the offset to skip them, and inconsistency due to data shifts, which can lead to the same row being returned on different pages or rows being skipped.

This is how it works:

```ts
const getUsers = async (page = 1, pageSize = 3) => {
  await db
    .select()
    .from(users)
    .orderBy(asc(users.id))
    .limit(pageSize)
    .offset((page - 1) * pageSize);
};

// user is browsing the first page
await getUsers();
```
```ts
// results for the first page
[
  {
    id: 1,
    firstName: 'Alice',
    lastName: 'Johnson',
    createdAt: 2024-03-10T17:17:06.148Z
  },
  {
    id: 2,
    firstName: 'Alex',
    lastName: 'Smith',
    createdAt: 2024-03-10T17:19:06.147Z
  },
  {
    id: 3,
    firstName: 'Aaron',
    lastName: 'Williams',
    createdAt: 2024-03-10T17:22:06.147Z
  }
]
```
```ts
// while user is browsing the first page, a row with id 2 is deleted
await db.delete(users).where(eq(users.id, 2));

// user navigates to the second page
await getUsers(2);
```
```ts
// second page, row with id 3 was skipped
[
  {
    id: 5,
    firstName: 'Beth',
    lastName: 'Davis',
    createdAt: 2024-03-10T17:34:06.147Z
  },
  {
    id: 6,
    firstName: 'Charlie',
    lastName: 'Miller',
    createdAt: 2024-03-10T17:58:06.147Z
  },
  {
    id: 7,
    firstName: 'Clara',
    lastName: 'Wilson',
    createdAt: 2024-03-10T18:16:06.147Z
  }
]
```

So, if your database experiences frequently insert and delete operations in real time or you need high performance to paginate large tables, you should consider using [cursor-based](../../guides/cursor-based-pagination.md) pagination instead.

To learn more about `deferred join` technique you should follow these guides: [Planetscale Pagination Guide](https://planetscale.com/blog/mysql-pagination) and [Efficient Pagination Guide by Aaron Francis](https://aaronfrancis.com/2022/efficient-pagination-using-deferred-joins).

### MSSQL

To implement an MSSQL Limit/Offset pagination with Drizzle you can use following example:

index.ts

schema.ts

```ts
import { asc } from 'drizzle-orm';
import { users } from './schema';

const db = drizzle(...);

await db
  .select()
  .from(users)
  .orderBy(asc(users.id)) // order by is mandatory
  .offset(4) // the number of rows to skip
  .fetch(4) // the number of rows to fetch
```
```sql
select * from [users] order by [id] asc offset 5 rows fetch next 10 rows only;
```
```ts
// 5-8 rows returned
[
  {
    id: 5,
    firstName: 'Beth',
    lastName: 'Davis',
    createdAt: 2024-03-11T20:51:46.787Z
  },
  {
    id: 6,
    firstName: 'Charlie',
    lastName: 'Miller',
    createdAt: 2024-03-11T21:15:46.787Z
  },
  {
    id: 7,
    firstName: 'Clara',
    lastName: 'Wilson',
    createdAt: 2024-03-11T21:33:46.787Z
  },
  {
    id: 8,
    firstName: 'David',
    lastName: 'Moore',
    createdAt: 2024-03-11T21:45:46.787Z
  }
]
```
