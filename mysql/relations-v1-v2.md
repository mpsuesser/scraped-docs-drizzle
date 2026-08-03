---
url: https://orm.drizzle.team/docs/mysql/relations-v1-v2
title: "Relations V1 V2"
description: ""
access_date: 2026-08-03T19:43:09.006Z
current_date: 2026-08-03T19:43:09.006Z
---

## Migrating to Relational Queries version 2

```shell
npm i drizzle-orm@rc
npm i drizzle-kit@rc -D
```

This guide assumes familiarity with:

- **Relations Fundamentals** - [read here](relations-schema-declaration.md)
- **drizzle-kit pull** - [read here](drizzle-kit-pull.md)

Below is the table of contents. Click an item to jump to that section:

### API changes

#### What is working differently from v1

One of the biggest updates were in **Relations Schema definition**

The first difference is that you no longer need to specify `relations` for each table separately in different objects and then pass them all to `drizzle()` along with your schema. In Relational Queries v2, you now have one dedicated place to specify all the relations for all the tables you need.

The `r` parameter in the callback provides comprehensive autocomplete functionality - including all tables from your schema and functions such as `one`, `many`, and `through` - essentially offering everything you need to specify your relations.

```ts
// relations.ts
import * as schema from "./schema"
import { defineRelations } from "drizzle-orm"

export const relations = defineRelations(schema, (r) => ({
    ...
}));
```
```ts
// index.ts
import { relations } from "./relations"
import { drizzle } from "drizzle-orm/mysql2"

const db = drizzle(process.env.DATABASE_URL, { relations })
```

##### What is different?

Schema Definition

```ts
import * as p from 'drizzle-orm/mysql-core';

export const users = p.mysqlTable('users', {
  id: p.int().primaryKey(),
  name: p.text(),
  invitedBy: p.int('invited_by'),
});

export const posts = p.mysqlTable('posts', {
  id: p.int().primaryKey(),
  content: p.text(),
  authorId: p.int('author_id'),
});
```

**One place for all your relations**

❌ v1

```ts
import { relations } from "drizzle-orm/_relations";
import { users, posts } from './schema';

export const usersRelation = relations(users, ({ one, many }) => ({
  invitee: one(users, {
    fields: [users.invitedBy],
    references: [users.id],
  }),
  posts: many(posts),
}));

export const postsRelation = relations(posts, ({ one, many }) => ({
    fields: [posts.authorId],
    references: [users.id],
  }),
}));
```

✅ v2

```ts
import { defineRelations } from "drizzle-orm";
import * as schema from "./schema";

export const relations = defineRelations(schema, (r) => ({
  users: {
    invitee: r.one.users({
      from: r.users.invitedBy,
      to: r.users.id,
    }),
    posts: r.many.posts(),
  },
  posts: {
      from: r.posts.authorId,
      to: r.users.id,
    }),
  },
}));
```

You can still separate it into different `parts`, and you can make the parts any size you want

```ts
import { defineRelations, defineRelationsPart } from 'drizzle-orm';
import * as schema from "./schema";

export const relations = defineRelations(schema, (r) => ({
  users: {
    invitee: r.one.users({
      from: r.users.invitedBy,
      to: r.users.id,
    }),
    posts: r.many.posts(),
  }
}));

export const part = defineRelationsPart(schema, (r) => ({
  posts: {
      from: r.posts.authorId,
      to: r.users.id,
    }),
  }
}));
```

and then you can provide it to the db instance

```ts
const db = drizzle(process.env.DB_URL, { relations: { ...relations, ...part } })
```

**Define `many` without `one`**

In v1, if you wanted only the `many` side of a relationship, you had to specify the `one` side on the other end, which made for a poor developer experience.

In v2, you can simply use the `many` side without any additional steps

❌ v1

```ts
import { relations } from "drizzle-orm/_relations";
import { users, posts } from './schema';

export const usersRelation = relations(users, ({ one, many }) => ({
  posts: many(posts),
}));

export const postsRelation = relations(posts, ({ one, many }) => ({
    fields: [posts.authorId],
    references: [users.id],
  }),
}));
```

✅ v2

