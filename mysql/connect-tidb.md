---
url: https://orm.drizzle.team/docs/mysql/connect-tidb
title: "Connect Tidb"
description: ""
access_date: 2026-08-03T19:43:09.006Z
current_date: 2026-08-03T19:43:09.006Z
---

## Drizzle <> TiDB Serverless

This guide assumes familiarity with:

- Database [connection basics](connect-overview.md) with Drizzle
- TiDB database - [website](https://docs.pingcap.com/)
- TiDB HTTP Driver - [website](https://docs.pingcap.com/developer/serverless-driver)
- Drizzle MySQL drivers - [docs](get-started-mysql.md)

According to the **[official website](https://www.pingcap.com/tidb-serverless/)**, TiDB Serverless is a fully-managed, autonomous DBaaS with split-second cluster provisioning and consumption-based pricing.

TiDB Serverless is compatible with MySQL, so you can use [MySQL connection guide](get-started-mysql.md) to connect to it.

TiDB Serverless provides an [HTTP driver](https://docs.pingcap.com/developer/serverless-driver/) for edge environments. It is natively supported by Drizzle ORM via `drizzle-orm/tidb-serverless` package.

#### Step 1 - Install packages

```shell
npm i drizzle-orm@rc @tidbcloud/serverless
npm i -D drizzle-kit@rc
```

#### Step 2 - Initialize the driver and make a query

```typescript
import { drizzle } from 'drizzle-orm/tidb-serverless';

const db = drizzle({ connection: { url: process.env.TIDB_URL }});

const response = await db.select().from(...)
```

If you need to provide your existing driver:

```typescript
import { connect } from '@tidbcloud/serverless';
import { drizzle } from 'drizzle-orm/tidb-serverless';

const client = connect({ url: process.env.TIDB_URL });
const db = drizzle({ client });
```
