---
url: https://orm.drizzle.team/docs/mssql/zod
title: "Zod"
description: ""
access_date: 2026-08-03T18:54:19.022Z
current_date: 2026-08-03T18:54:19.022Z
---

## zod

### Install the dependencies

```shell
npm i drizzle-orm@rc zod
```

### Select schema

Defines the shape of data queried from the database - can be used to validate API responses.

```ts
import { int, mssqlTable, text } from 'drizzle-orm/mssql-core';
import { createSelectSchema } from 'drizzle-orm/zod';

const users = mssqlTable('users', {
  id: int().primaryKey().identity(),
  name: text().notNull(),
  age: int().notNull()
});

const userSelectSchema = createSelectSchema(users);

const rows = await db.select({ id: users.id, name: users.name }).top(1).from(users);
const parsed: { id: number; name: string; age: number } = userSelectSchema.parse(rows[0]); // Error: \`age\` is not returned in the above query

const rows = await db.select().top(1).from(users);
const parsed: { id: number; name: string; age: number } = userSelectSchema.parse(rows[0]); // Will parse successfully
```

Views are also supported.

```ts
import { createSelectSchema } from 'drizzle-orm/zod';
import { mssqlView } from 'drizzle-orm/mssql-core';
 
const usersView = mssqlView('users_view').as((qb) => qb.select().from(users).where(gt(users.age, 18)));
const usersViewSchema = createSelectSchema(usersView);
const parsed: { id: number; name: string; age: number } = usersViewSchema.parse(...);
```

### Insert schema

Defines the shape of data to be inserted into the database - can be used to validate API requests.

```ts
import { int, mssqlTable, text } from 'drizzle-orm/mssql-core';
import { createInsertSchema } from 'drizzle-orm/zod';

const users = mssqlTable('users', {
  id: int().primaryKey().identity(),
  name: text().notNull(),
  age: int().notNull()
});

const userInsertSchema = createInsertSchema(users);

const user = { name: 'John' };
const parsed: { name: string, age: number } = userInsertSchema.parse(user); // Error: \`age\` is not defined

const user = { name: 'Jane', age: 30 };
const parsed: { name: string, age: number } = userInsertSchema.parse(user); // Will parse successfully
await db.insert(users).values(parsed);
```

### Update schema

Defines the shape of data to be updated in the database - can be used to validate API requests.

```ts
import { int, mssqlTable, text } from 'drizzle-orm/mssql-core';
import { createUpdateSchema } from 'drizzle-orm/zod';
import { eq } from 'drizzle-orm';

const users = mssqlTable('users', {
  id: int().primaryKey().identity(),
  name: text().notNull(),
  age: int().notNull()
});

const userUpdateSchema = createUpdateSchema(users);

const user = { age: 35 };
const parsed: { name?: string | undefined, age?: number | undefined } = userUpdateSchema.parse(user); // Will parse successfully
await db.update(users).set(parsed).where(eq(users.name, 'Jane'));
```

### Refinements

Each create schema function accepts an additional optional parameter that you can used to extend, modify or completely overwite a field’s schema. Defining a callback function will extend or modify while providing a Zod schema will overwrite it.

```ts
import { int, mssqlTable, text } from 'drizzle-orm/mssql-core';
import { createSelectSchema } from 'drizzle-orm/zod';
import { z } from 'zod/v4';

const users = mssqlTable('users', {
  id: int().primaryKey(),
  name: text().notNull(),
  bio: text(),
  preferences: text()
});

const userSelectSchema = createSelectSchema(users, {
  name: (schema) => schema.max(20), // Extends schema
  bio: (schema) => schema.max(1000), // Extends schema before becoming nullable/optional
  preferences: z.object({ theme: z.string() }) // Overwrites the field, including its nullability
});

const parsed: {
  id: number;
  name: string,
  bio: string | null;
  preferences: {
    theme: string;
  };
} = userSelectSchema.parse(...);
```

### Factory functions

For more advanced use cases, you can use the `createSchemaFactory` function.

**Use case: Using an extended Zod instance**

```ts
import { int, mssqlTable, text } from 'drizzle-orm/mssql-core';
import { createSchemaFactory } from 'drizzle-orm/zod';
import { z } from '@hono/zod-openapi'; // Extended Zod instance

const users = mssqlTable('users', {
  id: int().primaryKey().identity(),
  name: text().notNull(),
  age: int().notNull()
});

const { createInsertSchema } = createSchemaFactory({ zodInstance: z });

const userInsertSchema = createInsertSchema(users, {
  // We can now use the extended instance
  name: (schema) => schema.openapi({ example: 'John' })
});
```

**Use case: Type coercion**

```ts
import { mssqlTable, datetime2 } from 'drizzle-orm/mssql-core';
import { createSchemaFactory } from 'drizzle-orm/zod';
import { z } from 'zod/v4';

const users = mssqlTable('users', {
  ...,
  createdAt: datetime2().notNull()
});

const { createInsertSchema } = createSchemaFactory({
  // This configuration will only coerce dates. Set \`coerce\` to \`true\` to coerce all data types or specify others
  coerce: {
    date: true
  }
});

const userInsertSchema = createInsertSchema(users);
// The above is the same as this:
const userInsertSchema = z.object({
  ...,
  createdAt: z.coerce.date()
});
```

### Data type reference

MSSQL data type mappings follow the MSSQL column builders. See the [MSSQL data types](column-types.md) page for the full column reference.
