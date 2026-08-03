---
url: https://orm.drizzle.team/docs/connect-netlify-db
title: "Connect Netlify Db"
description: ""
access_date: 2026-08-03T19:43:09.006Z
current_date: 2026-08-03T19:43:09.006Z
---

## Drizzle <> Netlify Database

This guide assumes familiarity with:

- Database [connection basics](connect-overview.md) with Drizzle
- Netlify Database - [website](https://docs.netlify.com/build/data-and-storage/netlify-database/)

IMPORTANT

The Netlify Database driver is developed and maintained by the Netlify team.

Drizzle has native support for Netlify Database connections, that intelligently selects the optimal underlying Postgres driver based on the runtime environment.

Netlify Database is a managed Postgres database. On Netlify’s platform, the same application code runs in two very different contexts:

- Serverless functions: no persistent connections, best served by HTTP-based queries
- Server mode: persistent connections via node-postgres

This adapter abstracts that decision away so consumers can write `drizzle()` once and get the right driver automatically.

#### Step 1 - Install packages

```shell
npm i drizzle-orm@rc @netlify/database
npm i -D drizzle-kit@rc
```

#### Step 2 - Initialize the driver and make a query

```typescript
import { drizzle } from 'drizzle-orm/netlify-db';

// Connection string is set automatically by the platform
const db = drizzle();

const result = await db.execute('select 1');
```
