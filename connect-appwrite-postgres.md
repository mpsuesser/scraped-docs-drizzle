---
url: https://orm.drizzle.team/docs/connect-appwrite-postgres
title: "Connect Appwrite Postgres"
description: ""
access_date: 2026-09-18T11:13:17.168Z
current_date: 2026-09-18T11:13:17.168Z
---

## Drizzle <> Appwrite Postgres

This guide assumes familiarity with:

- Database [connection basics](connect-overview.md) with Drizzle
- Appwrite native PostgreSQL database - [website](https://appwrite.io/) & [docs](https://appwrite.io/docs/products/databases/postgresql)
- Appwrite PostgreSQL connections - [docs](https://appwrite.io/docs/products/databases/postgresql/connections)
- Drizzle PostgreSQL drivers - [docs](get-started-postgresql.md)

Appwrite offers dedicated, native PostgreSQL databases that you connect to directly over TLS. There is no Appwrite-specific driver or abstraction layer, so you can use Drizzle with either the [`node-postgres`](https://node-postgres.com/) or [`postgres.js`](https://github.com/porsager/postgres) driver, the same way you would with any PostgreSQL server.

To get your connection string, open your database in the Appwrite Console and click **Credentials**. The **Drizzle** tab contains a ready-made snippet, and the **DSN** and **.env** tabs contain the raw connection string. You can also fetch it from the Appwrite API with `postgresql.get()`. The connection string already includes `sslmode=require`, so no extra TLS configuration is needed.

```plaintext
DATABASE_URL="postgresql://admin:<password>@db-<hash>.<region>.appwrite.center:5432/<database>?sslmode=require"
```

#### Step 1 - Install packages

node-postgres (pg)

postgres.js

```shell
npm i drizzle-orm@rc pg
npm i -D drizzle-kit@rc
```

#### Step 2 - Initialize the driver and make a query

```typescript
// Make sure to install the 'pg' package
import { drizzle } from 'drizzle-orm/node-postgres';

const db = drizzle(process.env.DATABASE_URL);

const result = await db.execute('select 1');
```

If you need to provide your existing drivers:

```typescript
// Make sure to install the 'pg' package
import { drizzle } from "drizzle-orm/node-postgres";
import { Pool } from "pg";

const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
});
const db = drizzle({ client: pool });

const result = await db.execute('select 1');
```

#### Connection pooling for serverless

Appwrite runs a [connection pooler](https://appwrite.io/docs/products/databases/postgresql/connection-pooling) on port `6432` of the same hostname. In serverless and edge environments, route your application traffic through the pooler so short-lived instances don’t exhaust the engine’s connection limit, and keep Drizzle Kit pointed at the direct engine port `5432`, because migrations issue DDL that needs a session-level connection.

```plaintext
# Runtime: pooled, transaction mode
DATABASE_URL="postgresql://admin:<password>@db-<hash>.<region>.appwrite.center:6432/<database>?sslmode=require"

# Migrations & introspection: direct connection to the engine
DIRECT_URL="postgresql://admin:<password>@db-<hash>.<region>.appwrite.center:5432/<database>?sslmode=require"
```
```typescript
import { defineConfig } from 'drizzle-kit';

export default defineConfig({
  dialect: 'postgresql',
  schema: './src/schema.ts',
  out: './drizzle',
  dbCredentials: {
    url: process.env.DIRECT_URL!,
  },
});
```

IMPORTANT

The pooler defaults to **transaction mode**, which does not keep a backend connection across statements, so server-side prepared statements are unavailable. With `postgres.js`, build the client yourself with `prepare: false` when connecting through the pooler:

```typescript
import { drizzle } from 'drizzle-orm/postgres-js';
import postgres from 'postgres';

const queryClient = postgres(process.env.DATABASE_URL, { prepare: false });
const db = drizzle({ client: queryClient });
```

If you rely on prepared statements, advisory locks, `LISTEN` / `NOTIFY`, temporary tables or `SET LOCAL`, switch the pooler to session mode or use the direct port.
