---
url: https://orm.drizzle.team/docs/connect-pglite
title: "Connect Pglite"
description: ""
access_date: 2026-08-03T19:00:22.305Z
current_date: 2026-08-03T19:00:22.305Z
---

## Drizzle <> PGlite

This guide assumes familiarity with:

- Database [connection basics](connect-overview.md) with Drizzle
- ElectricSQL - [website](https://electric-sql.com/)
- PgLite driver - [docs](https://pglite.dev/) & [GitHub](https://github.com/electric-sql/pglite)

According to the **[official repo](https://github.com/electric-sql/pglite)**, PGlite is a WASM Postgres build packaged into a TypeScript client library that enables you to run Postgres in the browser, Node.js and Bun, with no need to install any other dependencies. It is only 2.6mb gzipped.

It can be used as an ephemeral in-memory database, or with persistence either to the file system (Node/Bun) or indexedDB (Browser).

Unlike previous “Postgres in the browser” projects, PGlite does not use a Linux virtual machine - it is simply Postgres in WASM.

#### Step 1 - Install packages

```shell
npm i drizzle-orm@rc @electric-sql/pglite
npm i -D drizzle-kit@rc
```

#### Step 2 - Initialize the driver and make a query

```typescript
import { drizzle } from 'drizzle-orm/pglite';

const db = drizzle();

await db.select().from(...);
```

If you need to provide your existing driver:

```typescript
import { PGlite } from '@electric-sql/pglite';
import { drizzle } from 'drizzle-orm/pglite';

// In-memory Postgres
const client = new PGlite();
const db = drizzle({ client });

await db.select().from(users);
```
