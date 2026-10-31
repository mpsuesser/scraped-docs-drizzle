---
url: https://orm.drizzle.team/docs/sqlite/relations-v1-v2
title: "Relations V1 V2"
description: ""
access_date: 2026-10-31T10:54:20.546Z
current_date: 2026-10-31T10:54:20.546Z
---

## Migrating to Relational Queries version 2

This guide migrates Relational Queries v1 from `drizzle-orm@0.x` or early v1 betas to Relational Queries v2 in `drizzle-orm@rc`. Notes about changes from earlier release candidates are marked below.

Legacy relations imports

The before examples below use the 0.x import: `import { relations } from 'drizzle-orm'`. In early v1 betas that kept both relational APIs, the legacy function was available as `import { relations } from 'drizzle-orm/_relations'`. Both are starting points for this migration. The `_relations` module still exists in rc.5, but RQB v1 support has been removed from the database drivers. For rc.5, use `import { defineRelations } from 'drizzle-orm'`.

```ts
// Before: drizzle-orm@0.x
import { relations } from 'drizzle-orm';
```
```ts
// Before: early v1 betas with the legacy RQB v1 API
import { relations } from 'drizzle-orm/_relations';
```
```ts
// After: drizzle-orm@rc (RQB v2)
import { defineRelations } from 'drizzle-orm';
```

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
import { drizzle } from "drizzle-orm/..."

const db = drizzle(process.env.DATABASE_URL, { relations })
```

##### What is different?

Schema Definition

```ts
import * as p from 'drizzle-orm/sqlite-core';

export const users = p.sqliteTable('users', {
  id: p.integer().primaryKey(),
  name: p.text(),
  invitedBy: p.integer('invited_by'),
});

export const posts = p.sqliteTable('posts', {
  id: p.integer().primaryKey(),
  content: p.text(),
  authorId: p.integer('author_id'),
});
```

**One place for all your relations**

Before: drizzle-orm@0.x (RQB v1)

```ts
import { relations } from "drizzle-orm";
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

Before: drizzle-orm@0.x (RQB v1)

```ts
import { relations } from "drizzle-orm";
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

`optional: false` removes `null` from the related value’s inferred type. It does not create a database constraint. Use it only when you are certain that the related row will always exist.

Before: drizzle-orm@0.x (RQB v1)

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

Before: drizzle-orm@0.x (RQB v1)

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

Before: drizzle-orm@0.x (RQB v1)

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

Before: drizzle-orm@0.x (RQB v1)

```ts
import { relations } from "drizzle-orm";
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

**Custom types in relational queries**

When a custom column needs different handling inside JSON results, use `forJsonSelect` to change its SQL selection and `fromJson` to decode the result.

```ts
import { customType } from 'drizzle-orm/sqlite-core';

const customBytes = customType<{
  data: Buffer;
  driverData: Buffer;
  jsonData: string;
}>({
  dataType: () => 'blob',
  fromJson: (value) => Buffer.from(value, 'hex'),
  forJsonSelect: (identifier, sql) =>
    sql\`case when ${identifier} is null then null else hex(${identifier}) end\`,
});
```

SQLite’s `hex()` returns the blob as a hex string without a prefix. The `case` expression preserves SQL `NULL`, which `hex()` alone would turn into an empty string. `fromJson` converts non-null hex strings back to a `Buffer`.

##### What is new?

**`through` for many-to-many relations**

Previously, you would need to query through a junction table and then map it out for every response

You don’t need to do it now!

Schema

```ts
import * as p from 'drizzle-orm/sqlite-core';

export const users = p.sqliteTable('users', {
  id: p.integer().primaryKey(),
  name: p.text(),
  verified: p.boolean().notNull(),
});

export const groups = p.sqliteTable('groups', {
  id: p.integer().primaryKey(),
  name: p.text(),
});

export const usersToGroups = p.sqliteTable(
  'users_to_groups',
  {
    userId: p
      .integer('user_id')
      .notNull()
      .references(() => users.id),
    groupId: p
      .integer('group_id')
      .notNull()
      .references(() => groups.id),
  },
  (t) => [p.primaryKey({ columns: [t.userId, t.groupId] })]
);
```

Before: drizzle-orm@0.x (RQB v1)

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

Before: drizzle-orm@0.x (RQB v1)

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

Before: drizzle-orm@0.x (RQB v1)

```ts
const response = db.query.users.findMany({
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
import { integer, json, sqliteTable, text, timestamp } from 'drizzle-orm/sqlite-core';

export const users = sqliteTable('users', {
  id: integer().primaryKey(),
  name: text(),
  email: text().notNull(),
  age: integer(),
  createdAt: timestamp('created_at').defaultNow(),
  lastLogin: timestamp('last_login'),
  subscriptionEnd: timestamp('subscription_end'),
  lastActivity: timestamp('last_activity'),
  preferences: text({ mode: "json" }), // JSON column for user settings/preferences
  interests: text({ mode: "json" }).$type<string[]>(), // Array column for user interests
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
        }
      ]
    }
  })
```

##### orderBy accepts objects and callbacks

The v1 callback form still works in v2. The object form is an alternative for column ordering.

0.x and rc.5: supported callback syntax

```ts
const response = db.query.users.findMany({
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

Before: drizzle-orm@0.x (RQB v1)

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

Before: drizzle-orm@0.x (RQB v1)

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

Before: drizzle-orm@0.x (RQB v1)

```ts
import * as schema from './schema'
import { drizzle } from 'drizzle-orm/...'

