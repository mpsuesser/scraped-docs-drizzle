---
url: https://orm.drizzle.team/docs/aliases
title: "Aliases"
description: ""
access_date: 2026-08-03T19:00:22.305Z
current_date: 2026-08-03T19:00:22.305Z
---

## Aliases

Drizzle supports aliasing for tables, columns, subqueries and CTEs — mapping directly to the SQL `AS` keyword.

### Table aliasing

You can alias tables using the `alias` function. This is useful when you need to join the same table multiple times, for example in self-referencing relationships like employees and their managers:

```ts
import { alias, pgTable, integer, text } from "drizzle-orm/pg-core";

const employees = pgTable("employees", {
  id: integer().primaryKey(),
  name: text(),
  managerId: integer("manager_id"),
});

const manager = alias(employees, "manager");

await db.select({
  employeeName: employees.name,
  managerName: manager.name,
})
  .from(employees)
  .leftJoin(manager, eq(employees.managerId, manager.id));
```
```sql
select "employees"."name", "manager"."name"
from "employees"
left join "employees" as "manager" on "employees"."manager_id" = "manager"."id";
```

### Column aliasing

You can alias columns using the `.as()` method on columns and `sql` expressions. This maps directly to the SQL `AS` keyword and lets you control the name of a column in the query result:

```ts
const result = await db.select({
  id: users.id,
  lowerName: users.name.as("lower_name"),
}).from(users);
```
```sql
select "id", "name" as "lower_name" from "users";
```

You can also use `.as()` with `sql` expressions:

```ts
const result = await db.select({
  id: users.id,
  lowerName: sql<string>\`lower(${users.name})\`.as("lower_name"),
}).from(users);
```
```sql
select "id", lower("name") as "lower_name" from "users";
```

### Subquery aliasing

When using a subquery as a data source, you must provide an alias using `.as()`. This lets you reference the subquery’s columns in the outer query:

```ts
const sq = db.select().from(users).where(eq(users.id, 42)).as('sq');

const result = await db.select().from(sq);
```
```sql
select "id", "name", "age" from (select "id", "name", "age" from "users" where "users"."id" = 42) "sq";
```

Subqueries can also be used in joins:

```ts
const sq = db.select().from(users).where(eq(users.id, 42)).as('sq');

const result = await db.select().from(users).leftJoin(sq, eq(users.id, sq.id));
```
```sql
select "users"."id", "users"."name", "users"."age", "sq"."id", "sq"."name", "sq"."age"
  from "users"
  left join (select "id", "name", "age" from "users" where "users"."id" = 42) "sq"
    on "users"."id" = "sq"."id";
```

### CTE aliasing

Common table expressions (CTEs) are aliased using `db.$with('alias')`. The alias name is used to reference the CTE in the main query:

```ts
const sq = db.$with('sq').as(db.select().from(users).where(eq(users.id, 42)));

const result = await db.with(sq).select().from(sq);
```
```sql
with "sq" as (select "id", "name", "age" from "users" where "users"."id" = 42)
select "id", "name", "age" from "sq";
```

When using `sql` expressions inside a CTE, you need to alias them with `.as()` to be able to reference them in the outer query:

```ts
const sq = db.$with('sq').as(db.select({
  name: sql<string>\`upper(${users.name})\`.as('name'),
}).from(users));

const result = await db.with(sq).select({ name: sq.name }).from(sq);
```
```sql
with "sq" as (select upper("name") as "name" from "users")
select "name" from "sq";
```
