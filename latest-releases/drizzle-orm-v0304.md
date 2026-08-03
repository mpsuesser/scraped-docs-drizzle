---
url: https://orm.drizzle.team/docs/latest-releases/drizzle-orm-v0304
title: "Drizzle Orm V0304"
description: ""
access_date: 2026-08-03T19:43:09.006Z
current_date: 2026-08-03T19:43:09.006Z
---

## DrizzleORM v0.30.4 release

## New Features

### 🎉 xata-http driver support

According their **[official website](https://xata.io/)**, Xata is a Postgres data platform with a focus on reliability, scalability, and developer experience. The Xata Postgres service is currently in beta, please see the [Xata docs](https://xata.io/docs/postgres) on how to enable it in your account.

Drizzle ORM natively supports both the `xata` driver with `drizzle-orm/xata` package and the **`postgres`** or **`pg`** drivers for accessing a Xata Postgres database.

The following example use the Xata generated client, which you obtain by running the [xata init](https://xata.io/docs/getting-started/installation) CLI command.

```shell
npm i drizzle-orm@rc @xata.io/client
npm i -D drizzle-kit@rc
```

```ts
import { drizzle } from 'drizzle-orm/xata-http';
import { getXataClient } from './xata'; // Generated client

const xata = getXataClient();
const db = drizzle(xata);

const result = await db.select().from(...);
```

You can also connect to Xata using `pg` or `postgres.js` drivers

To get started with Xata and Drizzle follow the [documentation](../get-started-postgresql.md#xata).
