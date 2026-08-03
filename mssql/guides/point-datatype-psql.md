---
url: https://orm.drizzle.team/docs/mssql/guides/point-datatype-psql
title: "Point Datatype Psql"
description: ""
access_date: 2026-08-03T19:38:26.356Z
current_date: 2026-08-03T19:38:26.356Z
---

## Point datatype in PostgreSQL

This guide assumes familiarity with:

- Get started with [PostgreSQL](../../get-started-postgresql.md)
- [Point datatype](../../column-types.md#point)
- [Filtering in select statement](../../select.md#filtering)
- [sql operator](../../sql.md)

PostgreSQL has a special datatype to store geometric data called `point`. It is used to represent a point in a two-dimensional space. The point datatype is represented as a pair of `(x, y)` coordinates. The point expects to receive longitude first, followed by latitude.

```ts
import { sql } from 'drizzle-orm';

const db = drizzle(...);

await db.execute(
  sql\`select point(-90.9, 18.7)\`,
);
```
```json
[ 
  { 
    point: '(-90.9,18.7)' 
  }
]
```

This is how you can create table with `point` datatype in Drizzle:

```ts
import { pgTable, point, serial, text } from 'drizzle-orm/pg-core';

export const stores = pgTable('stores', {
  id: serial('id').primaryKey(),
  name: text('name').notNull(),
  location: point('location', { mode: 'xy' }).notNull(),
});
```

This is how you can insert point data into the table in Drizzle:

```ts
// mode: 'xy'
await db.insert(stores).values({
  name: 'Test',
  location: { x: -90.9, y: 18.7 },
});

// mode: 'tuple'
await db.insert(stores).values({
  name: 'Test',
  location: [-90.9, 18.7],
});

// sql raw
await db.insert(stores).values({
  name: 'Test',
  location: sql\`point(-90.9, 18.7)\`,
});
```

To compute the distance between the objects you can use `<->` operator. This is how you can query for the nearest location by coordinates in Drizzle:

IMPORTANT

`getColumns` available starting from `drizzle-orm@1.0.0-beta.2` (read more [here](../../upgrade-v1.md))

If you are on pre-1 version(like `0.45.1`) then use `getTableColumns`

```ts
import { getColumns, sql } from 'drizzle-orm';
import { stores } from './schema';

const point = {
  x: -73.935_242,
  y: 40.730_61,
};

const sqlDistance = sql\`location <-> point(${point.x}, ${point.y})\`;

await db
  .select({
    ...getColumns(stores),
    distance: sql\`round((${sqlDistance})::numeric, 2)\`,
  })
  .from(stores)
  .orderBy(sqlDistance)
  .limit(1);
```
```sql
select *, round((location <-> point(-73.935242, 40.73061))::numeric, 2)
from stores order by location <-> point(-73.935242, 40.73061)
limit 1;
```

To filter rows to include only those where a `point` type `location` falls within a specified rectangular boundary defined by two diagonal points you can user `<@` operator. It checks if the first object is contained in or on the second object:

```ts
const point = {
  x1: -88,
  x2: -73,
  y1: 40,
  y2: 43,
};

await db
  .select()
  .from(stores)
  .where(
    sql\`${stores.location} <@ box(point(${point.x1}, ${point.y1}), point(${point.x2}, ${point.y2}))\`
  );
```
```sql
select * from stores where location <@ box(point(-88, 40), point(-73, 43));
```
