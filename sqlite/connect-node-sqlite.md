---
url: https://orm.drizzle.team/docs/sqlite/connect-node-sqlite
title: "Connect Node Sqlite"
description: ""
access_date: 2026-08-03T18:54:19.022Z
current_date: 2026-08-03T18:54:19.022Z
---

## Drizzle <> Node SQLite

This guide assumes familiarity with:

- Database [connection basics](connect-overview.md) with Drizzle
- Node - [website](https://nodejs.org/)
- Node SQLite driver - [docs](https://nodejs.org/api/sqlite.html)

Drizzle ORM natively supports **[`node:sqlite`](https://nodejs.org/api/sqlite.html)** module

We embrace SQL dialects and dialect specific drivers and syntax and unlike any other ORM, for synchronous drivers like `node:sqlite` we have both **async** and **sync** APIs and we mirror most popular SQLite-like `all`, `get`, `values` and `run` query methods syntax.

#### Step 1 - Install packages

```shell
npm i drizzle-orm@rc
npm i -D drizzle-kit@rc
```

#### Step 2 - Initialize the driver and make a query

```typescript
import { drizzle } from 'drizzle-orm/node-sqlite';

const db = drizzle("sqlite.db");

const result = await db.select().from(...);
```

If you need to provide your existing driver:

```typescript
import { drizzle } from 'drizzle-orm/node-sqlite';
import { DatabaseSync } from 'node:sqlite';

const sqlite = new DatabaseSync('sqlite.db');
const db = drizzle({ client: sqlite });

const result = await db.select().from(...);
```

If you want to use **sync** APIs:

```typescript
import { drizzle } from 'drizzle-orm/node-sqlite';
import { DatabaseSync } from 'node:sqlite';

const sqlite = new DatabaseSync('sqlite.db');
const db = drizzle({ client: sqlite });

const result = db.select().from(users).all();
const result = db.select().from(users).get();
const result = db.select().from(users).values();
const result = db.select().from(users).run();
```
