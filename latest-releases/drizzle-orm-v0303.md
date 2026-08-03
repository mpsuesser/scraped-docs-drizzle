---
url: https://orm.drizzle.team/docs/latest-releases/drizzle-orm-v0303
title: "Drizzle Orm V0303"
description: ""
access_date: 2026-08-03T19:00:22.305Z
current_date: 2026-08-03T19:00:22.305Z
---

## DrizzleORM v0.30.3 release

## New Features

- Added raw query support (`db.execute(...)`) to batch API in Neon HTTP driver

To get started with Neon and Drizzle follow the [documentation](../get-started-postgresql.md#neon)

## Fixes

- Fixed `@neondatabase/serverless` HTTP driver types issue ([#1945](https://github.com/drizzle-team/drizzle-orm/issues/1945), [neondatabase/serverless#66](https://github.com/neondatabase/serverless/issues/66))
- Fixed sqlite-proxy driver `.run()` result

To get started with SQLite proxy driver and Drizzle follow the [documentation](../sqlite/get-started-sqlite.md#http-proxy)