```ts
import { defineRelations } from "drizzle-orm";
import * as schema from "./schema";

export const relations = defineRelations(schema, (r) => ({
  users: {
    posts: r.many.posts({
      from: r.users.id,
      to: r.posts.authorId,
    }),
  },
}));
```

**New `optional` option**

`optional: false` at the type level makes the `author` key in the `posts` object required. This should be used when you are certain that this specific entity will always exist.

❌ v1

Was not supported in v1

✅ v2

```ts
import { defineRelations } from "drizzle-orm";
import * as schema from "./schema";

export const relations = defineRelations(schema, (r) => ({
  users: {
    posts: r.one.posts({
      from: r.users.id,
      to: r.posts.authorId,
      optional: false,
    }),
  },
}));
```

**No modes in `drizzle()`**

We found a way to use the same strategy for all MySQL dialects, so there’s no need to specify them

❌ v1

```ts
import * as schema from './schema'

const db = drizzle(process.env.DATABASE_URL, { mode: "planetscale", schema });
// or
const db = drizzle(process.env.DATABASE_URL, { mode: "default", schema });
```

✅ v2

```ts
import { relations } from './relations'

const db = drizzle(process.env.DATABASE_URL, { relations });
```

**`from` and `to` upgrades**

We’ve renamed `fields` to `from` and `references` to `to`, and we made both accept either a single value or an array

❌ v1

```ts
...
author: one(users, {
  fields: [posts.authorId],
  references: [users.id],
}),
...
```

✅ v2

```ts
... 
author: r.one.users({
  from: r.posts.authorId,
  to: r.users.id,
}),
...
```
```ts
... 
author: r.one.users({
  from: [r.posts.authorId],
  to: [r.users.id],
}),
...
```

**`relationName` -> `alias`**

❌ v1

```ts
import { relations } from "drizzle-orm/_relations";
import { users, posts } from './schema';

export const postsRelation = relations(posts, ({ one }) => ({
    fields: [posts.authorId],
    references: [users.id],
      relationName: "author_post",
  }),
}));
```

✅ v2

```ts
import { defineRelations } from "drizzle-orm";
import * as schema from "./schema";

export const relations = defineRelations(schema, (r) => ({
  posts: {
      from: r.posts.authorId,
      to: r.users.id,
      alias: "author_post",
    }),
  },
}));
```

**`custom types` new functions**

There are a few new function were added to custom types, so you can control how data is mapped on Relational Queries v2:

fromJson

Optional mapping function, that is used for transforming data returned by transformed to JSON in database data to desired format For example, when querying bigint column via [RQB](../rqb.md) or JSON functions, the result field will be returned as it’s string representation, as opposed to bigint from regular query To handle that, we need a separate function to handle such field’s mapping:

```ts
fromJson(value: string): bigint {
    return BigInt(value);
},
```

It’ll cause the returned data to change from:

```ts
{
    customField: "5044565289845416380";
}
```

to:

```ts
{
    customField: 5044565289845416380n;
}
```

forJsonSelect

Optional selection modifier function, that is used for modifying selection of column inside JSON functions Additional mapping that could be required for such scenarios can be handled using fromJson function Used by [relational queries](../rqb.md)

For example, when using bigint we need to cast field to text to preserve data integrity

```ts
forJsonSelect(identifier: SQL, sql: SQLGenerator, arrayDimensions?: number): SQL {
    return sql\`${identifier}::text\`
},
```

This will change query from:

```sql
SELECT
    row_to_json("t".*)
    FROM
    (
        SELECT
        "table"."custom_bigint" AS "bigint"
        FROM
        "table"
    ) AS "t"
```

to:

```sql
SELECT
    row_to_json("t".*)
    FROM
    (
        SELECT
        "table"."custom_bigint"::text AS "bigint"
        FROM
        "table"
    ) AS "t"
```

Returned by query object will change from:

```ts
{
    bigint: 5044565289845416000; // Partial data loss due to direct conversion to JSON format
}
```

to:

```ts
{
    bigint: "5044565289845416380"; // Data is preserved due to conversion of field to text before JSON-ification
}
```

✅ v2

