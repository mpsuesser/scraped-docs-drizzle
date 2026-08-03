---
url: https://orm.drizzle.team/docs/guides/conditional-filters-in-query
title: "Conditional Filters In Query"
description: ""
access_date: 2026-08-03T19:43:09.006Z
current_date: 2026-08-03T19:43:09.006Z
---

## Conditional filters in query

PostgreSQL

MySQL

SQLite

MSSQL

Cockroach

This guide assumes familiarity with:

- Get started with [PostgreSQL](../get-started-postgresql.md), [MySQL](../mysql/get-started-mysql.md), [SQLite](../sqlite/get-started-sqlite.md), [MSSQL](../mssql/get-started-mssql.md) and [Cockroach](../cockroach/get-started-cockroach.md);
- [Select statement](../select.md);
- [Filtering](../select.md#filters) and [Filter operators](../operators.md);

To pass a conditional filter in query you can use `.where()` method and logical operator like below:

```ts
import { ilike } from 'drizzle-orm';

const db = drizzle(...)

const searchPosts = async (term?: string) => {
  await db
    .select()
    .from(posts)
    .where(term ? ilike(posts.title, term) : undefined);
};

await searchPosts();
await searchPosts('AI');
```
```sql
select * from posts;
select * from posts where title ilike 'AI';
```

To combine conditional filters you can use `and()` or `or()` operators like below:

```ts
import { and, gt, ilike, inArray } from 'drizzle-orm';

const searchPosts = async (term?: string, categories: string[] = [], views = 0) => {
  await db
    .select()
    .from(posts)
    .where(
      and(
        term ? ilike(posts.title, term) : undefined,
        categories.length > 0 ? inArray(posts.category, categories) : undefined,
        views > 100 ? gt(posts.views, views) : undefined,
      ),
    );
};

await searchPosts();
await searchPosts('AI', ['Tech', 'Art', 'Science'], 200);
```
```sql
select * from posts;
select * from posts
  where (
    title ilike 'AI'
    and category in ('Tech', 'Science', 'Art')
    and views > 200
  );
```

If you need to combine conditional filters in different part of the project you can create a variable, push filters and then use it in `.where()` method with `and()` or `or()` operators like below:

```ts
import { SQL, ... } from 'drizzle-orm';

const searchPosts = async (filters: SQL[]) => {
  await db
    .select()
    .from(posts)
    .where(and(...filters));
};

const filters: SQL[] = [];
filters.push(ilike(posts.title, 'AI'));
filters.push(inArray(posts.category, ['Tech', 'Art', 'Science']));
filters.push(gt(posts.views, 200));

await searchPosts(filters);
```

Drizzle has useful and flexible API, which lets you create your custom solutions. This is how you can create a custom filter operator:

```ts
import { AnyColumn, ... } from 'drizzle-orm';

// length less than
const lenlt = (column: AnyColumn, value: number) => {
  return sql\`length(${column}) < ${value}\`;
};

const searchPosts = async (maxLen = 0, views = 0) => {
  await db
    .select()
    .from(posts)
    .where(
      and(
        maxLen ? lenlt(posts.title, maxLen) : undefined,
        views > 100 ? gt(posts.views, views) : undefined,
      ),
    );
};

await searchPosts(8);
await searchPosts(8, 200);
```
```sql
select * from posts where length(title) < 8;
select * from posts where (length(title) < 8 and views > 200);
```

Drizzle filter operators are just SQL expressions under the hood. This is example of how `lt` operator is implemented in Drizzle:

```js
const lt = (left, right) => {
  return sql\`${left} < ${bindIfParam(right, left)}\`; // bindIfParam is internal magic function
};
```
