---
url: https://orm.drizzle.team/docs/cockroach/schemas
title: "Schemas"
description: ""
access_date: 2026-08-03T18:54:19.022Z
current_date: 2026-08-03T18:54:19.022Z
---

## Table schemas

If you declare an entity within a schema, query builder will prepend schema names in queries:  
`select * from "schema"."users"`

```ts
import { int4, string, cockroachSchema } from "drizzle-orm/cockroach-core";

export const mySchema = cockroachSchema("my_schema");

export const colors = mySchema.enum('colors', ['red', 'green', 'blue']);

export const mySchemaUsers = mySchema.table('users', {
  id: int4('id').primaryKey(),
  name: string('name'),
  color: colors('color').default('red'),
});
```
```sql
CREATE SCHEMA "my_schema";

CREATE TYPE "my_schema"."colors" AS ENUM('red', 'green', 'blue');

CREATE TABLE "my_schema"."users" (
    "id" int4 PRIMARY KEY,
    "name" string,
    "color" "my_schema"."colors" DEFAULT 'red'::"my_schema"."colors"
);
```
