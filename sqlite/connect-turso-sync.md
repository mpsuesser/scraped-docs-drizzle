---
url: https://orm.drizzle.team/docs/sqlite/connect-turso-sync
title: "Connect Turso Sync"
description: ""
access_date: 2026-08-03T19:00:22.305Z
current_date: 2026-08-03T19:00:22.305Z
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