```ts
const customBytes = customType<{
     data: Buffer;
     driverData: Buffer;
     jsonData: string;
 }>({
     dataType: () => 'bytea',
     fromJson: (value) => {
         return Buffer.from(value.slice(2, value.length), 'hex');
     },
     forJsonSelect: (identifier, sql, arrayDimensions) =>
         sql\`${identifier}::text${sql.raw('[]'.repeat(arrayDimensions ?? 0))}\`,
 });
```

##### What is new?

**`through` for many-to-many relations**

Previously, you would need to query through a junction table and then map it out for every response

You don’t need to do it now!

Schema

```ts
import * as p from 'drizzle-orm/mysql-core';

export const users = p.mysqlTable('users', {
  id: p.int().primaryKey(),
  name: p.text(),
  verified: p.boolean().notNull(),
});

export const groups = p.mysqlTable('groups', {
  id: p.int().primaryKey(),
  name: p.text(),
});

export const usersToGroups = p.mysqlTable(
  'users_to_groups',
  {
    userId: p
      .int('user_id')
      .notNull()
      .references(() => users.id),
    groupId: p
      .int('group_id')
      .notNull()
      .references(() => groups.id),
  },
  (t) => [p.primaryKey({ columns: [t.userId, t.groupId] })],
);
```

❌ v1

```ts
export const usersRelations = relations(users, ({ many }) => ({
  usersToGroups: many(usersToGroups),
}));

export const groupsRelations = relations(groups, ({ many }) => ({
  usersToGroups: many(usersToGroups),
}));

export const usersToGroupsRelations = relations(usersToGroups, ({ one }) => ({
  group: one(groups, {
    fields: [usersToGroups.groupId],
    references: [groups.id],
  }),
  user: one(users, {
    fields: [usersToGroups.userId],
    references: [users.id],
  }),
}));
```
```ts
// Query example
const response = await db.query.users.findMany({
  with: {
    usersToGroups: {
      columns: {},
      with: {
        group: true,
      },
    },
  },
});
```

✅ v2

```ts
import * as schema from './schema';
import { defineRelations } from 'drizzle-orm';

export const relations = defineRelations(schema, (r) => ({
  users: {
    groups: r.many.groups({
      from: r.users.id.through(r.usersToGroups.userId),
      to: r.groups.id.through(r.usersToGroups.groupId),
    }),
  },
  groups: {
    participants: r.many.users(),
  },
}));
```
```ts
// Query example
const response = await db.query.users.findMany({
  with: {
    groups: true,
  },
});
```

**Predefined filters**

❌ v1

Was not supported in v1

✅ v2

```ts
import * as schema from './schema';
import { defineRelations } from 'drizzle-orm';

export const relations = defineRelations(schema,
  (r) => ({
    groups: {
      verifiedUsers: r.many.users({
        from: r.groups.id.through(r.usersToGroups.groupId),
        to: r.users.id.through(r.usersToGroups.userId),
        where: {
          verified: true,
        },
      }),
    },
  })
);
```
```ts
// Query example: get groups with all verified users
const response = await db.query.groups.findMany({
  with: {
    verifiedUsers: true,
  },
});
```

##### where is now object

❌ v1

```ts
const response = db._query.users.findMany({
  where: (users, { eq }) => eq(users.id, 1),
});
```

✅ v2

```ts
const response = db.query.users.findMany({
  where: {
    id: 1,
  },
});
```

