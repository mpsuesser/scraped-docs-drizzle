---
url: https://orm.drizzle.team/docs/cockroach/arktype
title: "Arktype"
description: ""
access_date: 2026-08-03T19:00:22.305Z
current_date: 2026-08-03T19:00:22.305Z
---

## arktype

### Install the dependencies

```shell
npm i arktype
```

### Select schema

Defines the shape of data queried from the database - can be used to validate API responses.

```ts
import { int4, cockroachTable, text } from 'drizzle-orm/cockroach-core';
import { createSelectSchema } from 'drizzle-orm/arktype';
import type { ArkErrors } from 'arktype';

const users = cockroachTable('users', {
  id: int4().primaryKey().generatedAlwaysAsIdentity(),
  name: text().notNull(),
  age: int4().notNull()
});

const userSelectSchema = createSelectSchema(users);

const rows = await db.select({ id: users.id, name: users.name }).from(users).limit(1);
const parsed: ArkErrors | { id: number; name: string; age: number } = userSelectSchema(rows[0]); // Error: \`age\` is not returned in the above query

const rows = await db.select().from(users).limit(1);
const parsed: ArkErrors | { id: number; name: string; age: number } = userSelectSchema(rows[0]); // Will parse successfully
```

Views and enums are also supported.

```ts
import { cockroachEnum, cockroachView } from 'drizzle-orm/cockroach-core';
import { gt } from 'drizzle-orm';
import { createSelectSchema } from 'drizzle-orm/arktype';
import type { ArkErrors } from 'arktype';

const roles = cockroachEnum('roles', ['admin', 'basic']);
const rolesSchema = createSelectSchema(roles);
const parsed: 'admin' | 'basic' = rolesSchema(...);

const usersView = cockroachView('users_view').as((qb) => qb.select().from(users).where(gt(users.age, 18)));
const usersViewSchema = createSelectSchema(usersView);
const parsed: ArkErrors | { id: number; name: string; age: number } = usersViewSchema(...);
```

### Insert schema

Defines the shape of data to be inserted into the database - can be used to validate API requests.

```ts
import { int4, cockroachTable, text } from 'drizzle-orm/cockroach-core';
import { createInsertSchema } from 'drizzle-orm/arktype';
import { ArkErrors } from "arktype";

const users = cockroachTable('users', {
  id: int4().primaryKey().generatedAlwaysAsIdentity(),
  name: text().notNull(),
  age: int4().notNull()
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
import { int4, cockroachTable, text } from 'drizzle-orm/cockroach-core';
import { createUpdateSchema } from 'drizzle-orm/arktype';
import { eq } from "drizzle-orm";
import { ArkErrors } from 'arktype';

const users = cockroachTable('users', {
  id: int4().primaryKey().generatedAlwaysAsIdentity(),
  name: text().notNull(),
  age: int4().notNull()
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
import { int4, jsonb, cockroachTable, text } from 'drizzle-orm/cockroach-core';
import { createSelectSchema } from 'drizzle-orm/arktype';
import { ArkErrors, type } from 'arktype';

const users = cockroachTable('users', {
  id: int4().primaryKey().generatedAlwaysAsIdentity(),
  name: text().notNull(),
  bio: text(),
  preferences: jsonb()
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

CockroachDB data type mappings follow the CockroachDB column builders. See the [CockroachDB data types](column-types.md) page for the full column reference.
