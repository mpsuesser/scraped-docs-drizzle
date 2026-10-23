---
url: https://orm.drizzle.team/docs/sqlite/connect-turso-sync
title: "Connect Turso Sync"
description: ""
access_date: 2026-10-23T04:35:39.137Z
current_date: 2026-10-23T04:35:39.137Z
---

## Drizzle <> Turso Database Sync

This guide assumes familiarity with:

- Database [connection basics](connect-overview.md) with Drizzle
- Turso Database - [website](https://docs.turso.tech/introduction)
- Turso Database sync driver - [GitHub](https://github.com/tursodatabase/turso/tree/main/bindings/javascript/sync)

According to the **[official website](https://docs.turso.tech/introduction)**, Turso is the small database to power your big dreams in the age of AI.

#### Step 1 - Install packages

```shell
npm i drizzle-orm@rc @tursodatabase/sync
npm i -D drizzle-kit@rc
```

#### Step 2 - Initialize the driver and make a query

```typescript
import { drizzle } from 'drizzle-orm/tursodatabase-sync';

const db = drizzle({
    connection: {
        path: 'local.db',                // path used as a prefix for local files created by sync-engine
        url: 'https://<db>.turso.io',    // URL of the remote database: turso db show <db>
        authToken: '...',                // auth token issued from the Turso Cloud: turso db tokens create <db>
        clientName: 'turso-sync-example' // arbitrary client name
    }
});

const result = await db.all('select 1');
```

If you need to provide your existing drivers:

```typescript
import { connect } from '@tursodatabase/sync';
import { drizzle } from 'drizzle-orm/tursodatabase-sync';

const client = await connect({
    path: 'local.db',                // path used as a prefix for local files created by sync-engine
    url: 'https://<db>.turso.io',    // URL of the remote database: turso db show <db>
    authToken: '...',                // auth token issued from the Turso Cloud: turso db tokens create <db>
    clientName: 'turso-sync-example' // arbitrary client name
});
const db = drizzle({ client });

const result = await db.all('select 1');
```

#### Transaction modes

You can set the transaction mode with the `behavior` option. Drizzle passes it to the driver, which opens the transaction with `BEGIN <mode>`. Without `behavior`, a plain `BEGIN` is used, which SQLite runs as `deferred`.

```typescript
import { drizzle } from 'drizzle-orm/tursodatabase-sync';
import { users } from './schema';

const db = drizzle({ connection: { path: 'local.db', url: 'https://<db>.turso.io', authToken: '...' } });

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
