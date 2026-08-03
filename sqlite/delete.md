---
url: https://orm.drizzle.team/docs/sqlite/delete
title: "Delete"
description: ""
access_date: 2026-08-03T19:08:17.353Z
current_date: 2026-08-03T19:08:17.353Z
---

## SQL Delete

You can delete all rows in the table:

```typescript
await db.delete(users);
```

And you can delete with filters and conditions:

```typescript
await db.delete(users).where(eq(users.name, 'Dan'));
```

### Limit

Use `.limit()` to add `limit` clause to the query - for example:

```typescript
await db.delete(users).where(eq(users.name, 'Dan')).limit(2);
```
```sql
delete from "users" where "users"."name" = ? limit ?;
```

### Order By

Use `.orderBy()` to add `order by` clause to the query, sorting the results by the specified fields:

```typescript
import { asc, desc } from 'drizzle-orm';

await db.delete(users).where(eq(users.name, 'Dan')).orderBy(users.name);
await db.delete(users).where(eq(users.name, 'Dan')).orderBy(desc(users.name));

// order by multiple fields
await db.delete(users).where(eq(users.name, 'Dan')).orderBy(users.name, users.name2);
await db.delete(users).where(eq(users.name, 'Dan')).orderBy(asc(users.name), desc(users.name2));
```
```sql
delete from "users" where "users"."name" = ? order by "name";
delete from "users" where "users"."name" = ? order by "name" desc;

delete from "users" where "users"."name" = ? order by "name", "name2";
delete from "users" where "users"."name" = ? order by "name" asc, "name2" desc;
```

### Returning

You can delete a row and get it back in SQLite:

```typescript
const deletedUser = await db.delete(users)
  .where(eq(users.name, 'Dan'))
  .returning();

// partial return
const deletedUserIds: { deletedId: number }[] = await db.delete(users)
  .where(eq(users.name, 'Dan'))
  .returning({ deletedId: users.id });
```

## WITH DELETE clause

Check how to use WITH statement with [select](select.md#with-clause), [insert](insert.md#with-insert-clause), [update](update.md#with-update-clause)

Using the `with` clause can help you simplify complex queries by splitting them into smaller subqueries called common table expressions (CTEs):

```typescript
const averageAmount = db.$with('average_amount').as(
  db.select({ value: sql\`avg(${orders.amount})\`.as('value') }).from(orders)
);

const result = await db
    .with(averageAmount)
    .delete(orders)
    .where(gt(orders.amount, sql\`(select * from ${averageAmount})\`))
    .returning({
        id: orders.id
    });
```
```sql
with "average_amount" as (select avg("amount") as "value" from "orders") 
delete from "orders" 
where "orders"."amount" > (select * from "average_amount") 
returning "id"
```
