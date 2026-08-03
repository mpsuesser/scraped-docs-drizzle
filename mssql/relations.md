---
url: https://orm.drizzle.team/docs/mssql/relations
title: "Relations"
description: ""
access_date: 2026-08-03T18:54:19.022Z
current_date: 2026-08-03T18:54:19.022Z
---

## Drizzle soft relations

The sole purpose of Drizzle relations is to let you query your relational data in the most simple and consise way:

Relational queries

Select with joins

```ts
import * as schema from './schema';
import { drizzle } from 'drizzle-orm/node-mssql';

const db = drizzle({ client, schema });

const result = db._query.users.findMany({
  with: {
    posts: true,
  },
});
```
```ts
[{
  id: 10,
  name: "Dan",
  posts: [
    {
      id: 1,
      content: "SQL is awesome",
      authorId: 10,
    },
    {
      id: 2,
      content: "But check relational queries",
      authorId: 10,
    }
  ]
}]
```

### One-to-one

Drizzle ORM provides you an API to define `one-to-one` relations between tables with the `relations` operator.

An example of a `one-to-one` relation between users and users, where a user can invite another (this example uses a self reference):

```typescript
import { mssqlTable, ntext, int } from 'drizzle-orm/mssql-core';
import { relations } from 'drizzle-orm/_relations';

export const users = mssqlTable('users', {
    id: int().identity().primaryKey(),
    name: ntext(),
    invitedBy: int('invited_by'),
});

export const usersRelations = relations(users, ({ one }) => ({
    invitee: one(users, {
        fields: [users.invitedBy],
        references: [users.id],
    }),
}));
```

Another example would be a user having a profile information stored in separate table. In this case, because the foreign key is stored in the “profile\_info” table, the user relation have neither fields or references. This tells Typescript that `user.profileInfo` is nullable:

```typescript
import { mssqlTable, ntext, int } from 'drizzle-orm/mssql-core';
import { relations } from 'drizzle-orm/_relations';

export const users = mssqlTable('users', {
    id: int().identity().primaryKey(),
    name: ntext(),
});

export const usersRelations = relations(users, ({ one }) => ({
    profileInfo: one(profileInfo),
}));

export const profileInfo = mssqlTable('profile_info', {
    id: int().identity().primaryKey(),
    userId: int('user_id').references(() => users.id),
    metadata: ntext(),
});

export const profileInfoRelations = relations(profileInfo, ({ one }) => ({
    user: one(users, { fields: [profileInfo.userId], references: [users.id] }),
}));

const user = await queryUserWithProfileInfo();
//____^? type { id: number, profileInfo: string | null  }
```

### One-to-many

Drizzle ORM provides you an API to define `one-to-many` relations between tables with `relations` operator.

Example of `one-to-many` relation between users and posts they’ve written:

```typescript
import { mssqlTable, ntext, int } from 'drizzle-orm/mssql-core';
import { relations } from 'drizzle-orm/_relations';

export const users = mssqlTable('users', {
    id: int().identity().primaryKey(),
    name: ntext(),
});

export const usersRelations = relations(users, ({ many }) => ({
    posts: many(posts),
}));

export const posts = mssqlTable('posts', {
    id: int().identity().primaryKey(),
    content: ntext(),
    authorId: int('author_id'),
});

export const postsRelations = relations(posts, ({ one }) => ({
        fields: [posts.authorId],
        references: [users.id],
    }),
}));
```

Now lets add comments to the posts:

```typescript
...

export const posts = mssqlTable('posts', {
    id: int().identity().primaryKey(),
    content: ntext(),
    authorId: int('author_id'),
});

export const postsRelations = relations(posts, ({ one, many }) => ({
        fields: [posts.authorId],
        references: [users.id],
    }),
    comments: many(comments)
}));

export const comments = mssqlTable('comments', {
    id: int().identity().primaryKey(),
    text: ntext(),
    authorId: int('author_id'),
    postId: int('post_id'),
});

export const commentsRelations = relations(comments, ({ one }) => ({
    post: one(posts, {
        fields: [comments.postId],
        references: [posts.id],
    }),
}));
```

### Many-to-many

Drizzle ORM provides you an API to define `many-to-many` relations between tables through so called `junction` or `join` tables, they have to be explicitly defined and store associations between related tables.

Example of `many-to-many` relation between users and groups:

