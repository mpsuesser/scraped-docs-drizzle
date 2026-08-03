---
url: https://orm.drizzle.team/docs/cockroach/guides/include-or-exclude-columns
title: "Include Or Exclude Columns"
description: ""
access_date: 2026-08-03T19:38:26.356Z
current_date: 2026-08-03T19:38:26.356Z
---

## Include or Exclude Columns in Query

PostgreSQL

MySQL

SQLite

MSSQL

Cockroach

This guide assumes familiarity with:

- Get started with [PostgreSQL](../../get-started-postgresql.md), [MySQL](../../mysql/get-started-mysql.md), [SQLite](../../sqlite/get-started-sqlite.md), [MSSQL](../../mssql/get-started-mssql.md) and [Cockroach](../get-started-cockroach.md)
- [Select statement](../../select.md)
- [Get typed table columns](../../goodies.md#get-typed-table-columns)
- [Joins](../../joins.md)
- [Relational queries](../../rqb.md)
- [Partial select with relational queries](../../rqb.md#partial-fields-select)

Drizzle has flexible API for including or excluding columns in queries. To include all columns you can use `.select()` method like this:

index.ts

schema.ts

```ts
import { posts } from './schema';

const db = drizzle(...);

await db.select().from(posts);
```
```ts
// result type
type Result = {
  id: number;
  title: string;
  content: string;
  views: number;
}[];
```

To include specific columns you can use `.select()` method like this:

```ts
await db.select({ title: posts.title }).from(posts);
```
```ts
// result type
type Result = {
  title: string;
}[];
```

To include all columns with extra columns you can use `getColumns()` utility function like this:

```ts
import { getColumns, sql } from 'drizzle-orm';

await db
  .select({
    ...getColumns(posts),
    titleLength: sql<number>\`length(${posts.title})\`,
  })
  .from(posts);
```
```ts
// result type
type Result = {
  id: number;
  title: string;
  content: string;
  views: number;
  titleLength: number;
}[];
```

To exclude columns you can use `getColumns()` utility function like this:

```ts
import { getColumns } from 'drizzle-orm';

const { content, ...rest } = getColumns(posts); // exclude "content" column

await db.select({ ...rest }).from(posts); // select all other columns
```
```ts
// result type
type Result = {
  id: number;
  title: string;
  views: number;
}[];
```

This is how you can include or exclude columns with joins:

index.ts

schema.ts

```ts
import { eq, getColumns } from 'drizzle-orm';
import { comments, posts, users } from './db/schema';

// exclude "userId" and "postId" columns from "comments"
const { userId, postId, ...rest } = getColumns(comments);

await db
  .select({
    postId: posts.id, // include "id" column from "posts"
    comment: { ...rest }, // include all other columns
    user: users, // equivalent to getColumns(users)
  })
  .from(posts)
  .leftJoin(comments, eq(posts.id, comments.postId))
  .leftJoin(users, eq(users.id, posts.userId));
```
```ts
// result type
type Result = {
  postId: number;
  comment: {
    id: number;
    content: string;
    createdAt: Date;
  } | null;
  user: {
    id: number;
    name: string;
    email: string;
  } | null;
}[];
```

Drizzle has useful relational queries API, that lets you easily include or exclude columns in queries. This is how you can include all columns:

index.ts

schema.ts

relations.ts

```ts
import { relations } from './relations';

const db = drizzle(..., { relations });

await db.query.posts.findMany();
```
```ts
// result type
type Result = {
  id: number;
  title: string;
  content: string;
  views: number;
}[]
```

This is how you can include specific columns using relational queries:

```ts
await db.query.posts.findMany({
  columns: {
    title: true,
  },
});
```
```ts
// result type
type Result = {
  title: string;
}[]
```

This is how you can include all columns with extra columns using relational queries:

```ts
import { sql } from 'drizzle-orm';

await db.query.posts.findMany({
  extras: {
    titleLength: (t) => sql<number>\`length(${t.title})\`.as("title_length"),
  },
});
```
```ts
// result type
type Result = {
  id: number;
  title: string;
  content: string;
  views: number;
  titleLength: number;
}[];
```

This is how you can exclude columns using relational queries:

```ts
await db.query.posts.findMany({
  columns: {
    content: false,
  },
});
```
```ts
// result type
type Result = {
  id: number;
  title: string;
  views: number;
}[]
```

This is how you can include or exclude columns with relations using relational queries:

index.ts

schema.ts

relations.ts

```ts
import { relations } from './relations';

const db = drizzle(..., { relations });

await db.query.posts.findMany({
  columns: {
    id: true, // include "id" column
  },
  with: {
    comments: {
      columns: {
        userId: false, // exclude "userId" column
        postId: false, // exclude "postId" column
      },
    },
    user: true, // include all columns from "users" table
  },
});
```
```ts
// result type
type Result = {
  id: number;
  user: {
    id: number;
    name: string;
    email: string;
  };
  comments: {
    id: number;
    content: string;
    createdAt: Date;
  }[];
}[]
```

This is how you can create custom solution for conditional select:

index.ts

schema.ts

```ts
import { posts } from './schema';

const searchPosts = async (withTitle = false) => {
  return await db
    .select({
      id: posts.id,
      ...(withTitle && { title: posts.title }),
    })
    .from(posts);
};

await searchPosts();
await searchPosts(true);
```
```ts
// result type
type Result = {
  id: number;
  title?: string | undefined;
}[];
```
