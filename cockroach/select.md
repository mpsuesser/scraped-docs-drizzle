---
url: https://orm.drizzle.team/docs/cockroach/select
title: "Select"
description: ""
access_date: 2026-08-03T19:00:22.305Z
current_date: 2026-08-03T19:00:22.305Z
---

## SQL Select

Drizzle provides you the most SQL-like way to fetch data from your database, while remaining type-safe and composable. It natively supports mostly every query feature and capability of every dialect, and whatever it doesn’t support yet, can be added by the user with the powerful [sql](sql.md) operator.

For the following examples, let’s assume you have a `users` table defined like this:

```typescript
import { cockroachTable, serial, string } from 'drizzle-orm/cockroach-core';

export const users = cockroachTable('users', {
  id: int4().primaryKey(),
  name: string().notNull(),
  age: int4(),
});
```

### Basic select

Select all rows from a table including all columns:

```typescript
const result = await db.select().from(users);
/*
  {
    id: number;
    name: string;
    age: number | null;
  }[]
*/
```
```sql
select "id", "name", "age" from "users";
```

Notice that the result type is inferred automatically based on the table definition, including columns nullability.

Drizzle always explicitly lists columns in the `select` clause instead of using `select *`.  
This is required internally to guarantee the fields order in the query result, and is also generally considered a good practice.

### Partial select

In some cases, you might want to select only a subset of columns from a table. You can do that by providing a selection object to the `.select()` method:

```typescript
const result = await db.select({
  field1: users.id,
  field2: users.name,
}).from(users);

const { field1, field2 } = result[0];
```
```sql
select "id", "name" from "users";
```

Like in SQL, you can use arbitrary expressions as selection fields, not just table columns:

```typescript
const result = await db.select({
  id: users.id,
  lowerName: sql<string>\`lower(${users.name})\`,
}).from(users);
```
```sql
select "id", lower("name") from "users";
```

IMPORTANT

By specifying `sql<string>`, you are telling Drizzle that the **expected** type of the field is `string`.  
If you specify it incorrectly (e.g. use `sql<number>` for a field that will be returned as a string), the runtime value won’t match the expected type. Drizzle cannot perform any type casts based on the provided type generic, because that information is not available at runtime.

