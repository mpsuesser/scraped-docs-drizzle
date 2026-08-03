---
url: https://orm.drizzle.team/docs/mssql/guides/full-text-search-with-generated-columns
title: "Full Text Search With Generated Columns"
description: ""
access_date: 2026-08-03T19:43:09.006Z
current_date: 2026-08-03T19:43:09.006Z
---

## Full-text search with Generated Columns

This guide assumes familiarity with:

- Get started with [PostgreSQL](../../get-started-postgresql.md)
- [Select statement](../../select.md)
- [Indexes](../../indexes-constraints.md#indexes)
- [sql operator](../../sql.md)
- [Full-text search](https://orm.drizzle.team/learn/guides/postgresql-full-text-search)
- [Generated columns](../../generated-columns.md)

This guide demonstrates how to implement full-text search in PostgreSQL with Drizzle and generated columns. A generated column is a special column that is always computed from other columns. It is useful because you don’t have to compute the value of the column every time you query the table:

schema.ts

migration.sql

```ts
import { SQL, sql } from 'drizzle-orm';
import { index, pgTable, serial, text, customType } from 'drizzle-orm/pg-core';

export const tsvector = customType<{
data: string;
}>({
dataType() {
  return \`tsvector\`;
},
});

export const posts = pgTable(
'posts',
{
  id: serial('id').primaryKey(),
  title: text('title').notNull(),
  body: text('body').notNull(),
  bodySearch: tsvector('body_search')
    .notNull()
    .generatedAlwaysAs((): SQL => sql\`to_tsvector('english', ${posts.body})\`),
},
(t) => [
  index('idx_body_search').using('gin', t.bodySearch),
]
);
```

When you insert a row into a table, the value of a generated column is computed from an expression that you provide when you create the column:

```ts
import { posts } from './schema';

const db = drizzle(...);

const body = "Golden leaves cover the quiet streets as a crisp breeze fills the air, bringing the scent of rain and the promise of change"

await db.insert(posts).values({
    body,
    title: "The Beauty of Autumn",
  }
).returning();
```
```json
[
  {
    id: 1,
    title: 'The Beauty of Autumn',
    body: 'Golden leaves cover the quiet streets as a crisp breeze fills the air, bringing the scent of rain and the promise of change',
    bodySearch: "'air':13 'breez':10 'bring':14 'chang':23 'cover':3 'crisp':9 'fill':11 'golden':1 'leav':2 'promis':21 'quiet':5 'rain':18 'scent':16 'street':6"
  }
]
```

This is how you can implement full-text search with generated columns in PostgreSQL with Drizzle ORM. The `@@` operator is used for direct matches:

```ts
const searchParam = "bring";

await db
  .select()
  .from(posts)
  .where(sql\`${posts.bodySearch} @@ to_tsquery('english', ${searchParam})\`);
```
```sql
select * from posts where body_search @@ to_tsquery('english', 'bring');
```

This is more advanced schema with a generated column. The `search` column is generated from the `title` and `body` columns and `setweight()` function is used to assign different weights to the columns for full-text search. This is typically used to mark entries coming from different parts of a document, such as title versus body.

schema.ts

migration.sql

```ts
import { SQL, sql } from 'drizzle-orm';
import { index, pgTable, serial, text, customType } from 'drizzle-orm/pg-core';

export const tsvector = customType<{
data: string;
}>({
dataType() {
  return \`tsvector\`;
},
});

export const posts = pgTable(
'posts',
{
 id: serial('id').primaryKey(),
 title: text('title').notNull(),
 body: text('body').notNull(),
 search: tsvector('search')
   .notNull()
   .generatedAlwaysAs(
      (): SQL =>
       sql\`setweight(to_tsvector('english', ${posts.title}), 'A')
        ||
        setweight(to_tsvector('english', ${posts.body}), 'B')\`,
   ),
},
(t) => [
  index('idx_search').using('gin', t.search),
],
);
```

This is how you can query the table with full-text search:

```ts
const search = 'travel';

await db
  .select()
  .from(posts)
  .where(sql\`${posts.search} @@ to_tsquery('english', ${search})\`);
```
```sql
select * from posts where search @@ to_tsquery('english', 'travel');
```
