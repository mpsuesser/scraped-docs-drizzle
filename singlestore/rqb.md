---
url: https://orm.drizzle.team/docs/singlestore/rqb
title: "Rqb"
description: ""
access_date: 2026-08-03T19:43:09.006Z
current_date: 2026-08-03T19:43:09.006Z
---

## Drizzle Queries

Drizzle ORM is designed to be a thin typed layer on top of SQL. We truly believe we’ve designed the best way to operate an SQL database from TypeScript and it’s time to make it better.

Relational queries are meant to provide you with a great developer experience for querying nested relational data from an SQL database, avoiding multiple joins and complex data mappings.

It is an extension to the existing schema definition and query builder. You can opt-in to use it based on your needs. We’ve made sure you have both the best-in-class developer experience and performance.

index.ts

schema.ts

```typescript
import * as schema from './schema';
import { drizzle } from 'drizzle-orm/...';

const db = drizzle({ schema });

const result = await db._query.users.findMany({
    with: {
        posts: true            
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

⚠️ If you have SQL schema declared in multiple files you can do it like that

```typescript
import * as schema1 from './schema1';
import * as schema2 from './schema2';
import { drizzle } from 'drizzle-orm/...';

const db = drizzle({ schema: { ...schema1, ...schema2 } });

const result = await db._query.users.findMany({
    with: {
        posts: true            
    },
});
```

## Querying

Relational queries are an extension to Drizzle’s original **[query builder](select.md)**. You need to provide all `tables` and `relations` from your schema file/files upon `drizzle()` initialization and then just use the `db._query` API.

`drizzle` import path depends on the **[database driver](connect-overview.md)** you’re using.

index.ts

schema.ts

```ts
import * as schema from './schema';
import { drizzle } from 'drizzle-orm/...';

const db = drizzle({ schema });

await db._query.users.findMany(...);
```
```ts
// if you have schema in multiple files
import * as schema1 from './schema1';
import * as schema2 from './schema2';
import { drizzle } from 'drizzle-orm/...';

const db = drizzle({ schema: { ...schema1, ...schema2 } });

await db._query.users.findMany(...);
```

Drizzle provides `.findMany()` and `.findFirst()` APIs.

### Find many

```typescript
const users = await db._query.users.findMany();
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
const user = await db._query.users.findFirst();
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
const posts = await db._query.posts.findMany({
    with: {
        comments: true,
    },
});
```

**Getting first post with comments:**

```typescript
const post = await db._query.posts.findFirst({
    with: {
        comments: true,
    },
});
```

You can chain nested with statements as much as necessary.  
For any nested `with` queries Drizzle will infer types using [Core Type API](goodies.md#type-api).

**Get all users with posts. Each post should contain a list of comments:**

```typescript
const users = await db._query.users.findMany({
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
const posts = await db._query.posts.findMany({
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
const posts = await db._query.posts.findMany({
    columns: {
        content: false,
    },
});
```

When both `true` and `false` select options are present, all `false` options are ignored.

If you include the `name` field and exclude the `id` field, `id` exclusion will be redundant, all fields apart from `name` would be excluded anyways.

**Exclude and Include fields in the same query:**

```typescript
const users = await db._query.users.findMany({
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
const res = await db._query.users.findMany({
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
const posts = await db._query.posts.findMany({
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
import { eq } from 'drizzle-orm';

const users = await db._query.users.findMany({
    where: eq(users.id, 1)
})
```
```ts
const users = await db._query.users.findMany({
    where: (users, { eq }) => eq(users.id, 1),
})
```

Find post with `id=1` and comments that were created before particular date:

```typescript
await db._query.posts.findMany({
    where: (posts, { eq }) => (eq(posts.id, 1)),
    with: {
        comments: {
            where: (comments, { lt }) => lt(comments.createdAt, new Date()),
        },
    },
});
```

### Limit & Offset

Drizzle ORM provides `limit` & `offset` API for queries and for the nested entities.

**Find 5 posts:**

```typescript
await db._query.posts.findMany({
    limit: 5,
});
```

**Find posts and get 3 comments at most:**

```typescript
await db._query.posts.findMany({
    with: {
        comments: {
            limit: 3,
        },
    },
});
```

IMPORTANT

`offset` is only available for top level query.

```typescript
await db._query.posts.findMany({
    limit: 5,
    offset: 2, // correct ✅
    with: {
        comments: {
            offset: 3, // incorrect ❌
            limit: 3,
        },
    },
});
```

Find posts with comments from the 5th to the 10th post:

```typescript
await db._query.posts.findMany({
    limit: 5,
  offset: 5,
    with: {
        comments: true,
    },
});
```

### Order By

Drizzle provides API for ordering in the relational query builder.

You can use same ordering **[core API](select.md#order-by)** or use `order by` operator from the callback with no imports.

```typescript
import { desc, asc } from 'drizzle-orm';

await db._query.posts.findMany({
    orderBy: [asc(posts.id)],
});
```
```typescript
await db._query.posts.findMany({
    orderBy: (posts, { asc }) => [asc(posts.id)],
});
```

**Order by `asc` + `desc`:**

```typescript
await db._query.posts.findMany({
    orderBy: (posts, { asc }) => [asc(posts.id)],
    with: {
        comments: {
            orderBy: (comments, { desc }) => [desc(comments.id)],
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

await db._query.users.findMany({
    extras: {
        loweredName: sql\`lower(${users.name})\`.as('lowered_name'),
    },
})
```
```typescript
await db._query.users.findMany({
    extras: {
        loweredName: (users, { sql }) => sql\`lower(${users.name})\`.as('lowered_name'),
    },
})
```

`lowerName` as a key will be included to all fields in returned object.

IMPORTANT

You have to explicitly specify `.as("<name_for_column>")`

To retrieve all users with groups, but with the fullName field included (which is a concatenation of firstName and lastName), you can use the following query with the Drizzle relational query builder.

```typescript
const res = await db._query.users.findMany({
    extras: {
        fullName: sql<string>\`concat(${users.name}, " ", ${users.name})\`.as('full_name'),
    },
    with: {
        usersToGroups: {
            with: {
                group: true,
            },
        },
    },
});
```
```ts
// result type
const res: {
    id: number;
    name: string;
    verified: boolean;
    invitedBy: number | null;
    fullName: string;
    usersToGroups: {
            group: {
                    id: number;
                    name: string;
                    description: string | null;
            };
    }[];
}[];
```

To retrieve all posts with comments and add an additional field to calculate the size of the post content and the size of each comment content:

```typescript
const res = await db._query.posts.findMany({
    extras: (table, { sql }) => ({
        contentLength: (sql<number>\`length(${table.content})\`).as('content_length'),
    }),
    with: {
        comments: {
            extras: {
                commentSize: sql<number>\`length(${comments.content})\`.as('comment_size'),
            },
        },
    },
});
```
```ts
// result type
const res: {
    id: number;
    createdAt: Date;
    content: string;
    authorId: number | null;
    contentLength: number;
    comments: {
            id: number;
            createdAt: Date;
            content: string;
            creator: number | null;
            postId: number | null;
            commentSize: number;
    }[];
};
```

### Prepared statements

Prepared statements are designed to massively improve query performance — [see here.](perf-queries.md)

In this section, you can learn how to define placeholders and execute prepared statements using the Drizzle relational query builder.
