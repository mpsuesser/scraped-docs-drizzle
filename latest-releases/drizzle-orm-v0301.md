---
url: https://orm.drizzle.team/docs/latest-releases/drizzle-orm-v0301
title: "Drizzle Orm V0301"
description: ""
access_date: 2026-08-03T19:43:09.006Z
current_date: 2026-08-03T19:43:09.006Z
---

## DrizzleORM v0.30.1 release

## New Features

### 🎉 OP-SQLite driver Support

Usage Example

```ts
import { open } from '@op-engineering/op-sqlite';
import { drizzle } from 'drizzle-orm/op-sqlite';

const opsqlite = open({
    name: 'myDB',
});

const db = drizzle(opsqlite);

await db.select().from(users);
```

For more usage and setup details, please check our [op-sqlite docs](../sqlite/get-started-sqlite.md#op-sqlite)

## Fixes

- Migration hook fixed for Expo driver
