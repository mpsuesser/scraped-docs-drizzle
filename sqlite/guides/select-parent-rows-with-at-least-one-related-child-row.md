---
url: https://orm.drizzle.team/docs/sqlite/guides/select-parent-rows-with-at-least-one-related-child-row
title: "Select Parent Rows With At Least One Related Child Row"
description: ""
access_date: 2026-08-03T19:08:17.353Z
current_date: 2026-08-03T19:08:17.353Z
---

PostgreSQL

MySQL

SQLite

MSSQL

Cockroach

This guide assumes familiarity with:

- Get started with [PostgreSQL](../../get-started-postgresql.md), [MySQL](../../mysql/get-started-mysql.md), [SQLite](../get-started-sqlite.md), [MSSQL](../../mssql/get-started-mssql.md) and [Cockroach](../../cockroach/get-started-cockroach.md)
- [Select statement](../../select.md) and [select from subquery](../../select.md#select-from-subquery)
- [Inner join](../../joins.md#inner-join)
- [Filter operators](../../operators.md) and [exists function](../../operators.md#exists)

This guide demonstrates how to select parent rows with the condition of having at least one related child row. Below, there are examples of schema definitions and the corresponding database data:

```ts
import { integer, pgTable, serial, text } from 'drizzle-orm/pg-core';

export const users = pgTable('users', {
  id: serial('id').primaryKey(),
  name: text('name').notNull(),
  email: text('email').notNull(),
});

export const posts = pgTable('posts', {
  id: serial('id').primaryKey(),
  title: text('title').notNull(),
  content: text('content').notNull(),
  userId: integer('user_id').notNull().references(() => users.id),
});
```

users.db

posts.db

```plaintext
+----+------------+----------------------+
| id |    name    |        email         |
+----+------------+----------------------+
|  1 | John Doe   | john_doe@email.com   |
+----+------------+----------------------+
|  2 | Tom Brown  | tom_brown@email.com  |
+----+------------+----------------------+
|  3 | Nick Smith | nick_smith@email.com |
+----+------------+----------------------+
```

To select parent rows with at least one related child row and retrieve child data you can use `.innerJoin()` method:

```ts
import { eq } from 'drizzle-orm';
import { users, posts } from './schema';

const db = drizzle(...);

await db
  .select({
    user: users,
    post: posts,
  })
  .from(users)
  .innerJoin(posts, eq(users.id, posts.userId));
  .orderBy(users.id);
```
```sql
select users.*, posts.* from users
  inner join posts on users.id = posts.user_id
  order by users.id;
```
```ts
// result data, there is no user with id 2 because he has no posts
[
  {
    user: { id: 1, name: 'John Doe', email: 'john_doe@email.com' },
    post: {
      id: 1,
      title: 'Post 1',
      content: 'This is the text of post 1',
      userId: 1
    }
  },
  {
    user: { id: 1, name: 'John Doe', email: 'john_doe@email.com' },
    post: {
      id: 2,
      title: 'Post 2',
      content: 'This is the text of post 2',
      userId: 1
    }
  },
  {
    user: { id: 3, name: 'Nick Smith', email: 'nick_smith@email.com' },
    post: {
      id: 3,
      title: 'Post 3',
      content: 'This is the text of post 3',
      userId: 3
    }
  }
]
```

To only select parent rows with at least one related child row you can use subquery with `exists()` function like this:

```ts
import { eq, exists, sql } from 'drizzle-orm';

const sq = db
  .select({ id: sql\`1\` })
  .from(posts)
  .where(eq(posts.userId, users.id));

await db.select().from(users).where(exists(sq));
```
```sql
select * from users where exists (select 1 from posts where posts.user_id = users.id);
```
```ts
// result data, there is no user with id 2 because he has no posts
[
  { id: 1, name: 'John Doe', email: 'john_doe@email.com' },
  { id: 3, name: 'Nick Smith', email: 'nick_smith@email.com' }
]
```
