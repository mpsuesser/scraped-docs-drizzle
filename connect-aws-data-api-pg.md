---
url: https://orm.drizzle.team/docs/connect-aws-data-api-pg
title: "Connect Aws Data Api Pg"
description: ""
access_date: 2026-08-03T19:43:09.006Z
current_date: 2026-08-03T19:43:09.006Z
---

## Drizzle <> AWS Data API Postgres

This guide assumes familiarity with:

- Database [connection basics](connect-overview.md) with Drizzle
- AWS Data API - [website](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/data-api.html)
- AWS SDK - [website](https://docs.aws.amazon.com/AWSJavaScriptSDK/v3/latest/Package/-aws-sdk-client-rds-data/)

#### Step 1 - Install packages

```shell
npm i drizzle-orm@rc @aws-sdk/client-rds-data
npm i -D drizzle-kit@rc
```

#### Step 2 - Initialize the driver and make a query

```typescript
import { drizzle } from 'drizzle-orm/aws-data-api/pg';

// These three properties are required. You can also specify
// any property from the RDSDataClient type inside the connection object.
const db = drizzle({ connection: {
  database: process.env['DATABASE']!,
  secretArn: process.env['SECRET_ARN']!,
  resourceArn: process.env['RESOURCE_ARN']!,
}});

await db.select().from(...);
```

If you need to provide your existing driver:

```typescript
import { drizzle } from 'drizzle-orm/aws-data-api/pg';
import { RDSDataClient } from '@aws-sdk/client-rds-data';

const rdsClient = new RDSDataClient({ region: 'us-east-1' });

const db = drizzle({
  client: rdsClient,
  database: process.env['DATABASE']!,
  secretArn: process.env['SECRET_ARN']!,
  resourceArn: process.env['RESOURCE_ARN']!,
});

await db.select().from(...);
```
