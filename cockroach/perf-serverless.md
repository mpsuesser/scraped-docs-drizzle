---
url: https://orm.drizzle.team/docs/cockroach/perf-serverless
title: "Perf Serverless"
description: ""
access_date: 2026-08-03T19:43:09.006Z
current_date: 2026-08-03T19:43:09.006Z
---

## Drizzle Serverless performance

You can get immense benefits with `serverless functions` like AWS Lambda or Vercel Server Functions (they’re AWS Lambda based), since they can live up to 15mins and reuse both database connections and prepared statements.

On the other, hand `edge functions` tend to clean up straight after they’re invoked which leads to little to no performance benefits.

To reuse your database connection and prepared statements you just have to declare them outside of handler scope:

```ts
const databaseConnection = ...;
const db = drizzle({ client: databaseConnection });
const prepared = db.select().from(...).prepare();

// AWS handler
export const handler = async (event: APIGatewayProxyEvent) => {
  return prepared.execute();
}
```
