---
url: https://orm.drizzle.team/docs/sqlite/connect-sqlite-cloud
title: "Connect Sqlite Cloud"
description: ""
access_date: 2026-08-03T18:54:19.022Z
current_date: 2026-08-03T18:54:19.022Z
---

## Drizzle <> SQLite Cloud

This guide assumes familiarity with:

- Database [connection basics](connect-overview.md) with Drizzle
- **SQLite Cloud database** - [docs](https://docs.sqlitecloud.io/docs/overview)
- **SQLite Cloud driver** - [docs](https://docs.sqlitecloud.io/docs/sdk-js-introduction) & [GitHub](https://github.com/sqlitecloud/sqlitecloud-js)

According to the **[official website](https://docs.sqlitecloud.io/docs/overview)**, SQLite Clouds is a managed, distributed relational database system built on top of the SQLite database engine.

#### Step 1 - Install packages

```shell
npm i drizzle-orm@rc @sqlitecloud/drivers
npm i -D drizzle-kit@rc
```

#### Step 2 - Initialize the driver and make a query

```typescript
import { drizzle } from 'drizzle-orm/sqlite-cloud';

const db = drizzle(process.env.SQLITE_CLOUD_CONNECTION_STRING);

const result = await db.all('select 1');
```

If you need to provide your existing drivers:

```typescript
import { Database } from '@sqlitecloud/drivers';
import { drizzle } from 'drizzle-orm/sqlite-cloud';

const client = new Database(process.env.SQLITE_CLOUD_CONNECTION_STRING!);
const db = drizzle({ client });

const result = await db.all('select 1');
```
