---
url: https://orm.drizzle.team/docs/guides/postgresql-full-text-search
title: "Postgresql Full Text Search"
description: ""
access_date: 2026-08-03T18:54:19.022Z
current_date: 2026-08-03T18:54:19.022Z
---

## PostgreSQL full-text search

This guide assumes familiarity with:

- Get started with [PostgreSQL](../get-started-postgresql.md)
- [Select statement](../select.md)
- [Indexes](../indexes-constraints.md#indexes)
- [sql operator](../sql.md)
- You should have `drizzle-orm@0.31.0` and `drizzle-kit@0.22.0` or higher.

This guide demonstrates how to implement full-text search in PostgreSQL with Drizzle ORM. Full-text search is a technique used to search for text within a document or a set of documents. A document is the unit of searching in a full text search system. PostgreSQL provides a set of functions to work with full-text search, such as `to_tsvector` and `to_tsquery`:

The `to_tsvector` function parses a textual document into tokens, reduces the tokens to lexemes, and returns a `tsvector` which lists the lexemes together with their positions in the document:

```ts
import { sql } from 'drizzle-orm';

const db = drizzle(...);

await db.execute(
  sql\`select to_tsvector('english', 'Guide to PostgreSQL full-text search with Drizzle ORM')\`,
);
```
```json
[
  {
    to_tsvector: "'drizzl':9 'full':5 'full-text':4
    'guid':1 'orm':10 'postgresql':3 'search':7 'text':6"
  }
]
```

The `to_tsquery` function converts a keyword to normalized tokens and returns a `tsquery` that matches the lexemes in a `tsvector`. The `@@` operator is used for direct matches:

```ts
await db.execute(
  sql\`select to_tsvector('english', 'Guide to PostgreSQL full-text search with Drizzle ORM')
    @@ to_tsquery('english', 'Drizzle') as match\`,
);
```
```json
[ { match: true } ]
```

As for now, Drizzle doesn’t support `tsvector` type natively, so you need to convert your data in the `text` column on the fly. To enhance the performance, you can create a `GIN` index on your column like this:

schema.ts

migration.sql

db\_data

```ts
import { index, pgTable, serial, text } from 'drizzle-orm/pg-core';

export const posts = pgTable(
  'posts',
  {
    id: serial('id').primaryKey(),
    title: text('title').notNull(),
  },
  (table) => [
    index('title_search_index').using('gin', sql\`to_tsvector('english', ${table.title})\`),
  ]
);
```

To implement full-text search in PostgreSQL with Drizzle ORM, you can use the `to_tsvector` and `to_tsquery` functions with `sql` operator:

```ts
import { sql } from 'drizzle-orm';
import { posts } from './schema';

const title = 'trip';

await db
  .select()
  .from(posts)
  .where(sql\`to_tsvector('english', ${posts.title}) @@ to_tsquery('english', ${title})\`);
```
```json
[
  { id: 1, title: 'Planning Your First Trip to Europe' },
  { id: 3, title: 'Top 5 Destinations for a Family Trip' },
  { id: 5, title: 'Trip Planning: Choosing Your Next Destination' },
  { id: 7, title: 'The Ultimate Road Trip Guide for Explorers' }
]
```

To match by any of the keywords, you can use the `|` operator:

```ts
const title = 'Europe | Asia';

await db
  .select()
  .from(posts)
  .where(sql\`to_tsvector('english', ${posts.title}) @@ to_tsquery('english', ${title})\`);
```
```json
[
  { id: 1, title: 'Planning Your First Trip to Europe' },
  { id: 2, title: "Cultural Insights: Exploring Asia's Heritage" }
]
```

To match multiple keywords, you can use the `plainto_tsquery` function:

```ts
// 'discover & Italy'
const title = 'discover Italy';

await db
  .select()
  .from(posts)
  .where(sql\`to_tsvector('english', ${posts.title}) @@ plainto_tsquery('english', ${title})\`);
```
```sql
select * from posts
  where to_tsvector('english', title) @@ plainto_tsquery('english', 'discover Italy');
```
```json
[ { id: 6, title: 'Discovering Hidden Culinary Gems in Italy' } ]
```

To match a phrase, you can use the `phraseto_tsquery` function:

```ts
// if you query by "trip family", it will not return any result
// 'family <-> trip'
const title = 'family trip';

await db
  .select()
  .from(posts)
  .where(sql\`to_tsvector('english', ${posts.title}) @@ phraseto_tsquery('english', ${title})\`);
```
```sql
select * from posts
  where to_tsvector('english', title) @@ phraseto_tsquery('english', 'family trip');
```
```json
[ { id: 3, title: 'Top 5 Destinations for a Family Trip' } ]
```

You can also use `websearch_to_tsquery` function which is a simplified version of `to_tsquery` with an alternative syntax, similar to the one used by web search engines:

```ts
// 'family | first & trip & europ | asia'
const title = 'family or first trip Europe or Asia';

await db
  .select()
  .from(posts)
  .where(sql\`to_tsvector('english', ${posts.title}) @@ websearch_to_tsquery('english', ${title})\`);
```
```sql
select * from posts
  where to_tsvector('english', title)
  @@ websearch_to_tsquery('english', 'family or first trip Europe or Asia');
```
```json
[
  { id: 1, title: 'Planning Your First Trip to Europe' },
  { id: 2, title: "Cultural Insights: Exploring Asia's Heritage" },
  { id: 3, title: 'Top 5 Destinations for a Family Trip' }
]
```

To implement full-text search on multiple columns, you can create index on multiple columns and concatenate the columns with `to_tsvector` function:

schema.ts

migration.sql

db\_data

```ts
import { sql } from 'drizzle-orm';
import { index, pgTable, serial, text } from 'drizzle-orm/pg-core';

export const posts = pgTable(
  'posts',
  {
    id: serial('id').primaryKey(),
    title: text('title').notNull(),
    description: text('description').notNull(),
  },
  (table) => [
    index('search_index').using(
      'gin',
      sql\`(
          setweight(to_tsvector('english', ${table.title}), 'A') ||
          setweight(to_tsvector('english', ${table.description}), 'B')
      )\`,
    ),
  ],
);
```

The `setweight` function is used to label the entries of a tsvector with a given weight, where a weight is one of the letters A, B, C, or D. This is typically used to mark entries coming from different parts of a document, such as title versus body.

This is how you can query on multiple columns:

```ts
const title = 'plan';

await db.select().from(posts)
  .where(sql\`(
      setweight(to_tsvector('english', ${posts.title}), 'A') ||
      setweight(to_tsvector('english', ${posts.description}), 'B'))
      @@ to_tsquery('english', ${title}
    )\`
  );
```
```json
[
  {
    id: 1,
    title: 'Planning Your First Trip to Europe',
    description: 'Get essential tips on budgeting, sightseeing, and cultural etiquette for your inaugural European adventure.'
  },
  {
    id: 5,
    title: 'Trip Planning: Choosing Your Next Destination',
    description: 'Learn how to select destinations that align with your travel goals, whether for leisure, adventure, or cultural exploration.'
  },
  {
    id: 7,
    title: 'The Ultimate Road Trip Guide for Explorers',
    description: 'Plan your next great road trip with tips on route planning, packing, and discovering off-the-beaten-path attractions.'
  }
]
```

To rank the search results, you can use the `ts_rank` or `ts_rank_cd` functions and `orderBy` method:

IMPORTANT

`getColumns` available starting from `drizzle-orm@1.0.0-beta.2` (read more [here](../upgrade-v1.md))

If you are on pre-1 version(like `0.45.1`) then use `getTableColumns`

```ts
import { desc, getColumns, sql } from 'drizzle-orm';

const search = 'culture | Europe | Italy | adventure';

const matchQuery = sql\`(
  setweight(to_tsvector('english', ${posts.title}), 'A') ||
  setweight(to_tsvector('english', ${posts.description}), 'B')), to_tsquery('english', ${search})\`;

await db
  .select({
    ...getColumns(posts),
    rank: sql\`ts_rank(${matchQuery})\`,
    rankCd: sql\`ts_rank_cd(${matchQuery})\`,
  })
  .from(posts)
  .where(
    sql\`(
      setweight(to_tsvector('english', ${posts.title}), 'A') ||
      setweight(to_tsvector('english', ${posts.description}), 'B')
      ) @@ to_tsquery('english', ${search})\`,
  )
  .orderBy((t) => desc(t.rank));
```
```json
[
  {
    id: 1,
    title: 'Planning Your First Trip to Europe',
    description: 'Get essential tips on budgeting, sightseeing, and cultural etiquette for your inaugural European adventure.',
    rank: 0.2735672,
    rankCd: 1.8
  },
  {
    id: 6,
    title: 'Discovering Hidden Culinary Gems in Italy',
    description: "Unearth Italy's lesser-known eateries and food markets that offer authentic and traditional flavors.",
    rank: 0.16717994,
    rankCd: 1.4
  },
  {
    id: 2,
    title: "Cultural Insights: Exploring Asia's Heritage",
    description: 'Dive deep into the rich history and traditions of Asia through immersive experiences and local interactions.',
    rank: 0.15198177,
    rankCd: 1
  },
  {
    id: 5,
    title: 'Trip Planning: Choosing Your Next Destination',
    description: 'Learn how to select destinations that align with your travel goals, whether for leisure, adventure, or cultural exploration.',
    rank: 0.12158542,
    rankCd: 0.8
  }
]
```

The `ts_rank` function focuses on the frequency of query terms throughout the document. The `ts_rank_cd` function focuses on the proximity of query terms within the document.
