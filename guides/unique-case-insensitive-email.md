---
url: https://orm.drizzle.team/docs/guides/unique-case-insensitive-email
title: "Unique Case Insensitive Email"
description: ""
access_date: 2026-08-03T19:00:22.305Z
current_date: 2026-08-03T19:00:22.305Z
---

## Unique and Case-Insensitive Email Handling

This guide assumes familiarity with:

- Get started with [PostgreSQL](../get-started-postgresql.md), [MySQL](../mysql/get-started-mysql.md) and [SQLite](../sqlite/get-started-sqlite.md)
- [Indexes](../indexes-constraints.md#indexes)
- [Insert statement](../insert.md) and [Select method](../select.md)
- [sql operator](../sql.md)
- You should have `drizzle-orm@0.31.0` and `drizzle-kit@0.22.0` or higher.

### PostgreSQL

To implement a unique and case-insensitive `email` handling in PostgreSQL with Drizzle, you can create a unique index on the lowercased `email` column. This way, you can ensure that the `email` is unique regardless of the case.

Drizzle has simple and flexible API, which lets you easily create such an index using SQL-like syntax:

schema.ts

migration.sql

```ts
import { SQL, sql } from 'drizzle-orm';
import { AnyPgColumn, pgTable, serial, text, uniqueIndex } from 'drizzle-orm/pg-core';

export const users = pgTable(
  'users',
  {
    id: serial('id').primaryKey(),
    name: text('name').notNull(),
    email: text('email').notNull(),
  },
  (table) => [
    // uniqueIndex('emailUniqueIndex').on(sql\`lower(${table.email})\`),
    uniqueIndex('emailUniqueIndex').on(lower(table.email)),
  ],
);

// custom lower function
export function lower(email: AnyPgColumn): SQL {
  return sql\`lower(${email})\`;
}
```

This is how you can select user by `email` with `lower` function:

```ts
import { eq } from 'drizzle-orm';
import { lower, users } from './schema';

const db = drizzle(...);

const findUserByEmail = async (email: string) => {
  return await db
    .select()
    .from(users)
    .where(eq(lower(users.email), email.toLowerCase()));
};
```
```sql
select * from "users" where lower(email) = 'john@email.com';
```

### MySQL

In MySQL, the default collation setting for string comparison is case-insensitive, which means that when performing operations like searching or comparing strings in SQL queries, the case of the characters does not affect the results. However, because collation settings can vary and may be configured to be case-sensitive, we will explicitly ensure that the `email` is unique regardless of case by creating a unique index on the lowercased `email` column.

Drizzle has simple and flexible API, which lets you easily create such an index using SQL-like syntax:

schema.ts

migration.sql

```ts
import { SQL, sql } from 'drizzle-orm';
import { AnyMySqlColumn, mysqlTable, serial, uniqueIndex, varchar } from 'drizzle-orm/mysql-core';

export const users = mysqlTable(
  'users',
  {
    id: serial('id').primaryKey(),
    name: varchar('name', { length: 255 }).notNull(),
    email: varchar('email', { length: 255 }).notNull(),
  },
  (table) => [
    // uniqueIndex('emailUniqueIndex').on(sql\`(lower(${table.email}))\`),
    uniqueIndex('emailUniqueIndex').on(lower(table.email)),
  ]
);

// custom lower function
export function lower(email: AnyMySqlColumn): SQL {
  return sql\`(lower(${email}))\`;
}
```

IMPORTANT

Functional indexes are supported in MySQL starting from version `8.0.13`. For the correct syntax, the expression should be enclosed in parentheses, for example, `(lower(column))`.

This is how you can select user by `email` with `lower` function:

```ts
import { eq } from 'drizzle-orm';
import { lower, users } from './schema';

const db = drizzle(...);

const findUserByEmail = async (email: string) => {
  return await db
    .select()
    .from(users)
    .where(eq(lower(users.email), email.toLowerCase()));
};
```
```sql
select * from \`users\` where lower(email) = 'john@email.com';
```

### SQLite

To implement a unique and case-insensitive `email` handling in SQLite with Drizzle, you can create a unique index on the lowercased `email` column. This way, you can ensure that the `email` is unique regardless of the case.

Drizzle has simple and flexible API, which lets you easily create such an index using SQL-like syntax:

schema.ts

migration.sql

```ts
import { SQL, sql } from 'drizzle-orm';
import { AnySQLiteColumn, integer, sqliteTable, text, uniqueIndex } from 'drizzle-orm/sqlite-core';

export const users = sqliteTable(
  'users',
  {
    id: integer('id').primaryKey(),
    name: text('name').notNull(),
    email: text('email').notNull(),
  },
  (table) => [
    // uniqueIndex('emailUniqueIndex').on(sql\`lower(${table.email})\`),
    uniqueIndex('emailUniqueIndex').on(lower(table.email)),
  ]
);

// custom lower function
export function lower(email: AnySQLiteColumn): SQL {
  return sql\`lower(${email})\`;
}
```

This is how you can select user by `email` with `lower` function:

```ts
import { eq } from 'drizzle-orm';
import { lower, users } from './schema';

const db = drizzle(...);

const findUserByEmail = async (email: string) => {
  return await db
    .select()
    .from(users)
    .where(eq(lower(users.email), email.toLowerCase()));
};
```
```sql
select * from "users" where lower(email) = 'john@email.com';
```
