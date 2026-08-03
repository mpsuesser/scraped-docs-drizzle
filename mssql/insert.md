---
url: https://orm.drizzle.team/docs/mssql/insert
title: "Insert"
description: ""
access_date: 2026-08-03T18:54:19.022Z
current_date: 2026-08-03T18:54:19.022Z
---

## SQL Insert

All the values provided to `.values()` are parameterized automatically. For example, this query:

```ts
await db.insert(users).values({ name: 'Andrew' });
```

will be translated to:

```sql
insert into [users] ([name]) values (@par0); -- params: ['Andrew']
```

Drizzle ORM provides you the most SQL-like way to insert rows into the database tables.

Inserting data with Drizzle is extremely straightforward and sql-like. See for yourself:

```typescript
await db.insert(users).values({ name: 'Andrew' });
```
```sql
insert into [users] ([name], [age]) values ('Andrew', default)
```

If you need insert type for a particular table you can use `typeof usersTable.$inferInsert` syntax.

```typescript
type NewUser = typeof users.$inferInsert;

const insertUser = async (user: NewUser) => {
  return db.insert(users).values(user);
}

const newUser: NewUser = { name: "Alef" };
await insertUser(newUser);
```

## Insert with output

MSSQL supports the `OUTPUT` clause for returning inserted rows. Drizzle exposes it through the `.output()` method.

```typescript
const result = await db
  .insert(usersTable)
  .output({ id: usersTable.id })
  .values([{ name: 'John' }, { name: 'John1' }]);
//    ^? { id: number }[]
```

You can also return all inserted columns by calling `.output()` without arguments.

```ts
const result = await db.insert(usersTable).output().values({ name: 'John' });
//    ^? typeof usersTable.$inferSelect[]
```

## Insert multiple rows

```typescript
await db.insert(users).values([{ name: 'Andrew' }, { name: 'Dan' }]);
```

## Upserts and conflicts

MSSQL does not support PostgreSQL-style `ON CONFLICT` or MySQL-style `ON DUPLICATE KEY UPDATE` clauses. For upsert workflows, use a transaction with explicit `select` / `update` / `insert` logic or execute a dialect-specific `MERGE` statement with the `sql` operator.

## Insert into … select

Currently not supported
