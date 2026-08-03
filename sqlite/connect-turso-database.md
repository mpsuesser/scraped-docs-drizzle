---
url: https://orm.drizzle.team/docs/sqlite/connect-turso-database
title: "Connect Turso Database"
description: ""
access_date: 2026-08-03T19:08:17.353Z
current_date: 2026-08-03T19:08:17.353Z
---

## Drizzle <> Turso Database

This guide assumes familiarity with:

- Database [connection basics](connect-overview.md) with Drizzle
- Turso Database - [website](https://docs.turso.tech/introduction)
- Turso Database driver - [website](https://docs.turso.tech/connect/javascript) & [GitHub](https://github.com/tursodatabase/turso/tree/main/bindings/javascript)

According to the **[official website](https://docs.turso.tech/introduction)**, Turso is the small database to power your big dreams in the age of AI.

#### Step 1 - Install packages

```shell
npm i drizzle-orm@rc @tursodatabase/database
npm i -D drizzle-kit@rc
```

#### Step 2 - Initialize the driver and make a query

```typescript
import { drizzle } from 'drizzle-orm/tursodatabase/database';

const db = drizzle('sqlite.db');

const result = await db.all('select 1');
```

If you need to provide your existing drivers:

```typescript
import { Database } from '@tursodatabase/database';
import { drizzle } from 'drizzle-orm/tursodatabase/database';

const client = new Database('sqlite.db');
const db = drizzle({ client });

const result = await db.all('select 1');
```
