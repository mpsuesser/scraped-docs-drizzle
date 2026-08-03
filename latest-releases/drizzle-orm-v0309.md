---
url: https://orm.drizzle.team/docs/latest-releases/drizzle-orm-v0309
title: "Drizzle Orm V0309"
description: ""
access_date: 2026-08-03T19:08:17.353Z
current_date: 2026-08-03T19:08:17.353Z
---

## DrizzleORM v0.30.9 release

## New Features

- Added `setWhere` and `targetWhere` fields to `.onConflictDoUpdate()` config in SQLite instead of single `where` field:
```ts
await db.insert(employees)
  .values({ employeeId: 123, name: 'John Doe' })
  .onConflictDoUpdate({
    target: employees.employeeId,
    targetWhere: sql\`name <> 'John Doe'\`,
    set: { name: sql\`excluded.name\` }
  });
  
await db.insert(employees)
  .values({ employeeId: 123, name: 'John Doe' })
  .onConflictDoUpdate({
    target: employees.employeeId,
    set: { name: 'John Doe' },
    setWhere: sql\`name <> 'John Doe'\`
  });
```

Read more about `.onConflictDoUpdate()` method [here](../insert.md#on-conflict-do-update).

- 🛠️ Added schema information to Drizzle instances via `db._.fullSchema`

## Fixes

- Fixed migrator in AWS Data API

To get started with Drizzle and AWS Data API follow the [documentation](../get-started-postgresql.md#aws-data-api).
