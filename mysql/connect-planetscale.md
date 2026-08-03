---
url: https://orm.drizzle.team/docs/mysql/connect-planetscale
title: "Connect Planetscale"
description: ""
access_date: 2026-08-03T18:54:19.022Z
current_date: 2026-08-03T18:54:19.022Z
---

## Drizzle <> PlanetScale MySQL

This guide assumes familiarity with:

- Database [connection basics](connect-overview.md) with Drizzle
- PlanetScale database - [website](https://planetscale.com/docs)
- PlanetScale http driver - [GitHub](https://github.com/planetscale/database-js)
- Drizzle MySQL drivers - [docs](get-started-mysql.md)

PlanetScale offers both MySQL (Vitess) and PostgreSQL databases. This page covers connecting to PlanetScale MySQL.

For PlanetScale Postgres, see the [PlanetScale Postgres connection guide](../connect-planetscale-postgres.md).

With Drizzle ORM you can access PlanetScale MySQL over http through their official **[`database-js`](https://github.com/planetscale/database-js)** driver from serverless and serverfull environments with our `drizzle-orm/planetscale-serverless` package.

You can also access PlanetScale MySQL through TCP with `mysql2` driver — **[see here.](get-started-mysql.md)**

#### Step 1 - Install packages

```shell
npm i drizzle-orm@rc @planetscale/database
npm i -D drizzle-kit@rc
```

#### Step 2 - Initialize the driver and make a query

```typescript
import { drizzle } from "drizzle-orm/planetscale-serverless";

const db = drizzle({ connection: {
  host: process.env["DATABASE_HOST"],
  username: process.env["DATABASE_USERNAME"],
  password: process.env["DATABASE_PASSWORD"],
}});

const response = await db.select().from(...)
```

If you need to provide your existing driver

```typescript
import { drizzle } from "drizzle-orm/planetscale-serverless";
import { Client } from "@planetscale/database";

const client = new Client({
  host: process.env["DATABASE_HOST"],
  username: process.env["DATABASE_USERNAME"],
  password: process.env["DATABASE_PASSWORD"],
});

const db = drizzle({ client });
```

Make sure to checkout the PlanetScale official **[MySQL courses](https://planetscale.com/courses/mysql-for-developers)**, we think they’re outstanding 🙌
