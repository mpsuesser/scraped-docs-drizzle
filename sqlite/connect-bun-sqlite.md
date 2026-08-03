---
url: https://orm.drizzle.team/docs/sqlite/connect-bun-sqlite
title: "Connect Bun Sqlite"
description: ""
access_date: 2026-08-03T19:00:22.305Z
current_date: 2026-08-03T19:00:22.305Z
---

## Drizzle <> Bun SQLite

This guide assumes familiarity with:

- Database [connection basics](connect-overview.md) with Drizzle
- Bun - [website](https://bun.sh/docs)
- Bun SQLite driver - [docs](https://bun.sh/docs/api/sqlite)

According to the **[official website](https://bun.sh/)**, Bun is a fast all-in-one JavaScript runtime.

Drizzle ORM natively supports **[`bun:sqlite`](https://bun.sh/docs/api/sqlite)** module and it’s crazy fast 🚀

We embrace SQL dialects and dialect specific drivers and syntax and unlike any other ORM, for synchronous drivers like `bun:sqlite` we have both **async** and **sync** APIs and we mirror most popular SQLite-like `all`, `get`, `values` and `run` query methods syntax.

#### Step 1 - Install packages

```shell
npm i drizzle-orm@rc
npm i -D drizzle-kit@rc
```

#### Step 2 - Initialize the driver and make a query

```typescript
import { drizzle } from 'drizzle-orm/bun-sqlite';

const db = drizzle();

const result = await db.select().from(...);
```

If you need to provide your existing driver:

```typescript
import { drizzle } from 'drizzle-orm/bun-sqlite';
import { Database } from 'bun:sqlite';

const sqlite = new Database('sqlite.db');
const db = drizzle({ client: sqlite });

const result = await db.select().from(...);
```

If you want to use **sync** APIs:

```typescript
import { drizzle } from 'drizzle-orm/bun-sqlite';
import { Database } from 'bun:sqlite';

const sqlite = new Database('sqlite.db');
const db = drizzle({ client: sqlite });

const result = db.select().from(users).all();
const result = db.select().from(users).get();
const result = db.select().from(users).values();
const result = db.select().from(users).run();
```
