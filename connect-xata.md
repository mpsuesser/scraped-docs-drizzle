---
url: https://orm.drizzle.team/docs/connect-xata
title: "Connect Xata"
description: ""
access_date: 2026-08-03T19:08:17.353Z
current_date: 2026-08-03T19:08:17.353Z
---

## Drizzle <> Xata

This guide assumes familiarity with:

- Database [connection basics](connect-overview.md) with Drizzle
- Drizzle PostgreSQL drivers - [docs](get-started-postgresql.md)

**[Xata](https://xata.io/)** is a PostgreSQL database platform designed to help developers operate and scale databases with enhanced productivity and performance. Xata provides features like instant copy-on-write database branches, zero-downtime schema changes, data anonymization, AI-powered performance monitoring, and BYOC.

Checkout official **[Xata + Drizzle](https://xata.io/documentation/quickstarts/drizzle)** docs.

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
