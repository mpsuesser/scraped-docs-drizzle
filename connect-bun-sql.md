---
url: https://orm.drizzle.team/docs/connect-bun-sql
title: "Connect Bun Sql"
description: ""
access_date: 2026-08-03T19:08:17.353Z
current_date: 2026-08-03T19:08:17.353Z
---

## Drizzle <> Bun SQL

This guide assumes familiarity with:

- Database [connection basics](connect-overview.md) with Drizzle
- Bun - [website](https://bun.sh/docs)
- Bun SQL - native bindings for working with PostgreSQL databases - [read here](https://bun.sh/docs/api/sql)

According to the **[official website](https://bun.sh/)**, Bun is a fast all-in-one JavaScript runtime.

Drizzle ORM natively supports **[`bun sql`](https://bun.sh/docs/api/sql)** module and it’s crazy fast 🚀

#### Step 1 - Install packages

```shell
npm i drizzle-orm@rc
npm i -D drizzle-kit@rc
```

#### Step 2 - Initialize the driver and make a query

```typescript
import 'dotenv/config';
import { drizzle } from 'drizzle-orm/bun-sql';

const db = drizzle(process.env.DATABASE_URL);

const result = await db.select().from(...);
```

If you need to provide your existing driver:

```typescript
import 'dotenv/config';
import { drizzle } from 'drizzle-orm/bun-sql';
import { SQL } from 'bun';

const client = new SQL(process.env.DATABASE_URL!);
const db = drizzle({ client });
```
