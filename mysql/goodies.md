---
url: https://orm.drizzle.team/docs/mysql/goodies
title: "Goodies"
description: ""
access_date: 2026-08-03T19:00:22.305Z
current_date: 2026-08-03T19:00:22.305Z
---

## Type API

To retrieve a type from your table schema for `select` and `insert` queries, you can make use of our type helpers.

```ts
import { int, text, mysqlTable } from 'drizzle-orm/mysql-core';
import { type InferSelectModel, type InferInsertModel } from 'drizzle-orm'

const users = mysqlTable('users', {
  id: int().primaryKey(),
  name: text().notNull(),
});

type SelectUser = typeof users.$inferSelect;
type InsertUser = typeof users.$inferInsert;
// or
type SelectUser = InferSelectModel<typeof users>;
type InsertUser = InferInsertModel<typeof users>;
```

## Logging

To enable default query logging, just pass `{ logger: true }` to the `drizzle` initialization function:

```typescript
import { drizzle } from 'drizzle-orm/mysql2';

const db = drizzle({ logger: true });
```

You can change the logs destination by creating a `DefaultLogger` instance and providing a custom `writer` to it:

```typescript
import { DefaultLogger, type LogWriter } from "drizzle-orm/logger";
import { drizzle } from "drizzle-orm/mysql2";

class MyLogWriter implements LogWriter {
  write(message: string) {
    // Write to file, stdout, etc.
  }
}

const logger = new DefaultLogger({ writer: new MyLogWriter() });
const db = drizzle(process.env.DB_URL, { logger });
```

You can also create a custom logger:

```typescript
import { type Logger } from 'drizzle-orm/logger';
import { drizzle } from 'drizzle-orm/mysql2';

class MyLogger implements Logger {
  logQuery(query: string, params: unknown[]): void {
    console.log({ query, params });
  }
}

const db = drizzle(process.env.DB_URL, { logger: new MyLogger() });
```

## Multi-project schema

**Table creator** API lets you define customize table names.  
It’s very useful when you need to keep schemas of different projects in one database.

```ts
import { int, text, mysqlTableCreator } from 'drizzle-orm/mysql-core';

const mysqlTable = mysqlTableCreator((name) => \`project1_${name}\`);

export const users = mysqlTable('users', {
  id: int().primaryKey(),
  name: text().notNull(),
});
```
```ts
import { defineConfig } from "drizzle-kit";

export default defineConfig({
  schema: "./src/schema/*",
  out: "./drizzle",
  dialect: "mysql", 
  dbCredentials: {
    url: process.env.DATABASE_URL,
  }
  tablesFilter: ["project1_*"],
});
```

You can apply multiple `or` filters:

```ts
tablesFilter: ["project1_*", "project2_*"]
```

## Printing SQL query

You can print SQL queries with `db` instance or by using **[`standalone query builder`](#standalone-query-builder)**.

```typescript
const query = db
  .select({ id: users.id, name: users.name })
  .from(users)
  .groupBy(users.id)
  .toSQL();
// query:
{
  sql: 'select \`id\`, \`name\` from \`users\` group by \`users\`.\`id\`',
  params: [],
}
```

## Raw SQL queries execution

If you have some complex queries to execute and `drizzle-orm` can’t handle them yet, you can use the `db.execute` method to execute raw `parametrized` queries.

```typescript
import { ..., type MySqlRawQueryResult } from "drizzle-orm/mysql2";

const userId = 11;
const statement = sql\`select * from ${users} where ${users.id} = ${userId}\`;

const res: MySqlRawQueryResult = await db.execute(statement);
```

## Standalone query builder

Drizzle ORM provides a standalone query builder that allows you to build queries without creating a database instance and get generated SQL.

```typescript
import { QueryBuilder } from 'drizzle-orm/mysql-core';

const qb = new QueryBuilder();

const query = qb.select().from(users).where(eq(users.name, 'Dan'));
const { sql, params } = query.toSQL();
```

## Get typed columns

You can get a typed columns map, very useful when you need to omit certain columns upon selection.

```ts
import { getColumns } from "drizzle-orm";
import { user } from "./schema";

const { password, role, ...rest } = getColumns(user);

await db.select({ ...rest }).from(user);
```

## Get table information

```ts
import { getTableConfig, mysqlTable } from 'drizzle-orm/mysql-core';

export const table = mysqlTable(...);

const {
  columns,
  indexes,
  foreignKeys,
  checks,
  primaryKeys,
  name,
  schema,
} = getTableConfig(table);
```

## Compare objects types (instanceof alternative)

You can check if an object is of a specific Drizzle type using the `is()` function. You can use it with any available type in Drizzle.

IMPORTANT

You should always use `is()` instead of `instanceof`

**Few examples**

```ts
import { Column, is } from 'drizzle-orm';

if (is(value, Column)) {
  // value's type is narrowed to Column
}
```

### Mock Driver

This API is a successor to an undefined `drizzle({} as any)` API which we’ve used internally in Drizzle tests and rarely recommended to external developers.

We decided to build and expose a proper API, every `drizzle` driver now has `drizzle.mock()`:

```ts
import { drizzle } from "drizzle-orm/mysql2";

const db = drizzle.mock();
```

you can provide schema if necessary for types

```ts
import { drizzle } from "drizzle-orm/mysql2";
import { relations } from "./relations"

const db = drizzle.mock({ relations  });
```
