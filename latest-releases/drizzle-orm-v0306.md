---
url: https://orm.drizzle.team/docs/latest-releases/drizzle-orm-v0306
title: "Drizzle Orm V0306"
description: ""
access_date: 2026-08-03T19:00:22.305Z
current_date: 2026-08-03T19:00:22.305Z
---

## DrizzleORM v0.30.6 release

## New Features

### PGlite driver Support

[PGlite](https://github.com/electric-sql/pglite) is a WASM Postgres build packaged into a TypeScript client library that enables you to run Postgres in the browser, Node.js and Bun, with no need to install any other dependencies. It is only 2.6mb gzipped.

It can be used as an ephemeral in-memory database, or with persistence either to the file system (Node/Bun) or indexedDB (Browser).

Unlike previous “Postgres in the browser” projects, PGlite does not use a Linux virtual machine - it is simply Postgres in WASM.

Read get started with Drizzle and PGlite guide [here](../get-started-postgresql.md#pglite).

Usage Example

```ts
import { PGlite } from '@electric-sql/pglite';
import { drizzle } from 'drizzle-orm/pglite';
import { users } from './schema';

// In-memory Postgres
const client = new PGlite();
const db = drizzle(client);

await db.select().from(users);
```
