---
url: https://orm.drizzle.team/docs/singlestore/guides/empty-array-default-value
title: "Empty Array Default Value"
description: ""
access_date: 2026-08-03T18:54:19.022Z
current_date: 2026-08-03T18:54:19.022Z
---

## Empty array as a default value

This guide assumes familiarity with:

- Get started with [PostgreSQL](../../get-started-postgresql.md), [MySQL](../../mysql/get-started-mysql.md) and [SQLite](../../sqlite/get-started-sqlite.md)
- Learn about column data types for [PostgreSQL](../../column-types.md), [MySQL](../../mysql/column-types.md) and [SQLite](../../sqlite/column-types.md)
- [sql operator](../../sql.md)

### PostgreSQL

To set an empty array as a default value in PostgreSQL, you can use `sql` operator with `'{}'` or `ARRAY[]` syntax:

```ts
import { sql } from 'drizzle-orm';
import { pgTable, serial, text } from 'drizzle-orm/pg-core';

export const users = pgTable('users', {
  id: serial('id').primaryKey(),
  name: text('name').notNull(),
  tags1: text('tags1')
    .array()
    .notNull()
    .default(sql\`'{}'::text[]\`),
  tags2: text('tags2')
    .array()
    .notNull()
    .default(sql\`ARRAY[]::text[]\`),
});
```
```sql
CREATE TABLE IF NOT EXISTS "users" (
    "id" serial PRIMARY KEY NOT NULL,
    "name" text NOT NULL,
    "tags1" text[] DEFAULT '{}'::text[] NOT NULL,
    "tags2" text[] DEFAULT ARRAY[]::text[] NOT NULL
);
```

### MySQL

MySQL doesn’t have an array data type, but you can use `json` data type for the same purpose. To set an empty array as a default value in MySQL, you can use `JSON_ARRAY()` function or `sql` operator with `('[]')` syntax:

```ts
import { sql } from 'drizzle-orm';
import { json, mysqlTable, serial, varchar } from 'drizzle-orm/mysql-core';

export const users = mysqlTable('users', {
  id: serial('id').primaryKey(),
  name: varchar('name', { length: 255 }).notNull(),
  tags1: json('tags1').$type<string[]>().notNull().default([]),
  tags2: json('tags2')
    .$type<string[]>()
    .notNull()
    .default(sql\`('[]')\`), // the same as default([])
  tags3: json('tags3')
    .$type<string[]>()
    .notNull()
    .default(sql\`(JSON_ARRAY())\`),
});
```
```sql
CREATE TABLE \`users\` (
    \`id\` serial AUTO_INCREMENT NOT NULL,
    \`name\` varchar(255) NOT NULL,
    \`tags1\` json NOT NULL DEFAULT ('[]'),
    \`tags2\` json NOT NULL DEFAULT ('[]'),
    \`tags3\` json NOT NULL DEFAULT (JSON_ARRAY()),
    CONSTRAINT \`users_id\` PRIMARY KEY(\`id\`)
);
```

The `mode` option defines how values are handled in the application. With `json` mode, values are treated as JSON object literal.

You can specify `.$type<..>()` for json object inference, it will not check runtime values. It provides compile time protection for default values, insert and select schemas.

### SQLite

SQLite doesn’t have an array data type, but you can use `text` data type for the same purpose. To set an empty array as a default value in SQLite, you can use `json_array()` function or `sql` operator with `'[]'` syntax:

```ts
import { sql } from 'drizzle-orm';
import { integer, sqliteTable, text } from 'drizzle-orm/sqlite-core';

export const users = sqliteTable('users', {
  id: integer('id').primaryKey(),
  tags1: text('tags1', { mode: 'json' })
    .notNull()
    .$type<string[]>()
    .default(sql\`(json_array())\`),
  tags2: text('tags2', { mode: 'json' })
    .notNull()
    .$type<string[]>()
    .default(sql\`'[]'\`),
});
```
```sql
CREATE TABLE \`users\` (
    \`id\` integer PRIMARY KEY NOT NULL,
    \`tags1\` text DEFAULT (json_array()) NOT NULL,
    \`tags2\` text DEFAULT '[]' NOT NULL
);
```

The `mode` option defines how values are handled in the application. With `json` mode, values are treated as JSON object literal.

You can specify `.$type<..>()` for json object inference, it will not check runtime values. It provides compile time protection for default values, insert and select schemas.
