---
url: https://orm.drizzle.team/docs/mssql/arktype
title: "Arktype"
description: ""
access_date: 2026-08-03T18:54:19.022Z
current_date: 2026-08-03T18:54:19.022Z
---

## arktype

### Install the dependencies

```shell
npm i drizzle-orm@rc arktype
```

### Select schema

Defines the shape of data queried from the database - can be used to validate API responses.

```ts
import { int, mssqlTable, text } from 'drizzle-orm/mssql-core';
import { createSelectSchema } from 'drizzle-orm/arktype';
import type { ArkErrors } from 'arktype';

const users = mssqlTable('users', {
  id: int().primaryKey().identity(),
  name: text().notNull(),
  age: int().notNull()
});

const userSelectSchema = createSelectSchema(users);

const rows = await db.select({ id: users.id, name: users.name }).top(1).from(users);
const parsed: ArkErrors | { id: number; name: string; age: number } = userSelectSchema(rows[0]); // Error: \`age\` is not returned in the above query

const rows = await db.select().top(1).from(users);
const parsed: ArkErrors | { id: number; name: string; age: number } = userSelectSchema(rows[0]); // Will parse successfully
```

Views and enums are also supported.

```ts
import { pgEnum, pgView } from 'drizzle-orm/mssql-core';
import { gt } from 'drizzle-orm';
import { createSelectSchema } from 'drizzle-orm/arktype';
import { ArkErrors } from "arktype";

const usersView = mssqlView('users_view').as((qb) => qb.select().from(users).where(gt(users.age, 18)));
const usersViewSchema = createSelectSchema(usersView);
const parsed: ArkErrors | { id: number; name: string; age: number } = usersViewSchema(...);
```

### Insert schema

Defines the shape of data to be inserted into the database - can be used to validate API requests.

```ts
import { int, mssqlTable, text } from 'drizzle-orm/mssql-core';
import { createInsertSchema } from 'drizzle-orm/arktype';
import { ArkErrors } from 'arktype';

const users = mssqlTable('users', {
  id: int().primaryKey().identity(),
  name: text().notNull(),
  age: int().notNull()
});

const userInsertSchema = createInsertSchema(users);

const user = { name: 'John' };
const parsed: ArkErrors | { name: string, age: number } = userInsertSchema(user); // Error: \`age\` is not defined

const user = { name: 'Jane', age: 30 };
const parsed: ArkErrors | { name: string, age: number } = userInsertSchema(user); // Will parse successfully

if (parsed instanceof ArkErrors) {
  console.error(parsed.summary);
  process.exit(1);
}
await db.insert(users).values(parsed);
```

### Update schema

Defines the shape of data to be updated in the database - can be used to validate API requests.

```ts
import { int, mssqlTable, text } from 'drizzle-orm/mssql-core';
import { createUpdateSchema } from 'drizzle-orm/arktype';
import { parse } from 'arktype';
import { eq } from 'drizzle-orm';
import { ArkErrors } from 'arktype';

const users = mssqlTable('users', {
  id: int().primaryKey().identity(),
  name: text().notNull(),
  age: int().notNull()
});

const userUpdateSchema = createUpdateSchema(users);

const user = { age: 35 };
const parsed: ArkErrors | { name?: string | undefined, age?: number | undefined } = userUpdateSchema(user); // Will parse successfully

if (parsed instanceof ArkErrors) {
  console.error(parsed.summary);
  process.exit(1);
}

await db.update(users).set(parsed).where(eq(users.name, 'Jane'));
```

### Refinements

Each create schema function accepts an additional optional parameter that you can used to extend, modify or completely overwite a field’s schema. Defining a callback function will extend or modify while providing a arktype schema will overwrite it.

```ts
import { int, mssqlTable, text } from 'drizzle-orm/mssql-core';
import { createSelectSchema } from 'drizzle-orm/arktype';
import { ArkErrors, type } from 'arktype';

const users = mssqlTable('users', {
  id: int().primaryKey().identity(),
  name: text().notNull(),
  bio: text(),
  preferences: text()
});

const userSelectSchema = createSelectSchema(users, {
  name: (schema) => schema.atMostLength(20), // Extends schema
  bio: () => type.string.atMostLength(2000), // Extends schema before becoming nullable/optional
  preferences: type({ theme: 'string' }) // Overwrites the field, including its nullability
});

const parsed: ArkErrors | {
  id: number;
  name: string,
  bio: string | null;
  preferences: {
    theme: string;
  };
} = userSelectSchema(...);
```

### Data type reference

MSSQL data type mappings follow the MSSQL column builders. See the [MSSQL data types](column-types.md) page for the full column reference.
