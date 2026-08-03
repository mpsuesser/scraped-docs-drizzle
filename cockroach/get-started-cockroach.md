---
url: https://orm.drizzle.team/docs/cockroach/get-started-cockroach
title: "Get Started Cockroach"
description: ""
access_date: 2026-08-03T19:38:26.356Z
current_date: 2026-08-03T19:38:26.356Z
---

## Drizzle <> CockroachDB

This guide assumes familiarity with:

- Database [connection basics](connect-overview.md) with Drizzle
- node-postgres [basics](https://node-postgres.com/)

Drizzle has native support for CockroachDB connections with the `node-postgres` driver.

#### Step 1 - Install packages

```shell
npm i drizzle-orm@rc pg
npm i -D drizzle-kit@rc @types/pg
```

#### Step 2 - Initialize the driver and make a query

```typescript
// Make sure to install the 'pg' package 
import { drizzle } from 'drizzle-orm/cockroach';

const db = drizzle(process.env.DATABASE_URL);
 
const result = await db.execute('select 1');
```

If you need to provide your existing driver:

```typescript
// Make sure to install the 'pg' package 
import { drizzle } from "drizzle-orm/cockroach";
import { Pool } from "pg";

const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
});
const db = drizzle({ client: pool });
 
const result = await db.execute('select 1');
```
