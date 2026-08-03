---
url: https://orm.drizzle.team/docs/get-started-gel
title: "Get Started Gel"
description: ""
access_date: 2026-08-03T19:38:26.356Z
current_date: 2026-08-03T19:38:26.356Z
---

## Drizzle <> Gel

This guide assumes familiarity with:

- Database [connection basics](connect-overview.md) with Drizzle
- gel-js [basics](https://github.com/geldata/gel-js)

Drizzle has native support for Gel connections with the `gel-js` client.

#### Step 1 - Install packages

```shell
npm i drizzle-orm@rc gel
npm i -D drizzle-kit@rc
```

#### Step 2 - Initialize the driver and make a query

```typescript
// Make sure to install the 'gel' package 
import { drizzle } from 'drizzle-orm/gel';

const db = drizzle(process.env.DATABASE_URL);
 
const result = await db.execute('select 1');
```

If you need to provide your existing driver:

```typescript
// Make sure to install the 'gel' package 
import { drizzle } from "drizzle-orm/gel";
import { createClient } from "gel";

const gelClient = createClient();
const db = drizzle({ client: gelClient });

const result = await db.execute('select 1');
```
