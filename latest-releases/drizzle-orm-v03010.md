---
url: https://orm.drizzle.team/docs/latest-releases/drizzle-orm-v03010
title: "Drizzle Orm V03010"
description: ""
access_date: 2026-08-03T18:54:19.022Z
current_date: 2026-08-03T18:54:19.022Z
---

## DrizzleORM v0.30.10 release

## New Features

### 🎉.if() function added to all WHERE expressions

#### Select all posts with views greater than 100

```ts
async function someFunction(views = 0) {
  await db
    .select()
    .from(posts)
    .where(gt(posts.views, views).if(views > 100));
}
```

## Bug Fixes

- Fixed internal mappings for sessions `.all`, `.values`, `.execute` functions in AWS DataAPI

Read get started guide with AWS DataAPI in the [documentation](../get-started-postgresql.md#aws-data-api).
