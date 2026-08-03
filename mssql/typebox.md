---
url: https://orm.drizzle.team/docs/mssql/typebox
title: "Typebox"
description: ""
access_date: 2026-08-03T19:08:17.353Z
current_date: 2026-08-03T19:08:17.353Z
---

## typebox

#### Install the dependencies

```shell
npm i drizzle-orm@rc typebox
```

Allows you to generate [typebox](https://sinclairzx81.github.io/typebox/#/) schemas from Drizzle ORM schemas

**Features**

- Create a select schema for tables.
- Create insert and update schemas for tables.
- Supported dialect: MSSQL.

#### Usage

```ts
import { int, mssqlTable, text, datetime2 } from 'drizzle-orm/mssql-core';
import { createInsertSchema, createSelectSchema, createUpdateSchema } from 'drizzle-orm/typebox';
import { Type } from 'typebox';
import { Value } from 'typebox/value';

const users = mssqlTable('users', {
    id: int().primaryKey().identity(),
    name: text().notNull(),
    email: text().notNull(),
    role: text({ enum: ['admin', 'user'] }).notNull(),
    createdAt: datetime2('created_at').notNull(),
});

// Schema for inserting a user - can be used to validate API requests
const insertUserSchema = createInsertSchema(users);

// Schema for updating a user - can be used to validate API requests
const updateUserSchema = createUpdateSchema(users);

// Schema for selecting a user - can be used to validate API responses
const selectUserSchema = createSelectSchema(users);

// Overriding the fields
const insertUserSchema = createInsertSchema(users, {
    role: Type.String(),
});

// Refining the fields - useful if you want to change the fields before they become nullable/optional in the final schema
const insertUserSchema = createInsertSchema(users, {
    id: (schema) => Type.Number({ ...schema, minimum: 0 }),
    role: Type.String(),
});

// Usage
const isUserValid: boolean = Value.Check(insertUserSchema, {
    name: 'John Doe',
    email: 'johndoe@test.com',
    role: 'admin',
});
```
