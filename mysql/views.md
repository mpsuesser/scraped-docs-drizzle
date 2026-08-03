---
url: https://orm.drizzle.team/docs/mysql/views
title: "Views"
description: ""
access_date: 2026-08-03T18:54:19.022Z
current_date: 2026-08-03T18:54:19.022Z
---

## Views

There’re several ways you can declare views with Drizzle ORM.

You can declare views that have to be created or you can declare views that already exist in the database.

You can declare views statements with an inline `query builder` syntax, with `standalone query builder` and with raw `sql` operators.

When views are created with either inlined or standalone query builders, view columns schema will be automatically inferred, yet when you use `sql` you have to explicitly declare view columns schema.

### Declaring views

```ts
import { text, mysqlTable, mysqlView, int, timestamp } from "drizzle-orm/mysql-core";
import { eq } from "drizzle-orm";

export const user = mysqlTable("user", {
  id: int().primaryKey().autoincrement(),
  name: text(),
  email: text(),
  password: text(),
  role: text().$type<"admin" | "customer">(),
  createdAt: timestamp("created_at"),
  updatedAt: timestamp("updated_at"),
});

export const userView = mysqlView("user_view").as((qb) => qb.select().from(user));
export const customersView = mysqlView("customers_view").as((qb) => qb.select().from(user).where(eq(user.role, "customer")));
```
```sql
...

CREATE ALGORITHM = undefined SQL SECURITY definer VIEW \`customers_view\` AS (select * from \`user\` where \`user\`.\`role\` = 'customer');
CREATE ALGORITHM = undefined SQL SECURITY definer VIEW \`user_view\` AS (select * from \`user\`);
```

You can also declare views using `standalone query builder`, it works exactly the same way:

```ts
import { text, mysqlTable, mysqlView, int, timestamp, QueryBuilder } from "drizzle-orm/mysql-core";
import { eq } from "drizzle-orm";

const qb = new QueryBuilder();

export const user = mysqlTable("user", {
  id: int().primaryKey().autoincrement(),
  name: text(),
  email: text(),
  password: text(),
  role: text().$type<"admin" | "customer">(),
  createdAt: timestamp("created_at"),
  updatedAt: timestamp("updated_at"),
});

export const userView = mysqlView("user_view").as(qb.select().from(user));
export const customersView = mysqlView("customers_view").as(qb.select().from(user).where(eq(user.role, "customer")));
```
```sql
...

CREATE ALGORITHM = undefined SQL SECURITY definer VIEW \`customers_view\` AS (select * from \`user\` where \`user\`.\`role\` = 'customer');
CREATE ALGORITHM = undefined SQL SECURITY definer VIEW \`user_view\` AS (select * from \`user\`);
```

### Declaring views with raw SQL

Whenever you need to declare view using a syntax that is not supported by the query builder, you can directly use `sql` operator and explicitly specify view columns schema.

```ts
// regular view
import { eq, sql } from "drizzle-orm";
import { int, mysqlTable, mysqlView, text, timestamp } from "drizzle-orm/mysql-core";

export const users = mysqlTable("users", {
  id: int().primaryKey().autoincrement(),
  name: text(),
  cityId: int("city_id"),
});

export const newYorkers = mysqlView("new_yorkers", {
  id: int().primaryKey().autoincrement(),
  name: text().notNull(),
  cityId: int("city_id").notNull(),
}).as(sql\`select * from ${users} where ${eq(users.cityId, 1)}\`);
```
```sql
...

CREATE ALGORITHM = undefined SQL SECURITY definer VIEW \`new_yorkers\` AS (select * from \`users\` where \`users\`.\`city_id\` = 1);
```

### Declaring existing views

When you’re provided with a read only access to an existing view in the database you should use `.existing()` view configuration, `drizzle-kit` will ignore and will not generate a `create view` statement in the generated migration.

```ts
import { int, mysqlTable, mysqlView, text, timestamp } from "drizzle-orm/mysql-core";

export const user = mysqlTable("user", {
  id: int().primaryKey().autoincrement(),
  name: text(),
  email: text(),
  password: text(),
  role: text().$type<"admin" | "customer">(),
  createdAt: timestamp("created_at"),
  updatedAt: timestamp("updated_at"),
});

// regular view
export const trimmedUser = mysqlView("trimmed_user", {
  id: int("id"),
  name: text("name"),
  email: text("email"),
}).existing();
```