If you need to apply runtime transformations to the returned value, you can use the [`.mapWith()`](sql.md#sqlmapwith) method.

### Conditional select

You can have a dynamic selection object based on some condition:

```typescript
async function selectUsers(withName: boolean) {
  return db
    .select({
      id: users.id,
      ...(withName ? { name: users.name } : {}),
    })
    .from(users);
}

const result = await selectUsers(true);
```

### Distinct select

You can use `.selectDistinct()` instead of `.select()` to retrieve only unique rows from a dataset:

```ts
await db.selectDistinct().from(users).orderBy(users.id, users.name);

await db.selectDistinct({ id: users.id }).from(users).orderBy(users.id);
```
```sql
select distinct "id", "name", "age" from "users" order by "users"."id", "users"."name";

select distinct "id" from "users" order by "users"."id";
```

In CockroachDB, you can also use the `distinct on` clause to specify how the unique rows are determined:

```ts
await db.selectDistinctOn([users.id]).from(users).orderBy(users.id);
await db.selectDistinctOn([users.name], { name: users.name }).from(users).orderBy(users.name);
```
```sql
select distinct on ("users"."id") "id", "name", "age" from "users" order by "users"."id";
select distinct on ("users"."name") "name" from "users" order by "users"."name";
```

### Advanced select

Powered by TypeScript, Drizzle APIs let you build your select queries in a variety of flexible ways.

Sneak peek of advanced partial select, for more detailed advanced usage examples - see our [dedicated guide](guides/include-or-exclude-columns.md).

```ts
import { getColumns, sql } from 'drizzle-orm';

await db.select({
    ...getColumns(posts),
    titleLength: sql<number>\`length(${posts.title})\`,
  }).from(posts);
```

## \---

### Filters

You can filter the query results using the [filter operators](operators.md) in the `.where()` method:

```typescript
import { eq, lt, gte, ne } from 'drizzle-orm';

await db.select().from(users).where(eq(users.id, 42));
await db.select().from(users).where(lt(users.id, 42));
await db.select().from(users).where(gte(users.id, 42));
await db.select().from(users).where(ne(users.id, 42));
...
```
```sql
select "id", "name", "age" from "users" where "users"."id" = 42;
select "id", "name", "age" from "users" where "users"."id" < 42;
select "id", "name", "age" from "users" where "users"."id" >= 42;
select "id", "name", "age" from "users" where "users"."id" <> 42;
```

All filter operators are implemented using the [`sql`](sql.md) function. You can use it yourself to write arbitrary SQL filters, or build your own operators. For inspiration, you can check how the operators provided by Drizzle are [implemented](https://github.com/drizzle-team/drizzle-orm/blob/main/drizzle-orm/src/sql/expressions/conditions.ts).

```typescript
import { sql } from 'drizzle-orm';
import type { CockroachColumn } from "drizzle-orm/cockroach-core";

function equals42(col: CockroachColumn) {
  return sql\`${col} = 42\`;
}

await db.select().from(users).where(sql\`${users.id} < 42\`);
await db.select().from(users).where(sql\`${users.id} = 42\`);
await db.select().from(users).where(equals42(users.id));
await db.select().from(users).where(sql\`${users.id} >= 42\`);
await db.select().from(users).where(sql\`${users.id} <> 42\`);
await db.select().from(users).where(sql\`lower(${users.name}) = 'aaron'\`);
```
```sql
select "id", "name", "age" from "users" where "users"."id" < 42;
select "id", "name", "age" from "users" where "users"."id" = 42;
select "id", "name", "age" from "users" where "users"."id" = 42;
select "id", "name", "age" from "users" where "users"."id" >= 42;
select "id", "name", "age" from "users" where "users"."id" <> 42;
select "id", "name", "age" from "users" where lower("users"."name") = 'aaron';
```

All the values provided to filter operators and to the `sql` function are parameterized automatically. For example, this query:

```ts
await db.select().from(users).where(eq(users.id, 42));
```

will be translated to:

```sql
select "id", "name", "age" from "users" where "users"."id" = $1; -- params: [42]
```

Inverting condition with a `not` operator:

```typescript
import { eq, not, sql } from 'drizzle-orm';

await db.select().from(users).where(not(eq(users.id, 42)));
await db.select().from(users).where(sql\`not ${users.id} = 42\`);
```
```sql
select "id", "name", "age" from "users" where not ("users"."id" = 42);
select "id", "name", "age" from "users" where not "users"."id" = 42;
```

You can safely alter schema, rename tables and columns and it will be automatically reflected in your queries because of template interpolation, as opposed to hardcoding column or table names when writing raw SQL.

### Combining filters

You can logically combine filter operators with `and()` and `or()` operators:

```typescript
import { eq, and, sql } from 'drizzle-orm';

await db.select().from(users).where(
  and(
    eq(users.id, 42),
    eq(users.name, 'Dan')
  )
);
await db.select().from(users).where(sql\`${users.id} = 42 and ${users.name} = 'Dan'\`);
```
```sql
select "id", "name", "age" from "users" where (("users"."id" = 42) and ("users"."name" = 'Dan'));
select "id", "name", "age" from "users" where "users"."id" = 42 and "users"."name" = 'Dan';
```

```typescript
import { eq, or, sql } from 'drizzle-orm';

await db.select().from(users).where(
  or(
    eq(users.id, 42), 
    eq(users.name, 'Dan')
  )
);
await db.select().from(users).where(sql\`${users.id} = 42 or ${users.name} = 'Dan'\`);
```
```sql
select "id", "name", "age" from "users" where (("users"."id" = 42) or ("users"."name" = 'Dan'));
select "id", "name", "age" from "users" where "users"."id" = 42 or "users"."name" = 'Dan';
```

### Advanced filters

In combination with TypeScript, Drizzle APIs provide you powerful and flexible ways to combine filters in queries.

Sneak peek of conditional filtering, for more detailed advanced usage examples - see our [dedicated guide](guides/conditional-filters-in-query.md).

```ts
const searchPosts = async (term?: string) => {
  await db
    .select()
    .from(posts)
    .where(term ? ilike(posts.title, term) : undefined);
};
await searchPosts();
await searchPosts('AI');
```

## \---

### Limit & offset

Use `.limit()` and `.offset()` to add limit and offset clauses to the query - for example, to implement pagination:

```ts
await db.select().from(users).limit(10);
await db.select().from(users).limit(10).offset(10);
```
```sql
select "id", "name", "age" from "users" limit 10;
select "id", "name", "age" from "users" limit 10 offset 10;
```

### Order By

Use `.orderBy()` to add `order by` clause to the query, sorting the results by the specified fields:

```typescript
import { asc, desc } from 'drizzle-orm';

await db.select().from(users).orderBy(users.name);
await db.select().from(users).orderBy(desc(users.name));

// order by multiple fields
await db.select().from(users).orderBy(users.name, users.name2);
await db.select().from(users).orderBy(asc(users.name), desc(users.name2));
```
```sql
select "id", "name", "name2", "age" from "users" order by "users"."name";
select "id", "name", "name2", "age" from "users" order by "users"."name" desc;

select "id", "name", "name2", "age" from "users" order by "users"."name", "users"."name";
select "id", "name", "name2", "age" from "users" order by "users"."name" asc, "users"."name2" desc;
```

### Advanced pagination

Powered by TypeScript, Drizzle APIs let you implement all possible SQL pagination and sorting approaches.

Sneak peek of advanced pagination, for more detailed advanced usage examples - see our dedicated [limit offset pagination](guides/limit-offset-pagination.md) and [cursor pagination](guides/cursor-based-pagination.md) guides.

```ts
await db
  .select()
  .from(users)
  .orderBy(asc(users.id)) // order by is mandatory
  .limit(4) // the number of rows to return
  .offset(4); // the number of rows to skip
```

## \---

### WITH clause

Check how to use WITH statement with [insert](insert.md#with-insert-clause), [update](update.md#with-update-clause), [delete](delete.md#with-delete-clause)

Using the `with` clause can help you simplify complex queries by splitting them into smaller subqueries called common table expressions (CTEs):

```typescript
const sq = db.$with('sq').as(db.select().from(users).where(eq(users.id, 42)));

const result = await db.with(sq).select().from(sq);
```
```sql
with "sq" as (select "id", "name", "age" from "users" where "users"."id" = 42)
select "id", "name", "age" from "sq";
```

You can also provide `insert`, `update` and `delete` statements inside `with`

```typescript
const sq = db.$with('sq').as(
    db.insert(users).values({ name: 'John' }).returning(),
);

const result = await db.with(sq).select().from(sq);
```
```sql
with "sq" as (insert into "users" ("id", "name", "age") values (default, 'John', default) returning "id", "name", "age")
select "id", "name", "age" from "sq";
```

```typescript
const sq = db.$with('sq').as(
    db.update(users).set({ age: 25 }).where(eq(users.name, 'John')).returning(),
);
const result = await db.with(sq).select().from(sq);
```
```sql
with "sq" as (update "users" set "age" = 25 where "users"."name" = 'John' returning "id", "name", "age") 
select "id", "name", "age" from "sq";
```

```typescript
const sq = db.$with('sq').as(
  db.delete(users).where(eq(users.name, 'John')).returning(),
);

const result = await db.with(sq).select().from(sq);
```
```sql
with "sq" as (delete from "users" where "users"."name" = $1 returning "id", "name", "age") 
select "id", "name", "age" from "sq";
```

To select arbitrary SQL values as fields in a CTE and reference them in other CTEs or in the main query, you need to add aliases to them:

```typescript
const sq = db.$with('sq').as(db.select({ 
  name: sql<string>\`upper(${users.name})\`.as('name'),
})
.from(users));

const result = await db.with(sq).select({ name: sq.name }).from(sq);
```

If you don’t provide an alias, the field type will become `DrizzleTypeError` and you won’t be able to reference it in other queries. If you ignore the type error and still try to use the field, you will get a runtime error, since there’s no way to reference that field without an alias.

### Select from subquery

Just like in SQL, you can embed queries into other queries by using the subquery API:

```typescript
const sq = db.select().from(users).where(eq(users.id, 42)).as('sq');
const result = await db.select().from(sq);
```
```sql
select "id", "name", "age" from (select "id", "name", "age" from "users" where "users"."id" = 42) "sq";
```

Subqueries can be used in any place where a table can be used, for example in joins:

```typescript
const sq = db.select().from(users).where(eq(users.id, 42)).as('sq');
const result = await db.select().from(users).leftJoin(sq, eq(users.id, sq.id));
```
```sql
select "users"."id", "users"."name", "users"."age", "sq"."id", "sq"."name", "sq"."age" from "users"
  left join (select "id", "name", "age" from "users" where "users"."id" = 42) "sq"
    on "users"."id" = "sq"."id";
```

## \---

### Aggregations

With Drizzle, you can do aggregations using functions like `sum`, `count`, `avg`, etc. by grouping and filtering with `.groupBy()` and `.having()` respectfully, same as you would do in raw SQL:

```typescript
import { gt, sql } from "drizzle-orm";

await db.select({
  age: users.age,
  count: sql<number>\`cast(count(${users.id}) as int)\`,
})
  .from(users)
  .groupBy(users.age);

await db.select({
  age: users.age,
  count: sql<number>\`cast(count(${users.id}) as int)\`,
})
  .from(users)
  .groupBy(users.age)
  .having(({ count }) => gt(count, 1));
```
```sql
select "age", cast(count("id") as int)
  from "users"
  group by "users"."age";

select "age", cast(count("id") as int)
  from "users"
  group by "users"."age"
  having cast(count("users"."id") as int) > 1;
```

Alternatively, you can use [`.mapWith(Number)`](sql.md#sqlmapwith) to cast the value to a number at runtime.

If you need count aggregation - we recommend using our [$count](select.md#count) API

### Aggregations helpers

Drizzle has a set of wrapped `sql` functions, so you don’t need to write `sql` templates for common cases in your app

Remember, aggregation functions are often used with the GROUP BY clause of the SELECT statement. So if you are selecting using aggregating functions and other columns in one query, be sure to use the `.groupBy` clause

**count**

Returns the number of values in `expression`.

```ts
import { count } from 'drizzle-orm'

await db.select({ value: count() }).from(users);
await db.select({ value: count(users.id) }).from(users);
```
```sql
select count(*) from "users";
select count("id") from "users";
```
```ts
// It's equivalent to writing
await db.select({ 
  value: sql\`count(*)\`.mapWith(Number) 
}).from(users);

await db.select({ 
  value: sql\`count(${users.id})\`.mapWith(Number) 
}).from(users);
```

**countDistinct**

Returns the number of non-duplicate values in `expression`.

```ts
import { countDistinct } from 'drizzle-orm'

await db.select({ value: countDistinct(users.id) }).from(users);
```
```sql
select count(distinct "id") from "users";
```
```ts
// It's equivalent to writing
await db.select({ 
  value: sql\`count(distinct ${users.id})\`.mapWith(Number) 
}).from(users);
```

**avg**

Returns the average (arithmetic mean) of all non-null values in `expression`.

```ts
import { avg } from 'drizzle-orm'

await db.select({ value: avg(users.id) }).from(users);
```
```sql
select avg("id") from "users";
```
```ts
// It's equivalent to writing
await db.select({ 
  value: sql\`avg(${users.id})\`.mapWith(String) 
}).from(users);
```

**avgDistinct**

Returns the average (arithmetic mean) of all non-null values in `expression`.

```ts
import { avgDistinct } from 'drizzle-orm'

await db.select({ value: avgDistinct(users.id) }).from(users);
```
```sql
select avg(distinct "id") from "users";
```
```ts
// It's equivalent to writing
await db.select({ 
  value: sql\`avg(distinct ${users.id})\`.mapWith(String) 
}).from(users);
```

**sum**

Returns the sum of all non-null values in `expression`.

```ts
import { sum } from 'drizzle-orm'

await db.select({ value: sum(users.id) }).from(users);
```
```sql
select sum("id") from "users";
```
```ts
// It's equivalent to writing
await db.select({ 
  value: sql\`sum(${users.id})\`.mapWith(String) 
}).from(users);
```

**sumDistinct**

Returns the sum of all non-null and non-duplicate values in `expression`.

```ts
import { sumDistinct } from 'drizzle-orm'

await db.select({ value: sumDistinct(users.id) }).from(users);
```
```sql
select sum(distinct "id") from "users";
```
```ts
// It's equivalent to writing
await db.select({ 
  value: sql\`sum(distinct ${users.id})\`.mapWith(String) 
}).from(users);
```

**max**

Returns the maximum value in `expression`.

```ts
import { max } from 'drizzle-orm'

await db.select({ value: max(users.id) }).from(users);
```
```sql
select max("id") from "users";
```
```ts
// It's equivalent to writing
await db.select({ 
  value: sql\`max(${expression})\`.mapWith(users.id) 
}).from(users);
```

**min**

Returns the minimum value in `expression`.

```ts
import { min } from 'drizzle-orm'

await db.select({ value: min(users.id) }).from(users);
```
```sql
select min("id") from "users";
```
```ts
// It's equivalent to writing
await db.select({ 
  value: sql\`min(${users.id})\`.mapWith(users.id) 
}).from(users);
```

A more advanced example:

```typescript
const orders = cockroachTable("order", {
  id: int4().primaryKey(),
  orderDate: timestamp("order_date", { withTimezone: true }).notNull(),
  requiredDate: timestamp("required_date", { withTimezone: true }).notNull(),
  shippedDate: timestamp("shipped_date", { withTimezone: true }),
  shipVia: int4("ship_via").notNull(),
  freight: numeric("freight").notNull(),
  shipName: string("ship_name").notNull(),
  shipCity: string("ship_city").notNull(),
  shipRegion: string("ship_region"),
  shipPostalCode: string("ship_postal_code"),
  shipCountry: string("ship_country").notNull(),
  customerId: string("customer_id").notNull(),
  employeeId: int4("employee_id").notNull(),
});

const details = cockroachTable("order_detail", {
  unitPrice: numeric("unit_price").notNull(),
  quantity: int4("quantity").notNull(),
  discount: numeric("discount").notNull(),
  orderId: int4("order_id").notNull(),
  productId: int4("product_id").notNull(),
});

await db
  .select({
    id: orders.id,
    shippedDate: orders.shippedDate,
    shipName: orders.shipName,
    shipCity: orders.shipCity,
    shipCountry: orders.shipCountry,
    productsCount: sql<number>\`cast(count(${details.productId}) as int)\`,
    quantitySum: sql<number>\`sum(${details.quantity})\`,
    totalPrice: sql<number>\`sum(${details.quantity} * ${details.unitPrice})\`,
  })
  .from(orders)
  .leftJoin(details, eq(orders.id, details.orderId))
  .groupBy(orders.id)
  .orderBy(asc(orders.id))
```

### $count

`db.$count()` is a utility wrapper of `count(*)`, it is a very flexible operator which can be used as is or as a subquery, more details in our [GitHub discussion](https://github.com/drizzle-team/drizzle-orm/discussions/3119).

```ts
const count = await db.$count(users);
//    ^? number

const count = await db.$count(users, eq(users.name, "Dan")); // works with filters
```
```sql
select count(*) from "users";
select count(*) from "users" where "users"."name" = 'Dan';
```

It is exceptionally useful in [subqueries](select.md#select-from-subquery):

```ts
const users = await db.select({
  ...users,
  postsCount: db.$count(posts, eq(posts.authorId, users.id)),
}).from(users);
```

usage example with [relational queries](rqb.md)

```ts
const users = await db._query.users.findMany({
  extras: {
    postsCount: db.$count(schema.users).as('posts_count'),
  },
});
```
