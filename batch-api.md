---
url: https://orm.drizzle.team/docs/batch-api
title: "Batch Api"
description: ""
access_date: 2026-08-03T19:08:17.353Z
current_date: 2026-08-03T19:08:17.353Z
---

## Batch API

Drizzle supports running SQL statements in a batch with the Neon HTTP driver for PostgreSQL.

```ts
const batchResponse = await db.batch([
  db.insert(usersTable).values({ id: 1, name: 'John' }).returning({ id: usersTable.id }),
  db.update(usersTable).set({ name: 'Dan' }).where(eq(usersTable.id, 1)),
  db.query.usersTable.findMany({}),
  db.select().from(usersTable).where(eq(usersTable.id, 1)),
  db.select({ id: usersTable.id, invitedBy: usersTable.invitedBy }).from(usersTable),
]);
```
```ts
type BatchResponse = [
  { id: number }[],
  NeonHttpQueryResult,
  { id: number; name: string; verified: number; invitedBy: number | null }[],
  { id: number; name: string; verified: number; invitedBy: number | null }[],
  { id: number; invitedBy: number | null }[],
]
```

All possible builders that can be used inside `db.batch`:

```ts
db.query.<table>.findMany(),
db.query.<table>.findFirst(),
db.select()...,
db.update()...,
db.delete()...,
db.insert()...,
```
