---
url: https://orm.drizzle.team/docs/guides/cursor-based-pagination
title: "Cursor Based Pagination"
description: ""
access_date: 2026-08-03T19:00:22.305Z
current_date: 2026-08-03T19:00:22.305Z
---

## SQL Cursor-based pagination

PostgreSQL

MySQL

SQLite

MSSQL

Cockroach

This guide assumes familiarity with:

- Get started with [PostgreSQL](../get-started-postgresql.md), [MySQL](../mysql/get-started-mysql.md), [SQLite](../sqlite/get-started-sqlite.md), [MSSQL](../mssql/get-started-mssql.md) and [Cockroach](../cockroach/get-started-cockroach.md)
- [Select statement](../select.md) with [order by clause](../select.md#order-by)
- [Relational queries](../rqb.md) with [order by clause](../rqb.md#order-by)
- [Indices](../indexes-constraints.md)

This guide demonstrates how to implement `cursor-based` pagination in Drizzle:

index.ts

schema.ts

```ts
import { asc, gt } from 'drizzle-orm';
import { users } from './schema';

const db = drizzle(...);

const nextUserPage = async (cursor?: number, pageSize = 3) => {
  await db
    .select()
    .from(users)
    .where(cursor ? gt(users.id, cursor) : undefined) // if cursor is provided, get rows after it
    .limit(pageSize) // the number of rows to return
    .orderBy(asc(users.id)); // ordering
};

// pass the cursor of the last row of the previous page (id)
await nextUserPage(3);
```
```sql
select * from users order by id asc limit 3;
```
```ts
// next page, 4-6 rows returned
[
  {
    id: 4,
    firstName: 'Brian',
    lastName: 'Brown',
    createdAt: 2024-03-08T12:34:55.182Z
  },
  {
    id: 5,
    firstName: 'Beth',
    lastName: 'Davis',
    createdAt: 2024-03-08T12:40:55.182Z
  },
  {
    id: 6,
    firstName: 'Charlie',
    lastName: 'Miller',
    createdAt: 2024-03-08T13:04:55.182Z
  }
]
```

If you need dynamic order by you can do like below:

```ts
const nextUserPage = async (order: 'asc' | 'desc' = 'asc', cursor?: number, pageSize = 3) => {
  await db
    .select()
    .from(users)
    // cursor comparison
    .where(cursor ? (order === 'asc' ? gt(users.id, cursor) : lt(users.id, cursor)) : undefined)
    .limit(pageSize)
    .orderBy(order === 'asc' ? asc(users.id) : desc(users.id));
};

await nextUserPage();
await nextUserPage('asc', 3);
// descending order
await nextUserPage('desc');
await nextUserPage('desc', 7);
```

The main idea of this pagination is to use cursor as a pointer to a specific row in a dataset, indicating the end of the previous page. For correct ordering and cursor comparison, cursor should be unique and sequential.

If you need to order by a non-unique and non-sequential column, you can use multiple columns for cursor. This is how you can do it:

```ts
import { and, asc, eq, gt, or } from 'drizzle-orm';

const nextUserPage = async (
  cursor?: {
    id: number;
    firstName: string;
  },
  pageSize = 3,
) => {
  await db
    .select()
    .from(users)
    .where(
      cursor
        ? or(
            gt(users.firstName, cursor.firstName),
            and(eq(users.firstName, cursor.firstName), gt(users.id, cursor.id)),
          )
        : undefined,
    )
    .limit(pageSize)
    .orderBy(asc(users.firstName), asc(users.id));
};

// pass the cursor from previous page (id & firstName)
await nextUserPage({
  id: 2,
  firstName: 'Alex',
});
```
```sql
select * from users
  where (first_name > 'Alex' or (first_name = 'Alex' and id > 2))
  order by first_name asc, id asc limit 3;
```
```ts
// next page, 4-6 rows returned
[
  {
    id: 1,
    firstName: 'Alice',
    lastName: 'Johnson',
    createdAt: 2024-03-08T12:23:55.251Z
  },
  {
    id: 5,
    firstName: 'Beth',
    lastName: 'Davis',
    createdAt: 2024-03-08T12:40:55.182Z
  },
  {
    id: 4,
    firstName: 'Brian',
    lastName: 'Brown',
    createdAt: 2024-03-08T12:34:55.182Z
  }
]
```

Make sure to create indices for the columns that you use for cursor to make query efficient.

```ts
import { index, ...imports } from 'drizzle-orm/pg-core';

export const users = pgTable('users', {
  // columns declaration
},
(t) => [
  index('first_name_index').on(t.firstName).asc(),
  index('first_name_and_id_index').on(t.firstName, t.id).asc(),
]);
```
```sql
-- As of now drizzle-kit only supports index name and on() param, so you have to add order manually
CREATE INDEX IF NOT EXISTS "first_name_index" ON "users" ("first_name" ASC);
CREATE INDEX IF NOT EXISTS "first_name_and_id_index" ON "users" ("first_name" ASC,"id" ASC);
```

If you are using primary key which is not sequential (e.g. `UUIDv4`), you should add sequential column (e.g. `created_at` column) and use multiple cursor. This is how you can do it:

```ts
const nextUserPage = async (
  cursor?: {
    id: string;
    createdAt: Date;
  },
  pageSize = 3,
) => {
  await db
    .select()
    .from(users)
    .where(
      // make sure to add indices for the columns that you use for cursor
      cursor
        ? or(
            gt(users.createdAt, cursor.createdAt),
            and(eq(users.createdAt, cursor.createdAt), gt(users.id, cursor.id)),
          )
        : undefined,
    )
    .limit(pageSize)
    .orderBy(asc(users.createdAt), asc(users.id));
};

// pass the cursor from previous page (id & createdAt)
await nextUserPage({
  id: '66ed00a4-c020-4dfd-a1ca-5d2e4e54d174',
  createdAt: new Date('2024-03-09T17:59:36.406Z'),
});
```

Drizzle has useful relational queries API, that lets you easily implement `cursor-based` pagination:

```ts
import * as schema from './db/schema';

const db = drizzle(..., { schema });

const nextUserPage = async (cursor?: number, pageSize = 3) => {
  await db.query.users.findMany({
    where: (users, { gt }) => (cursor ? gt(users.id, cursor) : undefined),
    orderBy: (users, { asc }) => asc(users.id),
    limit: pageSize,
  });
};

// next page, cursor of last row of the first page (id = 3)
await nextUserPage(3);
```

**Benefits** of `cursor-based` pagination: consistent query results, with no skipped or duplicated rows due to insert or delete operations, and greater efficiency compared to `limit/offset` pagination because it does not need to scan and skip previous rows to access the next page.

**Drawbacks** of `cursor-based` pagination: the inability to directly navigate to a specific page and complexity of implementation. Since you add more columns to the sort order, you’ll need to add more filters to the `where` clause for the cursor comparison to ensure consistent pagination.

So, if you need to directly navigate to a specific page or you need simpler implementation of pagination, you should consider using [offset/limit](limit-offset-pagination.md) pagination instead.
