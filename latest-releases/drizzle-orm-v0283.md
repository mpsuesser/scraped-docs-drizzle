---
url: https://orm.drizzle.team/docs/latest-releases/drizzle-orm-v0283
title: "Drizzle Orm V0283"
description: ""
access_date: 2026-08-03T19:08:17.353Z
current_date: 2026-08-03T19:08:17.353Z
---

## DrizzleORM v0.28.3 release

## Fixes

- Fixed sqlite-proxy and SQL.js response from `.get()` when the result is empty

## New Features

### 🎉 Added SQLite simplified query API

### 🎉 Added.$defaultFn() /.$default() methods to column builders

For more information check docs for [PostgreSQL](../column-types.md#default-value), [MySQL](../mysql/column-types.md#default-value) and [SQLite](../sqlite/column-types.md#default-value).

You can specify any logic and any implementation for a function like `cuid()` for runtime defaults. Drizzle won’t limit you in the number of implementations you can add.

> Note: This value does not affect the `drizzle-kit` behavior, it is only used at runtime in `drizzle-orm`

```ts
import { varchar, mysqlTable } from "drizzle-orm/mysql-core";
import { createId } from '@paralleldrive/cuid2';

const table = mysqlTable('table', {
    id: varchar('id', { length: 128 }).$defaultFn(() => createId()),
});
```

### 🎉 Added table.$inferSelect / table.\_.inferSelect and table.$inferInsert / table.\_.inferInsert for more convenient table model type inference

- 🛠 Deprecated `InferModel` type in favor of more explicit `InferSelectModel` and `InferInsertModel`
```ts
import { InferSelectModel, InferInsertModel } from 'drizzle-orm'

const usersTable = pgTable('users', {
  id: serial('id').primaryKey(),
  name: text('name').notNull(),
  verified: boolean('verified').notNull().default(false),
  jsonb: jsonb('jsonb').$type<string[]>(),
  createdAt: timestamp('created_at', { withTimezone: true }).notNull().defaultNow(),
});

type SelectUser = typeof usersTable.$inferSelect;
type InsertUser = typeof usersTable.$inferInsert;

type SelectUser2 = InferSelectModel<typeof usersTable>;
type InsertUser2 = InferInsertModel<typeof usersTable>;
```
- 🛠 Disabled `.d.ts` files bundling
