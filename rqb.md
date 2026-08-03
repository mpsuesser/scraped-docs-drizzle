---
url: https://orm.drizzle.team/docs/rqb
title: "Rqb"
description: ""
access_date: 2026-08-03T18:54:19.022Z
current_date: 2026-08-03T18:54:19.022Z
---

## Drizzle Queries

Drizzle ORM is designed to be a thin typed layer on top of SQL. We truly believe we’ve designed the best way to operate an SQL database from TypeScript and it’s time to make it better.

Relational queries are meant to provide you with a great developer experience for querying nested relational data from an SQL database, avoiding multiple joins and complex data mappings.

It is an extension to the existing schema definition and query builder. You can opt-in to use it based on your needs. We’ve made sure you have both the best-in-class developer experience and performance.

Important

Inside relational queries, references to a table’s columns must go through the callback parameter, not through the imported table object. This applies to every clause that accepts a callback — `orderBy`, `where`.`RAW`, `extras` and `subqueries inside extras`. The callback exposes the aliased table for the current query scope, which is required for correct SQL generation in nested or self-referential queries.

❌

Example

```ts
await db.query.posts.findMany({
  orderBy: sql\`${posts.id} asc\`, // <- ❌ direct table usage
  with: {
    comments: {
      orderBy: sql\`${comments.id} desc\`, // <- ❌ direct table usage
    },
  },
});
```
```ts
await db.query.users.findMany({
  where: {
    RAW: sql\`LOWER(${users.name}) LIKE 'john%'\`, // <- ❌ direct table usage
  },
});
```
```ts
await db.query.users.findMany({
  extras: {
    loweredName: sql<string>\`lower(${users.name})\`, // <- ❌ direct table usage
    totalPostsCount: db.$count(posts, eq(posts.authorId, users.id)),  // <- ❌ direct table usage
  },
});
```

✅

Example

```ts
await db.query.posts.findMany({
  orderBy: (t) => sql\`${t.id} asc\`, // <- ✅ callback used
  with: {
    comments: {
      orderBy: (t, { desc }) => desc(t.id), // <- ✅ callback used
    },
  },
});
```
```ts
await db.query.users.findMany({
  where: {
    RAW: (t) => sql\`LOWER(${t.name}) LIKE 'john%'\`, // <- ✅ callback used
  },
});
```
```ts
await db.query.users.findMany({
  extras: {
    loweredName: (t, { sql }) => sql<string>\`lower(${t.name})\`, // <- ✅ callback used
    totalPostsCount: (t) => db.$count(posts, eq(posts.authorId, t.id)), // <- ✅ callback used
  },
});
```

`drizzle` import path depends on the **[database driver](connect-overview.md)** you’re using.

index.ts

schema.ts

relations.ts

```typescript
import { relations } from "./relations";
import { drizzle } from "drizzle-orm/...";

const db = drizzle(process.env.DATABASE_URL, { relations });

const result = await db.query.users.findMany({
  with: {
    posts: true,
  },
});
```
```ts
[{
    id: 10,
    name: "Dan",
    posts: [
        {
            id: 1,
            content: "SQL is awesome",
            authorId: 10,
        },
        {
            id: 2,
            content: "But check relational queries",
            authorId: 10,
        }
    ]
}]
```

Relational queries are an extension to Drizzle’s original **[query builder](select.md)**. You need to provide all `tables` and `relations` from your schema file/files upon `drizzle()` initialization and then just use the `db.query` API.

`drizzle` import path depends on the **[database driver](connect-overview.md)** you’re using.

index.ts

schema.ts

relations.ts

```ts
import { relations } from './relations';
import { drizzle } from 'drizzle-orm/...';

const db = drizzle(process.env.DATABASE_URL,{ relations });

await db.query.users.findMany(...);
```

Drizzle provides `.findMany()` and `.findFirst()` APIs.

### Find many