For a complete API Reference please check our [Select Filters docs](rqb.md#select-filters)

Complex filter example using RAW

```ts
// schema.ts
import { int, json, mysqlTable, text, timestamp } from 'drizzle-orm/mysql-core';

export const users = mysqlTable('users', {
  id: int().primaryKey(),
  name: text(),
  email: text().notNull(),
  age: int(),
  createdAt: timestamp('created_at').defaultNow(),
  lastLogin: timestamp('last_login'),
  subscriptionEnd: timestamp('subscription_end'),
  lastActivity: timestamp('last_activity'),
  preferences: json(), // JSON column for user settings/preferences
  interests: json().$type<string[]>(), // Array column for user interests
});
```
```ts
const response = await db.query.users.findMany({
    where: {
      AND: [
        {
          OR: [{ RAW: (table) => sql\`${table.name} LIKE 'john%'\` }, { name: { like: "jane%" } }],
        },
        { 
            RAW: (table) => sql\`${table.age} BETWEEN 25 AND 35\` 
        },
      ],
    },
  }),
```

##### orderBy is now object

❌ v1

```ts
const response = db._query.users.findMany({
  orderBy: (users, { asc }) => [asc(users.id)],
});
```

✅ v2

```ts
const response = db.query.users.findMany({
  orderBy: { id: "asc" },
});
```

##### Filtering by relations

❌ v1

Was not supported in v1

✅ v2

Example: Get all users whose ID>10 and who have at least one post with content starting with “M”

```ts
const usersWithPosts = await db.query.usersTable.findMany({
  where: {
    id: {
      gt: 10
    },
    posts: {
      content: {
        like: 'M%'
      }
    }
  },
});
```

##### Using offset on related objects

❌ v1

Was not supported in v1

✅ v2

```ts
await db.query.posts.findMany({
    limit: 5,
    offset: 2, // correct ✅
    with: {
        comments: {
            offset: 3, // correct ✅
            limit: 3,
        },
    },
});
```

#### How to migrate relations schema definition from v1 to v2

##### Option 1: Using drizzle-kit pull

In new version `drizzle-kit pull` supports pulling `relations.ts` file in a new syntax:

##### Step 1

```shell
npx drizzle-kit pull
```

#### Step 2

Transfer generated relations code from `drizzle/relations.ts` to the file you are using to specify your relations

```plaintext
├ 📂 drizzle
│ ├ 📂 meta
│ ├ 📜 migration.sql
│ ├ 📜 relations.ts ────────┐
│ └ 📜 schema.ts            |
├ 📂 src                    │ 
│ ├ 📂 db                   │
│ │ ├ 📜 relations.ts <─────┘
│ │ └ 📜 schema.ts 
│ └ 📜 index.ts         
└ …
```

`drizzle/relations.ts` includes an import of all tables from `drizzle/schema.ts`, which looks like this:

```ts
import * as schema from './schema'
```

You may need to change this import to a file where ALL your schema tables are located.

If there are multiple schema files, you can do the following:

```ts
import * as schema1 from './schema1'
import * as schema2 from './schema2'
...
```

#### Step 3

Change drizzle database instance creation and provide `relations` object instead of `schema`

Before

```ts
import * as schema from './schema'
import { drizzle } from 'drizzle-orm/mysql2'

const db = drizzle('<url>', { schema })
```

After

```ts
// should be imported from a file in Step 2
import { relations } from './relations'
import { drizzle } from 'drizzle-orm/mysql2'

const db = drizzle('<url>', { relations })
```

If you had MySQL dialect, you can remove `mode` from `drizzle()` as long as it’s not needed in version 2

##### Option 2: Manual migration

If you want to migrate manually, you can check our [Drizzle Relations section](relations.md) for the complete API reference and examples of one-to-one, one-to-many, and many-to-many relations.

#### How to migrate queries from v1 to v2

##### Migrate where statements

You can check our [Select Filters docs](rqb.md#select-filters) to see examples and a complete API reference.

With the new syntax, you can use `AND`, `OR`, `NOT`, and `RAW`, plus all the filtering operators that were previously available in Relations v1.

**Examples**

simple eq

using AND

using OR

using NOT

complex example using RAW

```ts
const response = await db.query.users.findMany({
  where: {
    age: 15,
  },
});
```
```sql
SELECT
    \`d0\`.\`id\` AS \`id\`,
    \`d0\`.\`name\` AS \`name\`,
    \`d0\`.\`age\` AS \`age\`
FROM
    \`users\` AS \`d0\`
WHERE
    \`d0\`.\`age\` = 15;
```

##### Migrate orderBy statements

Order by was simplified to a single object, where you specify the column and the sort direction (`asc` or `desc`)

❌ v1

```ts
const response = db._query.users.findMany({
  orderBy: (users, { asc }) => [asc(users.id)],
});
```

✅ v2

```ts
const response = db.query.users.findMany({
  orderBy: { id: "asc" },
});
```

##### Migrate many-to-many queries

Relational Queries v1 had a very complex way of managing many-to-many queries. You had to use junction tables to query through them explicitly, and then map those tables out, like this:

```ts
const response = await db.query.users.findMany({
  with: {
    usersToGroups: {
      columns: {},
      with: {
        group: true,
      },
    },
  },
});
```

After upgrading to Relational Queries v2, your many-to-many relation will look like this:

```ts
import * as schema from './schema';
import { defineRelations } from 'drizzle-orm';

export const relations = defineRelations(schema, (r) => ({
  users: {
    groups: r.many.groups({
      from: r.users.id.through(r.usersToGroups.userId),
      to: r.groups.id.through(r.usersToGroups.groupId),
    }),
  },
  groups: {
    participants: r.many.users(),
  },
}));
```

And when you migrate your query, it will become this:

```ts
// Query example
const response = await db.query.users.findMany({
  with: {
    groups: true,
  },
});
```

### Internal changes

1. Every `drizzle` database, `session`, `migrator` and `transaction` instance updated with new generic arguments for RQB v2 queries

Examples

**migrator**

before

```ts
export async function migrate<
  TSchema extends Record<string, unknown>
>(
  db: MySql2Database<TSchema>,
  config: MigrationConfig,
) {
  ...
}
```

now

```ts
export async function migrate<TRelations extends AnyRelations>(
    db: MySql2Database<TRelations>,
    config: MigrationConfig,
) {
  ...
}
```

**session**

before

```ts
export class MySql2Session<
    TFullSchema extends Record<string, unknown>,
    TSchema extends V1.TablesRelationalConfig,
> extends MySqlSession<MySqlQueryResultHKT, MySql2PreparedQueryHKT, TFullSchema, TSchema>
```

now

```ts
export class MySql2Session<
    TRelations extends AnyRelations,
> extends MySqlSession<MySqlQueryResultHKT, TRelations>
```

**transaction**

before

```ts
export class MySql2Transaction<
    TFullSchema extends Record<string, unknown>,
    TSchema extends V1.TablesRelationalConfig,
> extends MySqlTransaction<MySql2QueryResultHKT, MySql2PreparedQueryHKT, TFullSchema, TSchema>
```

now

```ts
export class MySql2Transaction<
    TRelations extends AnyRelations,
> extends MySqlTransaction<MySql2QueryResultHKT, TRelations>
```

**driver**

before

```ts
export class MySql2Database<
    TSchema extends Record<string, unknown> = Record<string, never>,
> extends MySqlDatabase<MySql2QueryResultHKT, MySql2PreparedQueryHKT, TSchema>
```

now

```ts
export class MySql2Database<
    TRelations extends AnyRelations = EmptyRelations,
> extends MySqlDatabase<MySql2QueryResultHKT, TRelations>
```

2. Updated `DrizzleConfig` generic with `TRelations` argument and `relations: TRelations` field

Examples

before

```ts
export interface DrizzleConfig<
  TSchema extends Record<string, unknown> = Record<string, never>
> {
  logger?: boolean | Logger;
  schema?: TSchema;
  casing?: Casing;
}
```

now

```ts
export interface DrizzleConfig<
    TRelationConfigs extends AnyRelations = EmptyRelations,
> {
    logger?: boolean | Logger | undefined;
    relations?: TRelationConfigs | undefined;
    cache?: Cache | undefined;
    jit?: boolean | undefined;
}
```

3. The following entities have been removed from `drizzle-orm` and `drizzle-orm/relations`. The original imports now include new types used by Relational Queries v2, so make sure to update your imports if you intend to use the older types:

A list of all removed entities

- `Relations`
- `TableRelationsKeysOnly`
- `ExtractTableRelationsFromSchema`
- `ExtractRelationsFromTableExtraConfigSchema`
- `getOperators`
- `FindTableByDBName`
- `RelationalSchemaConfig`
- `RelationConfig`
- `extractTablesRelationalConfig`
- `relations`
- `createOne`
- `createMany`
- `NormalizedRelation`
- `normalizeRelation`
- `createTableRelationsHelpers`
- `TableRelationsHelpers`

4. In the same manner, `${dialect}-core/query-builders/query` files were updated with RQB v2’s alternatives

Examples

```ts
import { RelationalQueryBuilder, MySqlRelationalQuery } from 'drizzle-orm/mysql-core/query-builders/query';
```
