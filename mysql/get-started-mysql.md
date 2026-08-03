---
url: https://orm.drizzle.team/docs/mysql/get-started-mysql
title: "Get Started Mysql"
description: ""
access_date: 2026-08-03T19:08:17.353Z
current_date: 2026-08-03T19:08:17.353Z
---

## Drizzle <> MySQL

To use Drizzle with a MySQL database, you should use the `mysql2` driver

According to the **[official website](https://github.com/sidorares/node-mysql2)**, `mysql2` is a MySQL client for Node.js with focus on performance.

Drizzle ORM natively supports `mysql2` with `drizzle-orm/mysql2` package.

#### Step 1 - Install packages

```shell
npm i drizzle-orm@rc mysql2
npm i -D drizzle-kit@rc
```

#### Step 2 - Initialize the driver and make a query

```typescript
import { drizzle } from "drizzle-orm/mysql2";

const db = drizzle(process.env.DATABASE_URL);

const response = await db.select().from(...)
```

If you need to provide your existing driver:

```typescript
import { drizzle } from "drizzle-orm/mysql2";
import mysql from "mysql2/promise";

const connection = await mysql.createConnection({
  host: "host",
  user: "user",
  database: "database",
  ...
});

const db = drizzle({ client: connection });
```

IMPORTANT

For the built in `migrate` function with DDL migrations we and drivers strongly encourage you to use single `client` connection.

For querying purposes feel free to use either `client` or `pool` based on your business demands.