```typescript
import { relations } from 'drizzle-orm/_relations';
import { int, mssqlTable, primaryKey, ntext } from 'drizzle-orm/mssql-core';

export const users = mssqlTable('users', {
  id: int().identity().primaryKey(),
  name: ntext(),
});

export const usersRelations = relations(users, ({ many }) => ({
  usersToGroups: many(usersToGroups),
}));

export const groups = mssqlTable('groups', {
  id: int().identity().primaryKey(),
  name: ntext(),
});

export const groupsRelations = relations(groups, ({ many }) => ({
  usersToGroups: many(usersToGroups),
}));

export const usersToGroups = mssqlTable(
  'users_to_groups',
  {
    userId: int('user_id')
      .notNull()
      .references(() => users.id),
    groupId: int('group_id')
      .notNull()
      .references(() => groups.id),
  },
  (t) => [
        primaryKey({ columns: [t.userId, t.groupId] })
    ],
);

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

### Foreign keys

You might’ve noticed that `relations` look similar to foreign keys — they even have a `references` property. So what’s the difference?

While foreign keys serve a similar purpose, defining relations between tables, they work on a different level compared to `relations`.

Foreign keys are a database level constraint, they are checked on every `insert` / `update` / `delete` operation and throw an error if a constraint is violated. On the other hand, `relations` are a higher level abstraction, they are used to define relations between tables on the application level only. They do not affect the database schema in any way and do not create foreign keys implicitly.

What this means is `relations` and foreign keys can be used together, but they are not dependent on each other. You can define `relations` without using foreign keys (and vice versa), which allows them to be used with databases that do not support foreign keys.

The following two examples will work exactly the same in terms of querying the data using Drizzle relational queries.

schema1.ts

schema2.ts

```ts
export const users = mssqlTable('users', {
  id: int().identity().primaryKey(),
  name: ntext(),
});

export const profileInfo = mssqlTable('profile_info', {
  id: int().identity().primaryKey(),
  userId: int('user_id'),
  metadata: nvarchar({ mode: 'json' }),
});

export const profileInfoRelations = relations(profileInfo, ({ one }) => ({
  user: one(users, {
    fields: [profileInfo.userId],
    references: [users.id],
  }),
}));
```

### Foreign key actions

For more information check the [MSSQL foreign keys docs](https://learn.microsoft.com/en-us/sql/relational-databases/tables/create-foreign-key-relationships?view=sql-server-ver17)

You can specify actions that should occur when the referenced data in the parent table is modified. These actions are known as “foreign key actions.” MSSQL provides several options for these actions.

On Delete/ Update Actions

- `CASCADE`: When a row in the parent table is deleted, all corresponding rows in the child table will also be deleted. This ensures that no orphaned rows exist in the child table.
- `NO ACTION`: This is the default action. It prevents the deletion of a row in the parent table if there are related rows in the child table. The DELETE operation in the parent table will fail.
- `SET DEFAULT`: If a row in the parent table is deleted, the foreign key column in the child table will be set to its default value if it has one. If it doesn’t have a default value, the DELETE operation will fail.
- `SET NULL`: When a row in the parent table is deleted, the foreign key column in the child table will be set to NULL. This action assumes that the foreign key column in the child table allows NULL values.

> Analogous to ON DELETE there is also ON UPDATE which is invoked when a referenced column is changed (updated). The possible actions are the same, except that column lists cannot be specified for SET NULL and SET DEFAULT. In this case, CASCADE means that the updated values of the referenced column(s) should be copied into the referencing row(s). in drizzle you can add foreign key action using `references()` second argument.

type of the actions

```typescript
export type UpdateDeleteAction = 'cascade' | 'no action' | 'set null' | 'set default';

// second argument of references interface
actions?: {
        onUpdate?: UpdateDeleteAction;
        onDelete?: UpdateDeleteAction;
    } | undefined
```

In the following example, adding `onDelete: 'cascade'` to the author field on the `posts` schema means that deleting the `user` will also delete all related Post records.

```typescript
import { mssqlTable, ntext, int } from 'drizzle-orm/mssql-core';

export const users = mssqlTable('users', {
    id: int().identity().primaryKey(),
    name: ntext(),
});

export const posts = mssqlTable('posts', {
    id: int().identity().primaryKey(),
    name: ntext(),
});
```

For constraints specified with the `foreignKey` operator, foreign key actions are defined with the syntax:

```typescript
import { foreignKey, mssqlTable, ntext, int } from 'drizzle-orm/mssql-core';

export const users = mssqlTable('users', {
    id: int().identity().primaryKey(),
    name: ntext(),
});

export const posts = mssqlTable('posts', {
    id: int().identity().primaryKey(),
    name: ntext(),
}, (table) => [
    foreignKey({
        name: "author_fk",
        columns: [table.author],
        foreignColumns: [users.id],
    })
        .onDelete('cascade')
        .onUpdate('cascade')
]);
```

### Disambiguating relations

Drizzle also provides the `relationName` option as a way to disambiguate relations when you define multiple of them between the same two tables. For example, if you define a `posts` table that has the `author` and `reviewer` relations.

```ts
import { mssqlTable, ntext, int } from 'drizzle-orm/mssql-core';
import { relations } from 'drizzle-orm/_relations';
 
export const users = mssqlTable('users', {
    id: int().identity().primaryKey(),
    name: ntext(),
});
 
export const usersRelations = relations(users, ({ many }) => ({
    reviewer: many(posts, { relationName: 'reviewer' }),
}));
 
export const posts = mssqlTable('posts', {
    id: int().identity().primaryKey(),
    content: ntext(),
    authorId: int('author_id'),
    reviewerId: int('reviewer_id'),
});
 
export const postsRelations = relations(posts, ({ one }) => ({
        fields: [posts.authorId],
        references: [users.id],
        relationName: 'author',
    }),
    reviewer: one(users, {
        fields: [posts.reviewerId],
        references: [users.id],
        relationName: 'reviewer',
    }),
}));
```
