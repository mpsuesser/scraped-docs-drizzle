---
url: https://orm.drizzle.team/docs/cockroach/guides/postgis-geometry-point
title: "Postgis Geometry Point"
description: ""
access_date: 2026-08-03T18:54:19.022Z
current_date: 2026-08-03T18:54:19.022Z
---

## PostGIS geometry point

This guide assumes familiarity with:

- Get started with [PostgreSQL](../../get-started-postgresql.md)
- [Postgis extension](../../extensions.md#postgis)
- [Indexes](../../indexes-constraints.md#indexes)
- [Filtering in select statement](../../select.md#filtering)
- [sql operator](../../sql.md)

`PostGIS` extends the capabilities of the PostgreSQL relational database by adding support for storing, indexing, and querying geospatial data.

As for now, Drizzle doesn’t create extension automatically, so you need to create it manually. Create an empty migration file and add SQL query:

```bash
npx drizzle-kit generate --custom
```
```sql
CREATE EXTENSION postgis;
```

This is how you can create table with `geometry` datatype and spatial index in Drizzle:

schema.ts

migration.sql

```ts
import { geometry, index, pgTable, serial, text } from 'drizzle-orm/pg-core';

export const stores = pgTable(
  'stores',
  {
    id: serial('id').primaryKey(),
    name: text('name').notNull(),
    location: geometry('location', { type: 'point', mode: 'xy', srid: 4326 }).notNull(),
  },
  (t) => [
    index('spatial_index').using('gist', t.location),
  ]
);
```

This is how you can insert `geometry` data into the table in Drizzle. `ST_MakePoint()` in PostGIS creates a geometric object of type `point` using the specified coordinates. `ST_SetSRID()` sets the `SRID` (unique identifier associated with a specific coordinate system, tolerance, and resolution) on a geometry to a particular integer value:

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
  location: sql\`ST_SetSRID(ST_MakePoint(-90.9, 18.7), 4326)\`,
});
```

To compute the distance between the objects you can use `<->` operator and `ST_Distance()` function, which for `geometry types` returns the minimum planar distance between two geometries. This is how you can query for the nearest location by coordinates in Drizzle with PostGIS:

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

const sqlPoint = sql\`ST_SetSRID(ST_MakePoint(${point.x}, ${point.y}), 4326)\`;

await db
  .select({
    ...getColumns(stores),
    distance: sql\`ST_Distance(${stores.location}, ${sqlPoint})\`,
  })
  .from(stores)
  .orderBy(sql\`${stores.location} <-> ${sqlPoint}\`)
  .limit(1);
```
```sql
select *, ST_Distance(location, ST_SetSRID(ST_MakePoint(-73.935_242, 40.730_61), 4326))
from stores order by location <-> ST_SetSRID(ST_MakePoint(-73.935_242, 40.730_61), 4326)
limit 1;
```

To filter stores located within a specified rectangular area, you can use `ST_MakeEnvelope()` and `ST_Within()` functions. `ST_MakeEnvelope()` creates a rectangular Polygon from the minimum and maximum values for X and Y. `ST_Within()` Returns TRUE if geometry A is within geometry B.

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
    sql\`ST_Within(
      ${stores.location}, ST_MakeEnvelope(${point.x1}, ${point.y1}, ${point.x2}, ${point.y2}, 4326)
    )\`,
  );
```
```sql
select * from stores where ST_Within(location, ST_MakeEnvelope(-88, 40, -73, 43, 4326));
```
