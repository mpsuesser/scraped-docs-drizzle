---
url: https://orm.drizzle.team/docs/singlestore/schemas
title: "Schemas"
description: ""
access_date: 2026-08-03T19:38:26.356Z
current_date: 2026-08-03T19:38:26.356Z
---

## Table schemas

If you declare an entity within a schema, query builder will prepend schema names in queries:  
“select \* from `schema`.\`users\`\`\`

```ts
import { int, text, singlestoreSchema } from "drizzle-orm/singlestore-core";

export const mySchema = singlestoreSchema("my_schema")

export const mySchemaUsers = mySchema.table("users", {
  id: int("id").primaryKey().autoincrement(),
  name: text("name"),
});
```
```sql
CREATE SCHEMA \`my_schema\`;

CREATE TABLE \`my_schema\`.\`users\` (
  \`id\` serial PRIMARY KEY,
  \`name\` text
);
```
