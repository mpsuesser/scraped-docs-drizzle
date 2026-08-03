---
url: https://orm.drizzle.team/docs/sqlite/get-started-sqlite
title: "Get Started Sqlite"
description: ""
access_date: 2026-08-03T19:00:22.305Z
current_date: 2026-08-03T19:00:22.305Z
---

## Drizzle <> SQLite

Drizzle has native support for SQLite connections with the `libsql`, `node:sqlite` and `better-sqlite3` drivers.

There are a few differences between the `libsql` and `node:sqlite` / `better-sqlite3` drivers that we discovered while using both and integrating them with the Drizzle ORM. For example:

At the driver level, there may not be many differences between the two, but the main one is that `libSQL` can connect to both SQLite files and `Turso` remote databases. LibSQL is a fork of SQLite that offers a bit more functionality compared to standard SQLite, such as:

- More ALTER statements are available with the `libSQL` driver, allowing you to manage your schema more easily than with just `better-sqlite3`.
- You can configure the encryption at rest feature natively.
- A large set of extensions supported by the SQLite database is also supported by `libSQL`.

## libsql

#### Step 1 - Install packages

```shell
npm i drizzle-orm@rc @libsql/client
npm i -D drizzle-kit@rc
```

#### Step 2 - Initialize the driver

Drizzle has native support for all @libsql/client driver variations:

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

#### Step 3 - make a query

```typescript
import { drizzle } from 'drizzle-orm/libsql';

const db = drizzle(process.env.DATABASE_URL);
 
const result = await db.all('select 1');
```

If you need a synchronous connection, you can use our additional connection API, where you specify a driver connection and pass it to the Drizzle instance.

```typescript
import { drizzle } from 'drizzle-orm/libsql';
import { createClient } from '@libsql/client';

const client = createClient({ url: process.env.DATABASE_URL, authToken: process.env.DATABASE_AUTH_TOKEN });
const db = drizzle({ client });

const result = await db.all('select 1');
```

## node:sqlite

#### Step 1 - Install packages

```shell
npm i drizzle-orm@rc
npm i -D drizzle-kit@rc
```

#### Step 2 - Initialize the driver and make a query

```typescript
import { drizzle } from 'drizzle-orm/node-sqlite';

const db = drizzle(process.env.DB_FILE_NAME);

const result = await db.execute('select 1');
```

If you need to provide your existing driver:

```typescript
import { drizzle } from 'drizzle-orm/node-sqlite';
import { DatabaseSync } from 'node:sqlite';

const sqlite = new DatabaseSync('sqlite.db');
const db = drizzle({ client: sqlite });

const result = await db.execute('select 1');
```

## better-sqlite3

#### Step 1 - Install packages

```shell
npm i drizzle-orm@rc better-sqlite3
npm i -D drizzle-kit@rc @types/better-sqlite3
```

#### Step 2 - Initialize the driver and make a query

```typescript
import { drizzle } from 'drizzle-orm/better-sqlite3';

const db = drizzle(process.env.DB_FILE_NAME);

const result = await db.all('select 1');
```

If you need to provide your existing driver:

```typescript
import { drizzle } from 'drizzle-orm/better-sqlite3';
import Database from 'better-sqlite3';

const sqlite = new Database('sqlite.db');
const db = drizzle({ client: sqlite });

const result = await db.all('select 1');
```
