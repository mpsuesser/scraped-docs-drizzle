---
url: https://orm.drizzle.team/docs/connect-planetscale-postgres
title: "Connect Planetscale Postgres"
description: ""
access_date: 2026-08-03T19:00:22.305Z
current_date: 2026-08-03T19:00:22.305Z
---

## Drizzle <> PlanetScale Postgres

This guide assumes familiarity with:

- Database [connection basics](connect-overview.md) with Drizzle
- PlanetScale Postgres database - [docs](https://planetscale.com/docs/postgres)
- Drizzle PostgreSQL drivers - [docs](get-started-postgresql.md)

PlanetScale offers both MySQL (Vitess) and PostgreSQL databases. This page covers connecting to PlanetScale Postgres.

For PlanetScale MySQL, see the [PlanetScale MySQL connection guide](mysql/connect-planetscale.md).

With Drizzle ORM you can connect to PlanetScale Postgres using:

- The standard `node-postgres` driver
- The `@neondatabase/serverless` driver for serverless environments

For detailed instructions on creating a PlanetScale Postgres database and obtaining credentials, see the [PlanetScale Postgres documentation](https://planetscale.com/docs/postgres/tutorials/planetscale-postgres-drizzle).

## node-postgres

#### Step 1 - Install packages

```shell
npm i drizzle-orm@rc pg
npm i -D drizzle-kit@rc @types/pg
```

#### Step 2 - Initialize the driver and make a query

```typescript
import { drizzle } from 'drizzle-orm/node-postgres';

const db = drizzle(process.env.DATABASE_URL);

const result = await db.execute('select 1');
```

## Neon serverless driver

PlanetScale Postgres also supports connections via the [Neon serverless driver](https://planetscale.com/docs/postgres/connecting/neon-serverless-driver). This is a good option for serverless environments like Vercel Functions, Cloudflare Workers, or AWS Lambda.

The driver supports two modes:

- **HTTP mode** — Faster for single queries and non-interactive transactions
- **WebSocket mode** — Required for interactive transactions or session-based features

#### Step 1 - Install packages

```shell
npm i drizzle-orm@rc @neondatabase/serverless
npm i -D drizzle-kit@rc
```

#### Step 2 - Initialize the driver and make a query

```typescript
import { neon, neonConfig } from '@neondatabase/serverless';
import { drizzle } from 'drizzle-orm/neon-http';

// Required for PlanetScale Postgres connections
neonConfig.fetchEndpoint = (host) => \`https://${host}/sql\`;

const sql = neon(process.env.DATABASE_URL!);
const db = drizzle({ client: sql });

const result = await db.execute('select 1');
```

Connection URL format

postgresql://{username}:{password}@{host}:{port}/postgres?sslmode=verify-full

Connection ports

PlanetScale Postgres supports two connection ports:

`5432`: Direct connection to PostgreSQL. Total connections are limited by your cluster’s `max_connections` setting.

`6432`: Connection via PgBouncer for connection pooling. Recommended when you have many simultaneous connections.
