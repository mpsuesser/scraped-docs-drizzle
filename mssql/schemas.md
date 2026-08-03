---
url: https://orm.drizzle.team/docs/mssql/schemas
title: "Schemas"
description: ""
access_date: 2026-08-03T19:38:26.356Z
current_date: 2026-08-03T19:38:26.356Z
---

## Table schemas

If you declare an entity within a schema, query builder will prepend schema names in queries:  
`select * from [schema].[users]`

```ts
import { int, text, mssqlSchema, nvarchar } from 'drizzle-orm/mssql-core';

export const mySchema = mssqlSchema('my_schema');

export const mySchemaUsers = mySchema.table('users', {
  id: int().identity().primaryKey(),
  name: text(),
  color: nvarchar({ length: 100 }).default('red'),
});
```
```sql
CREATE SCHEMA [my_schema];

CREATE TABLE [my_schema].[users] (
  [id] int IDENTITY(1, 1),
  [name] text,
  [color] nvarchar(100) CONSTRAINT [users_color_default] DEFAULT ('red'),
  CONSTRAINT [users_pkey] PRIMARY KEY([id])
);
```
