---
url: https://orm.drizzle.team/docs/sqlite/connect-turso-serverless
title: "Connect Turso Serverless"
description: ""
access_date: 2026-10-23T04:35:39.137Z
current_date: 2026-10-23T04:35:39.137Z
---

## Drizzle <> Turso Database Serverless

This guide assumes familiarity with:

- Database [connection basics](connect-overview.md) with Drizzle
- Turso Database - [website](https://docs.turso.tech/introduction)
- Turso Database serverless driver - [website](https://docs.turso.tech/sdk/ts/quickstart#recommended-@tursodatabase/serverless-remote-/-over-the-wire) & [GitHub](https://github.com/tursodatabase/turso/tree/main/serverless/javascript)

According to the **[official website](https://docs.turso.tech/introduction)**, Turso is the small database to power your big dreams in the age of AI.

#### Step 1 - Install packages

```shell
npm i drizzle-orm@rc @tursodatabase/serverless
npm i -D drizzle-kit@rc
```

#### Step 2 - Initialize the driver and make a query

```typescript
import { drizzle } from 'drizzle-orm/tursodatabase-serverless';

const db = drizzle({
    connection: {
        url: process.env.TURSO_URL,
        authToken: process.env.TURSO_TOKEN
    }
});

const result = await db.all('select 1');
```

If you need to provide your existing drivers:

```typescript
import { connect } from '@tursodatabase/serverless';
import { drizzle } from 'drizzle-orm/tursodatabase-serverless';

const client = connect({
    url: process.env.TURSO_URL,
    authToken: process.env.TURSO_TOKEN
});
const db = drizzle({ client });

const result = await db.all('select 1');
```

#### Transaction modes

You can set the transaction mode with the `behavior` option. Drizzle passes it to the driver, which opens the transaction with `BEGIN <mode>`. Without `behavior`, a plain `BEGIN` is used, which SQLite runs as `deferred`.

```typescript
import { drizzle } from 'drizzle-orm/tursodatabase-serverless';
import { users } from './schema';

const db = drizzle({ connection: { url: process.env.TURSO_URL, authToken: process.env.TURSO_TOKEN } });

await db.transaction(async (tx) => {
  await tx.insert(users).values({ name: 'Dan' });
}, {
  behavior: 'immediate', // 'deferred' | 'immediate' | 'exclusive' | 'concurrent'
});
```
- `deferred` - takes the lock on the first read or write
- `immediate` - takes the write lock at the start of the transaction
- `exclusive` - takes the write lock at the start and blocks other connections
- `concurrent` - lets several write transactions run at the same time. It needs MVCC to be turned on in Turso Database

Nested `tx.transaction()` calls use savepoints and don’t take a `behavior` option.
