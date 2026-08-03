---
url: https://orm.drizzle.team/docs/connect-prisma-postgres
title: "Connect Prisma Postgres"
description: ""
access_date: 2026-08-03T19:00:22.305Z
current_date: 2026-08-03T19:00:22.305Z
---

## Drizzle <> Prisma Postgres

This guide assumes familiarity with:

- Database [connection basics](connect-overview.md) with Drizzle
- Prisma Postgres serverless database - [website](https://prisma.io/postgres)
- Prisma Postgres direct connections - [docs](https://www.prisma.io/docs/postgres/database/direct-connections)
- Drizzle PostgreSQL drivers - [docs](get-started-postgresql.md)

Prisma Postgres is a serverless database built on [unikernels](https://www.prisma.io/blog/announcing-prisma-postgres-early-access). It has a large free tier, [operation-based pricing](https://www.prisma.io/blog/operations-based-billing) and no cold starts.

You can connect to it using either the [`node-postgres`](https://node-postgres.com/) or [`postgres.js`](https://github.com/porsager/postgres) drivers for PostgreSQL.

Prisma Postgres also has a [serverless driver](https://www.prisma.io/docs/postgres/database/serverless-driver) that will be supported with Drizzle ORM in the future.

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
import { drizzle } from "drizzle-orm/node-postgres";
import { Pool } from "pg";

const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
});
const db = drizzle({ client: pool });
 
const result = await db.execute('select 1');
```
