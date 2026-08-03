---
url: https://orm.drizzle.team/docs/cockroach/guides/count-rows
title: "Count Rows"
description: ""
access_date: 2026-08-03T19:38:26.356Z
current_date: 2026-08-03T19:38:26.356Z
---

## Count rows

PostgreSQL

MySQL

SQLite

MSSQL

Cockroach

This guide assumes familiarity with:

- Get started with [PostgreSQL](../../get-started-postgresql.md), [MySQL](../../mysql/get-started-mysql.md), [SQLite](../../sqlite/get-started-sqlite.md), [MSSQL](../../mssql/get-started-mssql.md) and [Cockroach](../get-started-cockroach.md)
- [Select statement](../../select.md)
- [Filters](../../operators.md) and [sql operator](../../sql.md)
- [Aggregations](../../select.md#aggregations) and [Aggregation helpers](../../select.md#aggregations-helpers)
- [Joins](../../joins.md)

To count all rows in table you can use `count()` function or `sql` operator like below:

index.ts

schema.ts

```ts
import { count, sql } from 'drizzle-orm';
import { products } from './schema';

const db = drizzle(...);

await db.select({ count: count() }).from(products);

// Under the hood, the count() function casts its result to a number at runtime.
await db.select({ count: sql\`count(*)\`.mapWith(Number) }).from(products);
```
```ts
// result type
type Result = {
  count: number;
}[];
```
```sql
select count(*) from products;
```

To count rows where the specified column contains non-NULL values you can use `count()` function with a column:

```ts
await db.select({ count: count(products.discount) }).from(products);
```
```ts
// result type
type Result = {
  count: number;
}[];
```
```sql
select count("discount") from products;
```

Drizzle has simple and flexible API, which lets you create your custom solutions. In PostgreSQL, MySQL and Cockroach `count()` function returns bigint, which is interpreted as string by their drivers, so it should be casted to integer:

```ts
import { AnyColumn, sql } from 'drizzle-orm';

const customCount = (column?: AnyColumn) => {
  if (column) {
    return sql<number>\`cast(count(${column}) as integer)\`; // In MySQL cast to unsigned integer
  } else {
    return sql<number>\`cast(count(*) as integer)\`; // In MySQL cast to unsigned integer
  }
};

await db.select({ count: customCount() }).from(products);
await db.select({ count: customCount(products.discount) }).from(products);
```
```sql
select cast(count(*) as integer) from products;
select cast(count("discount") as integer) from products;
```

In SQLite and MSSQL, `count()` result returns as integer.

```ts
import { sql } from 'drizzle-orm';

await db.select({ count: sql<number>\`count(*)\` }).from(products);
await db.select({ count: sql<number>\`count(${products.discount})\` }).from(products);
```
```sql
select count(*) from products;
select count("discount") from products;
```

IMPORTANT

By specifying `sql<number>`, you are telling Drizzle that the **expected** type of the field is `number`.  
If you specify it incorrectly (e.g. use `sql<string>` for a field that will be returned as a number), the runtime value won’t match the expected type. Drizzle cannot perform any type casts based on the provided type generic, because that information is not available at runtime.

If you need to apply runtime transformations to the returned value, you can use the [`.mapWith()`](../../sql.md#sqlmapwith) method.

To count rows that match a condition you can use `.where()` method:

```ts
import { count, gt } from 'drizzle-orm';

await db
  .select({ count: count() })
  .from(products)
  .where(gt(products.price, 100));
```
```sql
select count(*) from products where price > 100
```

This is how you can use `count()` function with joins and aggregations:

index.ts

schema.ts

```ts
import { count, eq } from 'drizzle-orm';
import { countries, cities } from './schema';

// Count cities in each country
await db
  .select({
    country: countries.name,
    citiesCount: count(cities.id),
  })
  .from(countries)
  .leftJoin(cities, eq(countries.id, cities.countryId))
  .groupBy(countries.id)
  .orderBy(countries.name);
```
```sql
select countries.name, count("cities"."id") from countries
  left join cities on countries.id = cities.country_id
  group by countries.id
  order by countries.name;
```
