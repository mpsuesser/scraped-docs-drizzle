---
url: https://orm.drizzle.team/docs/latest-releases/drizzle-orm-v0312
title: "Drizzle Orm V0312"
description: ""
access_date: 2026-08-03T19:43:09.006Z
current_date: 2026-08-03T19:43:09.006Z
---

[v1.0](https://orm.drizzle.team/roadmap)

[

98%

](https://orm.drizzle.team/roadmap)

[

Benchmarks

](https://orm.drizzle.team/benchmarks)[

Extension

](https://driz.link/extension)[

Studio

](https://orm.drizzle.team/drizzle-studio/overview)[

Studio Package

](https://github.com/drizzle-team/drizzle-studio-npm)[

Gateway

](https://gateway.drizzle.team/)[

Drizzle Run

](https://drizzle.run/)

Our goodies!

Product by Drizzle Team

[

One Dollar Stats $1 per mo web analytics

christmas  
deal

](https://driz.link/onedollarstats)

## DrizzleORM v0.31.2 release

- 🎉 Added support for TiDB Cloud Serverless driver:
```ts
import { connect } from '@tidbcloud/serverless';
import { drizzle } from 'drizzle-orm/tidb-serverless';

const client = connect({ url: '...' });
const db = drizzle(client);
await db.select().from(...);
```
