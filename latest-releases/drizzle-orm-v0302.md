---
url: https://orm.drizzle.team/docs/latest-releases/drizzle-orm-v0302
title: "Drizzle Orm V0302"
description: ""
access_date: 2026-08-03T19:43:09.006Z
current_date: 2026-08-03T19:43:09.006Z
---

## DrizzleORM v0.30.2 release

## Improvements

LibSQL migrations have been updated to utilize batch execution instead of transactions. As stated in the [documentation](https://docs.turso.tech/sdk/ts/reference#batch-transactions), LibSQL now supports batch operations

> A batch consists of multiple SQL statements executed sequentially within an implicit transaction. The backend handles the transaction: success commits all changes, while any failure results in a full rollback with no modifications.

To get started with Turso and Drizzle follow the [documentation](../sqlite/get-started-sqlite.md#turso)

## Fixes

- \[Sqlite\] Fix findFirst query for bun:sqlite ([#1885](https://github.com/drizzle-team/drizzle-orm/pull/1885))

To get started with Bun SQLite and Drizzle follow the [documentation](../sqlite/get-started-sqlite.md#bun-sqlite)
