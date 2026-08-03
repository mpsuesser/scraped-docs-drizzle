---
url: https://orm.drizzle.team/docs/connect-neon
title: "Connect Neon"
description: ""
access_date: 2026-08-03T19:08:17.353Z
current_date: 2026-08-03T19:08:17.353Z
---

## Drizzle <> Neon Postgres

This guide assumes familiarity with:

- Database [connection basics](connect-overview.md) with Drizzle
- Neon serverless database - [website](https://neon.tech/)
- Neon serverless driver - [docs](https://neon.tech/docs/serverless/serverless-driver) & [GitHub](https://github.com/neondatabase/serverless)
- Drizzle PostgreSQL drivers - [docs](get-started-postgresql.md)

Drizzle has native support for Neon connections with the `neon-http` and `neon-websockets` drivers. These use the **neon-serverless** driver under the hood.

With the `neon-http` and `neon-websockets` drivers, you can access a Neon database from serverless environments over HTTP or WebSockets instead of TCP.  
Querying over HTTP is faster for single, non-interactive transactions.

If you need session or interactive transaction support, or a fully compatible drop-in replacement for the `pg` driver, you can use the WebSocket-based `neon-serverless` driver.  
You can connect to a Neon database directly using [Postgres](get-started/postgresql-new.md)

For an example of using Drizzle ORM with the Neon Serverless driver in a Cloudflare Worker, **[see here.](http://driz.link/neon-cf-ex)**  
To use Neon from a serverful environment, you can use the node-postgres or Postgres.js drivers, as described in Neon’s **[official Node.js docs](https://neon.tech/docs/guides/node)**

#### Step 1 - Install packages

@neondatabase/serverless

node-postgres

postgres.js

```shell
npm i drizzle-orm@rc @neondatabase/serverless
npm i -D drizzle-kit@rc
```

#### Step 2 - Initialize the driver and make a query

```typescript
// Make sure to install the '@neondatabase/serverless' package 
import { drizzle } from 'drizzle-orm/neon-http';

const db = drizzle(process.env.DATABASE_URL);

const result = await db.execute('select 1');
```

If you need to provide your existing drivers:

```typescript
// Make sure to install the '@neondatabase/serverless' package 
import { neon } from '@neondatabase/serverless';
import { drizzle } from 'drizzle-orm/neon-http';

const sql = neon(process.env.DATABASE_URL!);
const db = drizzle({ client: sql });

const result = await db.execute('select 1');
```
