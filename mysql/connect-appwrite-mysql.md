---
url: https://orm.drizzle.team/docs/mysql/connect-appwrite-mysql
title: "Connect Appwrite Mysql"
description: ""
access_date: 2026-09-18T11:13:17.168Z
current_date: 2026-09-18T11:13:17.168Z
---

## Drizzle <> Appwrite MySQL

This guide assumes familiarity with:

- Database [connection basics](connect-overview.md) with Drizzle
- Appwrite native MySQL database - [website](https://appwrite.io/) & [docs](https://appwrite.io/docs/products/databases/mysql)
- Appwrite MySQL connections - [docs](https://appwrite.io/docs/products/databases/mysql/connections)
- Drizzle MySQL drivers - [docs](get-started-mysql.md)

Appwrite offers dedicated, native MySQL databases that you connect to directly over TLS. There is no Appwrite-specific driver or abstraction layer, so you can use Drizzle with the [`mysql2`](https://github.com/sidorares/node-mysql2) driver through the `drizzle-orm/mysql2` package, the same way you would with any MySQL server.

To get your connection string, open your database in the Appwrite Console and click **Credentials**, or fetch it from the Appwrite API with `mysql.get()`. The username is `admin.<hash>`, where the hash matches the one in the hostname, and the database name is always `default`. Appwrite’s proxy routes your connection by this username, so always use the full value from the connection string.

```plaintext
DATABASE_URL="mysql://admin.<hash>:<password>@db-<hash>.<region>.appwrite.center:3306/default?ssl=true"
```

#### Step 1 - Install packages

```shell
npm i drizzle-orm@rc mysql2
npm i -D drizzle-kit@rc
```

#### Step 2 - Initialize the driver and make a query

```typescript
import { drizzle } from "drizzle-orm/mysql2";

const db = drizzle(process.env.DATABASE_URL);

const response = await db.select().from(...)
```

If you need to provide your existing driver:

```typescript
import { drizzle } from "drizzle-orm/mysql2";
import mysql from "mysql2/promise";

const connection = await mysql.createConnection(process.env.DATABASE_URL);

const db = drizzle({ client: connection });
```

IMPORTANT

Connections on Appwrite Cloud are encrypted with TLS. If your runtime or `mysql2` configuration does not infer TLS from the connection string, pass `ssl: { rejectUnauthorized: true }` in the connection options.

For the built in `migrate` function with DDL migrations we and drivers strongly encourage you to use a single `client` connection.

#### Connection pooling for serverless

Appwrite runs a [connection pooler](https://appwrite.io/docs/products/databases/mysql/connection-pooling) on port `6033` of the same hostname. In serverless and edge environments, route your application traffic through the pooler so short-lived instances don’t exhaust the engine’s connection limit, and keep Drizzle Kit pointed at the direct engine port `3306`, because migrations issue DDL that needs a session-level connection.

```plaintext
# Runtime: pooled, transaction mode
DATABASE_URL="mysql://admin.<hash>:<password>@db-<hash>.<region>.appwrite.center:6033/default?ssl=true"

# Migrations & introspection: direct connection to the engine
DIRECT_URL="mysql://admin.<hash>:<password>@db-<hash>.<region>.appwrite.center:3306/default?ssl=true"
```
```typescript
import { defineConfig } from 'drizzle-kit';

export default defineConfig({
  dialect: 'mysql',
  schema: './src/schema.ts',
  out: './drizzle',
  dbCredentials: {
    url: process.env.DIRECT_URL!,
  },
});
```

The pooler defaults to transaction mode, which does not keep a backend connection across statements. Drizzle’s regular query builder sends text queries and works with transaction pooling, but session-level features such as explicit prepared queries, user variables and temporary tables need the direct port or a session-mode pooler.