const db = drizzle('<url>', { schema })
```

After

```ts
// should be imported from a file in Step 2
import { relations } from './relations'
import { drizzle } from 'drizzle-orm/...'

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
select
  "d0"."id" as "id",
  "d0"."name" as "name",
  "d0"."age" as "age"
from
  "users" as "d0"
where
  "d0"."age" = 15
```

##### Handle optional filters explicitly

Starting with `1.0.0-rc.5`, an explicit `undefined` in a relational filter throws instead of silently skipping the condition. Omit the field, or use `EmptyFilter` when skipping it is intentional:

```ts
import { EmptyFilter } from 'drizzle-orm/relations';

const userId: number | undefined = undefined;

const response = await db.query.users.findMany({
  where: {
    id: userId ?? EmptyFilter,
  },
});
```

Here, an absent `userId` leaves the query unfiltered. Only skip the condition if returning all users is intended. This change concerns `db.query` filters, not conditional filters in the SQL query builder.

##### Check whether a relation exists

Use `true` to require a related row and `false` to require its absence:

```ts
// Users with at least one post
const usersWithPosts = await db.query.users.findMany({
  where: { posts: true },
});

// Users without posts
const usersWithoutPosts = await db.query.users.findMany({
  where: { posts: false },
});
```

Starting with `1.0.0-rc.5`, `where: { posts: {} }` skips the relation filter. It does not check whether a post exists. The same applies when the object contains only empty conditions.

##### Migrate orderBy statements

You can use an object to specify each column and its sort direction (`asc` or `desc`). The callback form remains supported, so an existing `orderBy: (users, { asc }) => [asc(users.id)]` does not need to change.

0.x and rc.5: supported callback syntax

```ts
const response = db.query.users.findMany({
  orderBy: (users, { asc }) => [asc(users.id)],
});
```

✅ v2

```ts
const response = db.query.users.findMany({
  orderBy: { id: "asc" },
});
```

##### Order by related rows

You can order users by the number of related posts:

```ts
const usersByPostCount = await db.query.users.findMany({
  orderBy: { posts: 'desc(count)' },
});
```

For a field on a `many` relation, specify an aggregate, such as `orderBy: { posts: { id: 'desc(max)' } }`. For a `one` relation, use the field’s direction directly, such as `orderBy: { author: { name: 'asc' } }` when querying posts.

##### Migrate many-to-many queries

Relational Queries v1 had a very complex way of managing many-to-many queries. You had to use junction tables to query through them explicitly, and then map those tables out, like this:

```ts
// Before: drizzle-orm@0.x (RQB v1)
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

Before: drizzle-orm@0.x (RQB v1)

```ts
export function migrate<
  TSchema extends Record<string, unknown>
>(
    db: BetterSQLite3Database<TSchema>,
    config: MigrationConfig,
) {
    ...
}
```

now

```ts
export function migrate<
  TSchema extends Record<string, unknown>,
  TRelations extends AnyRelations
>(
    db: BetterSQLite3Database<TSchema, TRelations>, // SQLiteBunDatabase, NodeSQLiteDatabase, DrizzleSqliteDODatabase, ...
    config: MigrationConfig,
) {
    ...
}
```

**session**

Before: drizzle-orm@0.x (RQB v1)

```ts
export class BetterSQLiteSession<
    TFullSchema extends Record<string, unknown>,
    TSchema extends TablesRelationalConfig,
> extends SQLiteSession<'sync', RunResult, TFullSchema, TSchema>
```

now

```ts
export class BetterSQLiteSession< // SQLiteBunSession, NodeSQLiteSession, ...
    TFullSchema extends Record<string, unknown>,
    TRelations extends AnyRelations,
    TSchema extends V1.TablesRelationalConfig,
> extends SQLiteSession<'sync', RunResult, TFullSchema, TRelations, TSchema>
```

**transaction**

Before: drizzle-orm@0.x (RQB v1)

```ts
export class BetterSQLiteTransaction<
    TFullSchema extends Record<string, unknown>,
    TSchema extends TablesRelationalConfig,
> extends SQLiteTransaction<'sync', RunResult, TFullSchema, TSchema>
```

now

```ts
export class BetterSQLiteTransaction< // BunSQLiteTransaction, NodeSQLiteTransaction, ...
    TFullSchema extends Record<string, unknown>,
    TRelations extends AnyRelations,
    TSchema extends V1.TablesRelationalConfig,
> extends SQLiteTransaction<'sync', RunResult, TFullSchema, TRelations, TSchema>
```

**driver**

Before: drizzle-orm@0.x (RQB v1)

```ts
export class BetterSQLite3Database<
  TSchema extends Record<string, unknown> = Record<string, never>
> extends BaseSQLiteDatabase<'sync', RunResult, TSchema>
```

now

```ts
export class BetterSQLite3Database<
    TSchema extends Record<string, unknown> = Record<string, never>,
    TRelations extends AnyRelations = EmptyRelations,
> extends BaseSQLiteDatabase<'sync', RunResult, TSchema, TRelations>
```

2. Updated `DrizzleConfig` generic with `TRelations` argument and `relations: TRelations` field

Examples

Before: drizzle-orm@0.x (RQB v1)

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
import { RelationalQueryBuilder, SQLiteRelationalQuery } from 'drizzle-orm/sqlite-core/query-builders/query';
```
