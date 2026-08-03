---
url: https://orm.drizzle.team/docs/sqlite/batch-api
title: "Batch Api"
description: ""
access_date: 2026-08-03T19:43:09.006Z
current_date: 2026-08-03T19:43:09.006Z
---

## Batch API

Drizzle supports running SQL statements in a batch with SQLite-compatible libSQL and Cloudflare D1 drivers.

**LibSQL Batch API explanation**: *[source](https://docs.turso.tech/sdk/ts/reference#batch-transactions)*

> With the libSQL client library, a batch is one or more SQL statements executed in order in an implicit transaction. If all statements are successful, the transaction is committed. If any statement fails, the transaction is rolled back.

**D1 Batch API explanation**: *[source](https://developers.cloudflare.com/d1/worker-api/d1-database/#batch)*

> Batching sends multiple SQL statements inside a single call to the database. This can reduce latency from network round trips to D1.

```ts
const batchResponse = await db.batch([
  db.insert(usersTable).values({ id: 1, name: 'John' }).returning({ id: usersTable.id }),
  db.update(usersTable).set({ name: 'Dan' }).where(eq(usersTable.id, 1)),
  db.query.usersTable.findMany({}),
  db.select().from(usersTable).where(eq(usersTable.id, 1)),
  db.select({ id: usersTable.id, invitedBy: usersTable.invitedBy }).from(usersTable),
]);
```

libSQL

D1

```ts
type BatchResponse = [
  { id: number }[],
  ResultSet,
  { id: number; name: string; verified: number; invitedBy: number | null }[],
  { id: number; name: string; verified: number; invitedBy: number | null }[],
  { id: number; invitedBy: number | null }[],
]
```

All possible builders that can be used inside `db.batch`:

```ts
db.all(),
db.get(),
db.values(),
db.run(),
db.execute(),
db.query.<table>.findMany(),
db.query.<table>.findFirst(),
db.select()...,
db.update()...,
db.delete()...,
db.insert()...,
```
