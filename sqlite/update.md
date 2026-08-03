---
url: https://orm.drizzle.team/docs/sqlite/update
title: "Update"
description: ""
access_date: 2026-08-03T19:08:17.353Z
current_date: 2026-08-03T19:08:17.353Z
---

## SQL Update

All the values provided to `.set()` are parameterized automatically. For example, this query:

```ts
await db.update(users).set({ name: "Mr. Dan" }).where(eq(users.name, "Dan"));
```

will be translated to:

```sql
update "users" set "name" = ? where "users"."name" = ?; -- params: ['Mr. Dan', 'Dan']
```

The object that you pass to `update` should have keys that match column names in your database schema. Values of `undefined` are ignored in the object: to set a column to `null`, pass `null`.

```typescript
await db.update(users)
  .set({ name: 'Mr. Dan' })
  .where(eq(users.name, 'Dan'));
```
```typescript
await db.update(users)
  .set({ name: null })
  .where(eq(users.name, 'Dan'));
```

You can pass SQL as a value to be used in the update object, like this:

```typescript
await db.update(users)
  .set({ updatedAt: sql\`NOW()\` })
  .where(eq(users.name, 'Dan'));
```

### Limit

Use `.limit()` to add `limit` clause to the query - for example:

```typescript
await db.update(usersTable).set({ verified: true }).limit(2);
```
```sql
update "users" set "verified" = ? limit ?;
```

### Order By

Use `.orderBy()` to add `order by` clause to the query, sorting the results by the specified fields:

```typescript
import { asc, desc } from 'drizzle-orm';

await db.update(usersTable).set({ verified: true }).orderBy(usersTable.name);
await db.update(usersTable).set({ verified: true }).orderBy(desc(usersTable.name));

// order by multiple fields
await db.update(usersTable).set({ verified: true }).orderBy(usersTable.name, usersTable.name2);
await db.update(usersTable).set({ verified: true }).orderBy(asc(usersTable.name), desc(usersTable.name2));
```
```sql
update "users" set "verified" = ? order by "name";
update "users" set "verified" = ? order by "name" desc;

update "users" set "verified" = ? order by "name", "name2";
update "users" set "verified" = ? order by "name" asc, "name2" desc;
```

### Returning

You can update a row and get it back in SQLite:

```typescript
const updatedUserId = await db.update(users).set({ name: "Mr. Dan" }).where(eq(users.name, "Dan")).returning({ updatedId: users.id });
//      ^ { updatedId: number | null }[]
```
```sql
update "users" set "name" = 'Mr. Dan' where "users"."name" = 'Dan' returning "id";
```

## with update clause

Check how to use WITH statement with [select](select.md#with-clause), [insert](insert.md#with-insert-clause), [delete](delete.md#with-delete-clause)

Using the `with` clause can help you simplify complex queries by splitting them into smaller subqueries called common table expressions (CTEs):

```typescript
const averagePrice = db.$with('average_price').as(
    db.select({ value: sql\`avg(${products.price})\`.as('value') }).from(products)
);

const result = await db.with(averagePrice)
    .update(products)
    .set({
        cheap: true
    })
    .where(lt(products.price, sql\`(select * from ${averagePrice})\`))
    .returning({
        id: products.id
    });
```
```sql
with "average_price" as (select avg("price") as "value" from "products") 
update "products" set "cheap" = true 
where "products"."price" < (select * from "average_price") 
returning "id"
```

## Update … from

As the SQLite documentation mentions:

> The UPDATE-FROM idea is an extension to SQL that allows an UPDATE statement to be driven by other tables in the database. The “target” table is the specific table that is being updated. With UPDATE-FROM you can join the target table against other tables in the database in order to help compute which rows need updating and what the new values should be on those rows

```ts
await db
  .update(users)
  .set({ cityId: cities.id })
  .from(cities)
  .where(and(eq(cities.name, 'Seattle'), eq(users.name, 'John')))
```
```sql
update "users" set "city_id" = "cities"."id" 
from "cities" 
where (("cities"."name" = 'Seattle') and ("users"."name" = 'John'))
```

You can also alias tables that are joined (you can also alias the updating table too).

```ts
const c = alias(cities, 'c');
await db
  .update(users)
  .set({ cityId: c.id })
  .from(c);
```
```sql
update "users" set "city_id" = "c"."id" 
from "cities" "c"
```

In SQLite, you can also return columns from the joined tables.

```ts
const updatedUsers = await db
  .update(users)
  .set({ cityId: cities.id })
  .from(cities)
  .returning({ id: users.id, cityName: cities.name });
```
```sql
update "users" set "city_id" = "cities"."id" 
from "cities" 
returning "users"."id", "cities"."name"
```