```typescript
const users = await db.query.users.findMany();
```
```ts
// result type
const result: {
    id: number;
    name: string;
    verified: boolean;
    invitedBy: number | null;
}[];
```

### Find first

`.findFirst()` will add `limit 1` to the query.

```typescript
const user = await db.query.users.findFirst();
```
```ts
// result type
const result: {
    id: number;
    name: string;
    verified: boolean;
    invitedBy: number | null;
};
```

### Include relations

`With` operator lets you combine data from multiple related tables and properly aggregate results.

**Getting all posts with comments:**

```typescript
const posts = await db.query.posts.findMany({
    with: {
        comments: true,
    },
});
```

**Getting first post with comments:**

```typescript
const post = await db.query.posts.findFirst({
    with: {
        comments: true,
    },
});
```

You can chain nested with statements as much as necessary.  
For any nested `with` queries Drizzle will infer types using [Core Type API](goodies.md#type-api).

**Get all users with posts. Each post should contain a list of comments:**

```typescript
const users = await db.query.users.findMany({
    with: {
        posts: {
            with: {
                comments: true,
            },
        },
    },
});
```

### Partial fields select

`columns` parameter lets you include or omit columns you want to get from the database.

Drizzle performs partial selects on the query level, no additional data is transferred from the database.

Keep in mind that **a single SQL statement is outputted by Drizzle.**

**Get all posts with just `id`, `content` and include `comments`:**

```typescript
const posts = await db.query.posts.findMany({
    columns: {
        id: true,
        content: true,
    },
    with: {
        comments: true,
    }
});
```

**Get all posts without `content`:**

```typescript
const posts = await db.query.posts.findMany({
    columns: {
        content: false,
    },
});
```

When both `true` and `false` select options are present, all `false` options are ignored.

If you include the `name` field and exclude the `id` field, `id` exclusion will be redundant, all fields apart from `name` would be excluded anyways.

**Exclude and Include fields in the same query:**

```typescript
const users = await db.query.users.findMany({
    columns: {
        name: true,
        id: false //ignored
    },
});
```
```ts
// result type
const users: {
    name: string;
};
```

**Only include columns from nested relations:**

```typescript
const res = await db.query.users.findMany({
    columns: {},
    with: {
        posts: true
    }
});
```
```ts
// result type
const res: {
    posts: {
        id: number,
        text: string
    }
}[];
```

### Nested partial fields select

Just like with **[`partial select`](#partial-select)**, you can include or exclude columns of nested relations:

```typescript
const posts = await db.query.posts.findMany({
    columns: {
        id: true,
        content: true,
    },
    with: {
        comments: {
            columns: {
                authorId: false
            }
        }
    }
});
```

### Select filters

Just like in our SQL-like query builder, relational queries API lets you define filters and conditions with the list of our **[`operators`](operators.md)**.

You can either import them from `drizzle-orm` or use from the callback syntax:

```typescript
const users = await db.query.users.findMany({
    where: {
        id: 1
    }
});
```
```sql
select
  "d0"."id" as "id",
  "d0"."name" as "name"
from
  "users" as "d0"
where
  "d0"."id" = 1
```

Find post with `id=1` and comments that were created before particular date:

```typescript
await db.query.posts.findMany({
  where: {
    id: 1,
  },
  with: {
    comments: {
      where: {
        createdAt: { lt: new Date() },
      },
    },
  },
});
```

**List of all filter operators**

```ts
where: {
    OR: [],
    AND: [],
    NOT: {},
    RAW: (table) => sql\`${table.id} = 1\`,

    // filter by relations
    [relation]: {},

      // filter by columns
    [column]: {
      OR: [],
      AND: [],
      NOT: {},
      eq: 1,
      ne: 1,
      gt: 1,
      gte: 1,
      lt: 1,
      lte: 1,
      in: [1],
      notIn: [1],
      like: "",
      ilike: "",
      notLike: "",
      notIlike: "",
      isNull: true,
      isNotNull: true,
      arrayOverlaps: [1, 2],
      arrayContained: [1, 2],
      arrayContains: [1, 2]
    },
},
```

**Examples**

simple eq

using AND

using OR

using NOT

complex example using RAW

```ts
const response = await db.query.users.findMany({
  where: {
    age: 15,
  },
});
```
```sql
select
  "d0"."id" as "id",
  "d0"."name" as "name",
  "d0"."age" as "age"
from
  "users" as "d0"
where
  "d0"."age" = 15
```

### Relations Filters

With Drizzle Relations, you can filter not only by the table you’re querying but also by any table you include in the query.

**Example:** Get all `users` whose ID>10 and who have at least one post with content starting with “M”

```ts
const usersWithPosts = await db.query.usersTable.findMany({
  where: {
    id: {
      gt: 10
    },
    posts: {
      content: {
        like: 'M%'
      }
    }
  },
});
```

**Example:** Get all `users` with posts, only if user has at least 1 post

```ts
const response = await db.query.users.findMany({
  with: {
    posts: true,
  },
  where: {
    posts: true,
  },
});
```

### Limit & Offset

Drizzle ORM provides `limit` & `offset` API for queries and for the nested entities.

**Find 5 posts:**

```typescript
await db.query.posts.findMany({
    limit: 5,
});
```

**Find posts and get 3 comments at most:**

```typescript
await db.query.posts.findMany({
    with: {
        comments: {
            limit: 3,
        },
    },
});
```

IMPORTANT

`offset` now can be used in with tables as well!

```typescript
await db.query.posts.findMany({
    limit: 5,
    offset: 2, // correct ✅
    with: {
        comments: {
            offset: 3, // correct ✅
            limit: 3,
        },
    },
});
```

Find posts with comments from the 6th to the 10th post:

```typescript
await db.query.posts.findMany({
    with: {
        comments: true,
    },
  limit: 5,
  offset: 5,
});
```

### Order By

Drizzle provides API for ordering in the relational query builder.

You can use same ordering **[core API](select.md#order-by)** or use `order by` operator from the callback with no imports.

important

When you use multiple `orderBy` statements in the same table, they will be included in the query in the same order in which you added them

```typescript
await db.query.posts.findMany({
  orderBy: {
    id: "asc",
  },
});
```

**Order by `asc` + `desc`:**

```typescript
await db.query.posts.findMany({
  orderBy: { id: "asc" },
  with: {
    comments: {
      orderBy: { id: "desc" },
    },
  },
});
```

You can also use custom `sql` in order by statement:

```typescript
await db.query.posts.findMany({
  orderBy: (t) => sql\`${t.id} asc\`,
  with: {
    comments: {
      orderBy: (t, { desc }) => desc(t.id),
    },
  },
});
```

### Include custom fields

Relational query API lets you add custom additional fields. It’s useful when you need to retrieve data and apply additional functions to it.

IMPORTANT

As of now aggregations are not supported in `extras`, please use **[`core queries`](select.md)** for that.

```typescript
import { sql } from 'drizzle-orm';

await db.query.users.findMany({
  extras: {
    loweredName: (table) => sql\`lower(${table.name})\`,
  },
});
```
```typescript
await db.query.users.findMany({
    extras: {
        loweredName: (users, { sql }) => sql\`lower(${users.name})\`,
    },
})
```

`lowerName` as a key will be included to all fields in returned object.

IMPORTANT

If you will specify `.as("<alias>")` for any extras field - drizzle will ignore it

To retrieve all users with groups, but with the fullName field included (which is a concatenation of firstName and lastName), you can use the following query with the Drizzle relational query builder.

```typescript
const res = await db.query.users.findMany({
    extras: {
        fullName: (users, { sql }) => sql<string>\`concat(${users.name}, " ", ${users.name})\`,
    },
    with: {
        groups: true,
    },
});
```
```ts
// result type
const res: {
    id: number;
    name: string;
    age: number | null;
    invitedBy: number | null;
    fullName: string;
    groups: {
        id: number;
        name: string;
        description: string | null;
    }[];
}[]
```

To retrieve all posts with comments and add an additional field to calculate the size of the post content and the size of each comment content:

```typescript
const res = await db.query.posts.findMany({
    extras: {
        contentLength: (table, { sql }) => sql<number>\`length(${table.content})\`,
    },
    with: {
        comments: {
            extras: {
                commentSize: (table, { sql }) => sql<number>\`length(${table.content})\`,
            },
        },
    },
});
```
```ts
// result type
const res: {
    id: number;
    content: string;
    authorId: number | null;
    createdAt: Date;
    contentLength: number;
    comments: {
        id: number;
        content: string;
        createdAt: Date;
        creator: number | null;
        postId: number | null;
        commentSize: number;
    }[];
}[]
```

### Include subqueries

You can also use subqueries within Relational Queries to leverage the power of custom SQL syntax

**Get users with posts and total posts count for each user**

```ts
import { posts } from './schema';
import { eq } from 'drizzle-orm';

await db.query.users.findMany({
  with: {
    posts: true
  },
  extras: {
    totalPostsCount: (table) => db.$count(posts, eq(posts.authorId, table.id)),
  }
});
```
```sql
SELECT
  "d0"."id" AS "id",
  "d0"."name" AS "name",
  "d0"."age" AS "age",
  "d0"."invited_by" AS "invitedBy",
  (
    (
      SELECT
        count(*)
      FROM
        "posts"
      WHERE
        "posts"."author_id" = "d0"."id"
    )
  ) AS "totalPostsCount",
  "posts"."r" AS "posts"
FROM
  "users" AS "d0"
  LEFT JOIN LATERAL (
    SELECT
      coalesce(json_agg(row_to_json("t".*)), '[]') AS "r"
    FROM
      (
        SELECT
          "d1"."id" AS "id",
          "d1"."content" AS "content",
          "d1"."author_id" AS "authorId",
          "d1"."created_at"::text AS "createdAt"
        FROM
          "posts" AS "d1"
        WHERE
          "d0"."id" = "d1"."author_id"
      ) AS "t"
  ) AS "posts" ON TRUE
```

### Prepared statements

Prepared statements are designed to massively improve query performance — [see here.](perf-queries.md)

In this section, you can learn how to define placeholders and execute prepared statements using the Drizzle relational query builder.

##### Placeholder in where

```ts
const prepared = db.query.users.findMany({
    where: { id: { eq: sql.placeholder("id") } },
    with: {
      posts: {
        where: { id: 1 },
      },
    },
}).prepare("query_name");

const usersWithPosts = await prepared.execute({ id: 1 });
```

##### Placeholder in limit

```ts
const prepared = db.query.users.findMany({
    with: {
      posts: {
        limit: sql.placeholder("limit"),
      },
    },
  }).prepare("query_name");

const usersWithPosts = await prepared.execute({ limit: 1 });
```

##### Placeholder in offset

```ts
const prepared = db.query.users.findMany({
    offset: sql.placeholder('offset'),
    with: {
        posts: true,
    },
}).prepare('query_name');

const usersWithPosts = await prepared.execute({ offset: 1 });
```

##### Multiple placeholders

```ts
const prepared = db.query.users.findMany({
    limit: sql.placeholder("uLimit"),
    offset: sql.placeholder("uOffset"),
    where: {
      OR: [{ id: { eq: sql.placeholder("id") } }, { id: 3 }],
    },
    with: {
      posts: {
        where: { id: { eq: sql.placeholder("pid") } },
        limit: sql.placeholder("pLimit"),
      },
    },
}).prepare("query_name");

const usersWithPosts = await prepared.execute({ pLimit: 1, uLimit: 3, uOffset: 1, id: 2, pid: 6 });
```
