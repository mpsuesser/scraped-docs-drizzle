---
url: https://orm.drizzle.team/docs/sqlite/connect-turso
title: "Connect Turso"
description: ""
access_date: 2026-08-03T18:54:19.022Z
current_date: 2026-08-03T18:54:19.022Z
---

## Drizzle <> Turso Cloud

This guide assumes familiarity with:

- Database [connection basics](connect-overview.md) with Drizzle
- Turso Cloud - [website](https://docs.turso.tech/turso-cloud)
- Turso Cloud driver - [website](https://docs.turso.tech/sdk/ts/reference) & [GitHub](https://github.com/tursodatabase/libsql-client-ts)

According to the **[official website](https://turso.tech/drizzle)**, Turso is a **[libSQL](https://github.com/libsql/libsql)** powered edge SQLite database as a service.

Drizzle ORM natively supports libSQL driver. We embrace SQL dialects and dialect specific drivers and syntax and mirror most popular SQLite-like `all`, `get`, `values` and `run` query methods syntax.

#### Step 1 - Install packages

```shell
npm i drizzle-orm@rc @libsql/client
npm i -D drizzle-kit@rc
```

#### Step 2 - Initialize the driver

Drizzle has native support for all `@libsql/client` driver variations:

|  |  |
| --- | --- |
| `@libsql/client` | defaults to `node` import, automatically changes to `web` if `target` or `platform` is set for bundler, e.g. `esbuild --platform=browser` |
| `@libsql/client/node` | `node` compatible module, supports `:memory:`, `file`, `wss`, `http` and `turso` connection protocols |
| `@libsql/client/web` | module for fullstack web frameworks like `next`, `nuxt`, `astro`, etc. |
| `@libsql/client/http` | module for `http` and `https` connection protocols |
| `@libsql/client/ws` | module for `ws` and `wss` connection protocols |
| `@libsql/client/sqlite3` | module for `:memory:` and `file` connection protocols |
| `@libsql/client-wasm` | Separate experimental package for WASM |

```typescript
import { drizzle } from 'drizzle-orm/libsql';

const db = drizzle({ connection: {
  url: process.env.DATABASE_URL, 
  authToken: process.env.DATABASE_AUTH_TOKEN 
}});
```

If you need to provide your existing driver:

```typescript
import { drizzle } from 'drizzle-orm/libsql';
import { createClient } from '@libsql/client';

const client = createClient({ 
  url: process.env.DATABASE_URL,
  authToken: process.env.DATABASE_AUTH_TOKEN
});

const db = drizzle({ client });

const result = await db.select().from(users).all()
```

#### Step 3 - make a query

```ts
import { drizzle } from 'drizzle-orm/libsql';
import * as s from 'drizzle-orm/sqlite-core';

const db = drizzle({ connection: {
  url: process.env.DATABASE_URL, 
  authToken: process.env.DATABASE_AUTH_TOKEN 
}});

const users = s.sqliteTable("users", {
  id: s.integer(),
  name: s.text(),
})

const result = await db.select().from(users);
```
