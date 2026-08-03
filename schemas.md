---
url: https://orm.drizzle.team/docs/schemas
title: "Schemas"
description: ""
access_date: 2026-08-03T18:54:19.022Z
current_date: 2026-08-03T18:54:19.022Z
---

## Table schemas

If you declare an entity within a schema, query builder will prepend schema names in queries:  
`select * from "schema"."users"`

```ts
import { serial, text, pgSchema } from "drizzle-orm/pg-core";

export const mySchema = pgSchema("my_schema");

export const colors = mySchema.enum('colors', ['red', 'green', 'blue']);

export const mySchemaUsers = mySchema.table('users', {
  id: serial('id').primaryKey(),
  name: text('name'),
  color: colors('color').default('red'),
});
```
```sql
CREATE SCHEMA "my_schema";

CREATE TYPE "my_schema"."colors" AS ENUM ('red', 'green', 'blue');

CREATE TABLE "my_schema"."users" (
  "id" serial PRIMARY KEY,
  "name" text,
  "color" "my_schema"."colors" DEFAULT 'red'
);
```
