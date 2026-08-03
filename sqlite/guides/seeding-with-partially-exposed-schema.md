---
url: https://orm.drizzle.team/docs/sqlite/guides/seeding-with-partially-exposed-schema
title: "Seeding With Partially Exposed Schema"
description: ""
access_date: 2026-08-03T19:38:26.356Z
current_date: 2026-08-03T19:38:26.356Z
---

## Seeding Partially Exposed Tables with Foreign Key

PostgreSQL

MySQL

SQLite

MSSQL

Cockroach

This guide assumes familiarity with:

- Get started with [PostgreSQL](../../get-started-postgresql.md), [MySQL](../../mysql/get-started-mysql.md), [SQLite](../get-started-sqlite.md), [MSSQL](../../mssql/get-started-mssql.md) and [Cockroach](../../cockroach/get-started-cockroach.md)
- Get familiar with [Drizzle Seed](../../seed-overview.md)

## Example 1

Let’s assume you are trying to seed your database using the seeding script and schema shown below.

index.ts

schema.ts

```ts
import { bloodPressure } from './schema.ts';

async function main() {
  const db = drizzle(...);
  await seed(db, { bloodPressure });
}
main();
```

If the `bloodPressure` table has a not-null constraint on the `userId` column, running the seeding script will cause an error.

```plaintext
Error: Column 'userId' has not null constraint, 
and you didn't specify a table for foreign key on column 'userId' in 'bloodPressure' table.
```

What does it mean?

This means we can’t fill the `userId` column with Null values due to the not-null constraint on that column. Additionally, you didn’t expose the `users` table to the `seed` function schema, so we can’t generate `users.id` to populate the `userId` column with these values.

At this point, you have several options to resolve the error:

- You can remove the not-null constraint from the `userId` column;
- You can expose `users` table to `seed` function schema
```ts
await seed(db, { bloodPressure, users });
```
- You can [refine](../../guides/seeding-with-partially-exposed-schema.md#refining-the-userid-column-generator) the `userId` column generator;

## Example 2

index.ts

schema.ts

```ts
import { bloodPressure } from './schema.ts';

async function main() {
  const db = drizzle(...);
  await seed(db, { bloodPressure });
}
main();
```

By running the seeding script above you will see a warning

```plaintext
Column 'userId' in 'bloodPressure' table will be filled with Null values
because you specified neither a table for foreign key on column 'userId' 
nor a function for 'userId' column in refinements.
```

What does it mean?

This means you neither provided the `users` table to the `seed` function schema nor refined the `userId` column generator. As a result, the `userId` column will be filled with Null values.

Then you will have two choices:

- If you’re okay with filling the `userId` column with Null values, you can ignore the warning;
- Otherwise, you can [refine](../../guides/seeding-with-partially-exposed-schema.md#refining-the-userid-column-generator) the `userId` column generator.

## Refining the userId column generator

Doing so requires the `users` table to already have IDs such as 1 and 2 in the database.

index.ts

```ts
import { bloodPressure } from './schema.ts';

async function main() {
  const db = drizzle(...);
  await seed(db, { bloodPressure }).refine((funcs) => ({
    bloodPressure: {
      columns: {
        userId: funcs.valuesFromArray({ values: [1, 2] })
      }
    }
  }));
}
main();
```
