---
url: https://orm.drizzle.team/docs/mssql/get-started-mssql
title: "Get Started Mssql"
description: ""
access_date: 2026-08-03T19:38:26.356Z
current_date: 2026-08-03T19:38:26.356Z
---

## Drizzle <> MSSQL

This guide assumes familiarity with:

- Database [connection basics](connect-overview.md) with Drizzle
- node-mssql [basics](https://github.com/tediousjs/node-mssql)

Drizzle has native support for MSSQL connections with the `mssql` driver.

#### Step 1 - Install packages

```shell
npm i drizzle-orm@rc mssql
npm i -D drizzle-kit@rc
```

#### Step 2 - Initialize the driver and make a query

```typescript
// Make sure to install the 'mssql' package 
import { drizzle } from 'drizzle-orm/node-mssql';

const db = drizzle(process.env.DATABASE_URL);
 
const result = await db.execute('select 1');
```

IMPORTANT

As long as the `node-mssql` driver requires `await` on `Pool` initialization, we need to `await` it before each request - unless you are providing your own Pool instance to Drizzle. In that case, when you want to access `db.$client`, you first need to `await` it, and then use it

```ts
const awaitedClient = await db.$client.$instance();
const response = awaitedClient.query...;
```

If you need to provide your existing driver:

```typescript
// Make sure to install the 'mssql' package 
import { drizzle } from "drizzle-orm/node-mssql";
import { connect } from 'mssql';

const pool = await connect(process.env.DATABASE_URL!);
const db = drizzle({ client: pool });
 
const result = await db.execute('select 1');
```
