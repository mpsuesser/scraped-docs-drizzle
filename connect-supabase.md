---
url: https://orm.drizzle.team/docs/connect-supabase
title: "Connect Supabase"
description: ""
access_date: 2026-08-03T19:38:26.356Z
current_date: 2026-08-03T19:38:26.356Z
---

## Drizzle <> Supabase

This guide assumes familiarity with:

- Database [connection basics](connect-overview.md) with Drizzle
- Drizzle PostgreSQL drivers - [docs](get-started-postgresql.md)

According to the , Supabase is an open source Firebase alternative for building secure and performant Postgres backends with minimal configuration.

Checkout official **[Supabase + Drizzle](https://supabase.com/docs/guides/database/drizzle)** docs.

#### Step 1 - Install packages

```shell
npm i drizzle-orm@rc postgres
npm i -D drizzle-kit@rc
```

#### Step 2 - Initialize the driver and make a query

```typescript
import { drizzle } from 'drizzle-orm/postgres-js'

const db = drizzle(process.env.DATABASE_URL);

const allUsers = await db.select().from(...);
```

If you need to provide your existing driver:

```typescript
import { drizzle } from 'drizzle-orm/postgres-js'
import postgres from 'postgres'

const client = postgres(process.env.DATABASE_URL)
const db = drizzle({ client });

const allUsers = await db.select().from(...);
```

If you decide to use connection pooling via Supabase (described [here](https://supabase.com/docs/guides/database/connecting-to-postgres#poolers)), and have “Transaction” pool mode enabled, then ensure to turn off prepare, as prepared statements are not supported.

```typescript
import { drizzle } from 'drizzle-orm/postgres-js'
import postgres from 'postgres'

// Disable prefetch as it is not supported for "Transaction" pool mode 
const client = postgres(process.env.DATABASE_URL, { prepare: false })
const db = drizzle({ client });

const allUsers = await db.select().from(...);
```

Connect to your database using the Connection Pooler for **serverless environments**, and the Direct Connection for **long-running servers**.
