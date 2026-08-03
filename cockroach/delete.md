---
url: https://orm.drizzle.team/docs/cockroach/delete
title: "Delete"
description: ""
access_date: 2026-08-03T18:54:19.022Z
current_date: 2026-08-03T18:54:19.022Z
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

### Returning

You can delete a row and get it back in PostgreSQL:

```typescript
const deletedUser = await db.delete(users)
  .where(eq(users.name, 'Dan'))
  .returning();

// partial return
const deletedUserId = await db.delete(users)
  .where(eq(users.name, "Dan"))
  .returning({ deletedId: users.id });

// deletedUserId: { deletedId: number | null }[]
```
```sql
delete from "users" where "users"."name" = 'Dan' returning "id", "name";

delete from "users" where "users"."name" = 'Dan' returning "id";
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
