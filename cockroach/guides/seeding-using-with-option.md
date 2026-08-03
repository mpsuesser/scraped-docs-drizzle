---
url: https://orm.drizzle.team/docs/cockroach/guides/seeding-using-with-option
title: "Seeding Using With Option"
description: ""
access_date: 2026-08-03T19:43:09.006Z
current_date: 2026-08-03T19:43:09.006Z
---

## Seeding using 'with' option

PostgreSQL

MySQL

SQLite

MSSQL

Cockroach

This guide assumes familiarity with:

- Get started with [PostgreSQL](../../get-started-postgresql.md), [MySQL](../../mysql/get-started-mysql.md), [SQLite](../../sqlite/get-started-sqlite.md), [MSSQL](../../mssql/get-started-mssql.md) and [Cockroach](../get-started-cockroach.md)
- Get familiar with [One-to-many Relation](../../relations.md#one-to-many)
- Get familiar with [Drizzle Seed](../../seed-overview.md)

Warning

Using `with` implies tables to have a one-to-many relationship.

Therefore, if `one` user has `many` posts, you can use `with` as follows:

```ts
users: {
    count: 2,
    with: {
        posts: 3,
    },
},
```

## Example 1

index.ts

schema.ts

```ts
import { users, posts } from './schema.ts';

async function main() {
    const db = drizzle(...);
    await seed(db, { users, posts }).refine(() => ({
        users: {
            count: 2,
            with: {
                posts: 3,
            },
        },
    }));
}
main();
```

Running the seeding script above will cause an error.

```plaintext
Error: "posts" table doesn't have a reference to "users" table or
you didn't include your one-to-many relation in the seed function schema.
You can't specify "posts" as parameter in users.with object.
```

You will have several options to resolve an error:

- You can add reference to the `authorId` column in `posts` table in your schema

index.ts

schema.ts

```ts
import { users, posts } from './schema.ts';

async function main() {
    const db = drizzle(...);
    await seed(db, { users, posts }).refine(() => ({
        users: {
            count: 2,
            with: {
                posts: 3,
            },
        },
    }));
}
main();

// Running the seeding script above will fill you database with values shown below
```
```mdx
\`users\`

| id |   name   |   
| -- | -------- |
|  1 | 'Melanny' | 
|  2 | 'Elvera' |

\`posts\`

| id |        content        | author_id |   
| -- | --------------------- | --------- |
|  1 | 'tf02gUXb0LZIdEg6SL'  |     2     |
|  2 | 'j15YdT7Sma'          |     2     |
|  3 | 'LwwvWtLLAZzIpk'      |     1     |
|  4 | 'mgyUnBKSrQw'         |     1     |
|  5 | 'CjAJByKIqilHcPjkvEw' |     2     |
|  6 | 'S5g0NzXs'            |     1     |
```

- You can add one-to-many relation to your schema and include it in the seed function schema

index.ts

schema.ts

```ts
import { users, posts, postsRelations } from './schema.ts';

async function main() {
    const db = drizzle(...);
    await seed(db, { users, posts, postsRelations }).refine(() => ({
        users: {
            count: 2,
            with: {
                posts: 3,
            },
        },
    }));
}
main();

// Running the seeding script above will fill you database with values shown below
```
```mdx
\`users\`

| id |   name   |   
| -- | -------- |
|  1 | 'Melanny' | 
|  2 | 'Elvera' |

\`posts\`

| id |        content        | author_id |   
| -- | --------------------- | --------- |
|  1 | 'tf02gUXb0LZIdEg6SL'  |     2     |
|  2 | 'j15YdT7Sma'          |     2     |
|  3 | 'LwwvWtLLAZzIpk'      |     1     |
|  4 | 'mgyUnBKSrQw'         |     1     |
|  5 | 'CjAJByKIqilHcPjkvEw' |     2     |
|  6 | 'S5g0NzXs'            |     1     |
```

## Example 2

index.ts

schema.ts

```ts
import { users, posts } from './schema.ts';

async function main() {
    const db = drizzle(...);
    await seed(db, { users, posts }).refine(() => ({
        posts: {
            count: 2,
            with: {
                users: 3,
            },
        },
    }));
}
main();
```

Running the seeding script above will cause an error.

```plaintext
Error: "posts" table doesn't have a reference to "users" table or
you didn't include your one-to-many relation in the seed function schema.
You can't specify "posts" as parameter in users.with object.
```

Why?

You have a `posts` table referencing a `users` table in your schema,

```ts
.
.
.
export const posts = pgTable('posts', {
    id: serial('id').primaryKey(),
    content: text('content'),
    authorId: integer('author_id').notNull().references(() => users.id),
});
```

or in other words, you have one-to-many relation where `one` user can have `many` posts.

However, in your seeding script, you’re attempting to generate 3 (`many`) users for `one` post.

```ts
posts: {
    count: 2,
    with: {
        users: 3,
    },
},
```

To resolve the error, you can modify your seeding script as follows:

```ts
import { users, posts, postsRelations } from './schema.ts';

async function main() {
    const db = drizzle(...);
    await seed(db, { users, posts, postsRelations }).refine(() => ({
        users: {
            count: 2,
            with: {
                posts: 3,
            },
        },
    }));
}
main();

// Running the seeding script above will fill you database with values shown below
```
```mdx
\`users\`

| id |   name   |   
| -- | -------- |
|  1 | 'Melanny' | 
|  2 | 'Elvera' |

\`posts\`

| id |        content        | author_id |   
| -- | --------------------- | --------- |
|  1 | 'tf02gUXb0LZIdEg6SL'  |     2     |
|  2 | 'j15YdT7Sma'          |     2     |
|  3 | 'LwwvWtLLAZzIpk'      |     1     |
|  4 | 'mgyUnBKSrQw'         |     1     |
|  5 | 'CjAJByKIqilHcPjkvEw' |     2     |
|  6 | 'S5g0NzXs'            |     1     |
```

## Example 3

index.ts

schema.ts

```ts
import { users } from './schema.ts';

async function main() {
    const db = drizzle(...);
    await seed(db, { users }).refine(() => ({
        users: {
            count: 2,
            with: {
                users: 3,
            },
        },
    }));
}
main();
```

Running the seeding script above will cause an error.

```plaintext
Error: "users" table has self reference.
You can't specify "users" as parameter in users.with object.
```

Why?

You have a `users` table referencing a `users` table in your schema,

```ts
.
.
.
export const users = pgTable('users', {
    id: serial('id').primaryKey(),
    name: text('name'),
    reportsTo: integer('reports_to').references((): AnyPgColumn => users.id),
});
```

or in other words, you have one-to-one relation where `one` user can have only `one` user.

However, in your seeding script, you’re attempting to generate 3 (`many`) users for `one` user, which is impossible.

```ts
users: {
    count: 2,
    with: {
        users: 3,
    },
},
```
