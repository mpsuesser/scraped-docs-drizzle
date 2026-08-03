---
url: https://orm.drizzle.team/docs/sqlite/query-utils
title: "Query Utils"
description: ""
access_date: 2026-08-03T19:08:17.353Z
current_date: 2026-08-03T19:08:17.353Z
---

## Drizzle query utils

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
const users = await db.query.users.findMany({
  extras: {
    postsCount: (t) => db.$count(posts, eq(posts.authorId, t.id)),
  },
});
```
