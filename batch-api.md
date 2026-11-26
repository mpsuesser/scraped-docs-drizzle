---
url: https://orm.drizzle.team/docs/batch-api
title: "Batch Api"
description: ""
access_date: 2026-11-26T10:39:33.329Z
current_date: 2026-11-26T10:39:33.329Z
---

## Batch API

These methods are available on the Drizzle Postgres driver (`drizzle-orm/postgres`).

| Method | Use it to | On failure |
| --- | --- | --- |
| [`batch`](#batch) | Commit together | Rolls back the batch |
| [`pipeline`](#pipeline) | Send queries on one connection | Other queries can commit |
| [`parallel`](#parallel) | Run queries in parallel | Other queries can commit |
| [`bulkInsert`](#bulkinsert) | Insert many rows | Rolls back by default |
| [`bulkUpdate`](#bulkupdate) | Update rows by key | Rolls back by default |

## Batch

`db.batch()` runs all statements in one transaction. If the insert fails on a unique constraint on `stockReceipts.id`, PostgreSQL rolls back the stock update.

```ts
const sku = 'usb-c-cable';
const quantity = 20;

const response = await db.batch([
  db.update(products)
    .set({ stock: sql\`${products.stock} + ${quantity}\` })
    .where(eq(products.sku, sku))
    .returning({ stock: products.stock }),
  db.insert(stockReceipts).values({ id: 'receipt-1042', sku, quantity }),
]);

console.log(response[0]); // [{ stock: 20 }] if the product started at zero
```

The driver sends the batch in one round trip and returns results in input order.

### Batch options

**Isolation**

Use `isolation` to control which changes from other transactions each query can see.

At PostgreSQL’s default isolation level, `read committed`, the count can include a product inserted after the list was read. With `repeatable read`, both queries read from the same snapshot. [PostgreSQL transaction isolation](https://www.postgresql.org/docs/current/transaction-iso.html)

```ts
const [page, total] = await db.batch([
  db.select().from(products).orderBy(products.sku).limit(25),
  db.$count(products),
], { isolation: 'repeatable read', readOnly: true });
```

Set `readOnly: true` to run a read-only transaction. In a read-only `serializable` transaction, `deferrable: true` lets PostgreSQL wait for a safe snapshot.

Use `snapshot` to import a snapshot exported by another transaction that is still open. This requires `repeatable read` or `serializable`. [PostgreSQL transaction options](https://www.postgresql.org/docs/current/sql-set-transaction.html)

**Metrics**

Set `metrics` to measure the batch. `response.metrics.total` includes time spent waiting for a connection, and `roundTrips` counts network round trips. Find each query’s timings in `response.metrics.queries`, in input order.

```ts
const response = await db.batch([
  db.$count(products),
  db.$count(stockReceipts),
], { metrics: 'us' });

console.log(response.metrics.total, response.metrics.roundTrips);
```

Use `metrics: true` for fractional milliseconds, `'ms'` for whole milliseconds, or `'us'` for whole microseconds.

## Pipeline

`db.pipeline()` sends queries back-to-back on one connection. Each statement runs in its own transaction.

```ts
const [usb, hdmi] = await db.pipeline([
  db.update(products)
    .set({ stock: 19 })
    .where(eq(products.sku, 'usb-c-cable'))
    .returning({ sku: products.sku }),
  db.update(products)
    .set({ stock: 8 })
    .where(eq(products.sku, 'hdmi-cable'))
    .returning({ sku: products.sku }),
]);

if (usb.status === 'rejected') console.error(usb.reason);
if (hdmi.status === 'rejected') console.error(hdmi.reason);
```

Each query returns a settled result in input order. If the HDMI update fails, `hdmi.reason` contains the error and a successful USB update stays committed. If a SKU is missing, its update succeeds and `value` is an empty array.

## Parallel

`db.parallel()` runs independent queries concurrently across the pool. Like `pipeline`, it returns one settled result per query in input order.

```ts
const [page, total] = await db.parallel([
  db.select().from(products).orderBy(products.sku).limit(25),
  db.$count(products),
]);

if (page.status === 'rejected') throw page.reason;

const catalog = {
  products: page.value,
  total: total.status === 'fulfilled' ? total.value : null,
};
```

When the list and count need the same snapshot, use a [batch with `repeatable read`](#batch-options).

## bulkInsert

`db.bulkInsert()` inserts rows into one table. Omitted values use Drizzle’s runtime defaults when configured, then PostgreSQL column defaults.

```ts
const inserted = await db.bulkInsert(products, [
  { sku: 'usb-c-hub', name: 'USB-C Hub' },
  { sku: 'usb-c-adapter', name: 'USB-C Adapter' },
]);

console.log(inserted.rowCount); // 2
```

## bulkUpdate

`db.bulkUpdate()` matches input rows to existing rows using `by`. Pass a unique key column or an array of columns for a composite key. Each input row needs the full key and the same update columns. Drizzle can supply values for columns with `$onUpdateFn`.

```ts
const updated = await db.bulkUpdate(products, [
  { sku: 'usb-c-hub', name: 'USB-C Hub', stock: 40 },
  { sku: 'usb-c-adapter', name: 'USB-C Adapter', stock: 75 },
  { sku: 'usb-c-charger', name: 'USB-C Charger', stock: 12 },
], { by: products.sku });

console.log(updated.rowCount); // 2 when the charger SKU is missing
```

Unmatched keys are skipped. `bulkUpdate` sends values as arrays and turns them into rows with PostgreSQL’s `unnest`. Use `db.update()` for column types that this method cannot send as arrays.

### Bulk writes and transactions

Both bulk methods split the input into chunks and run them in one transaction by default. Set `chunk` to control how many rows go into each chunk. With `atomic: false`, each chunk commits independently. If a later chunk fails, earlier committed chunks stay in the database.

Use `rowCount` to read the total affected rows after a successful call. To get the written rows back, use `.returning()` on the regular insert or update builder.

### Cancel a bulk write

With a pooled connection from `drizzle-orm/postgres`, you can pass your own `AbortSignal` to either bulk method through the `signal` option.

In this example, if the insert is still running after five seconds, `AbortSignal.timeout(5_000)` requests cancellation. This can help limit how long the import holds locks.

```ts
const inserted = await db.bulkInsert(products, [
  { sku: 'usb-c-hub', name: 'USB-C Hub' },
  { sku: 'usb-c-adapter', name: 'USB-C Adapter' },
], { signal: AbortSignal.timeout(5_000) });
```

For a stock update that the caller can cancel, use an `AbortController`.

```ts
const controller = new AbortController();

const pending = db.bulkUpdate(products, [
  { sku: 'usb-c-hub', name: 'USB-C Hub', stock: 40 },
  { sku: 'usb-c-adapter', name: 'USB-C Adapter', stock: 75 },
], {
  by: products.sku,
  signal: controller.signal,
});

// Call controller.abort() from your cancel handler.
const updated = await pending;
```

If cancellation interrupts the write, the call rejects. By default, PostgreSQL rolls back the whole bulk write. With `atomic: false`, chunks that already committed stay in the database.

## Prepared queries

`batch`, `pipeline`, and `parallel` accept prepared queries. The bulk methods take rows. In this example, call `.prepare()` once and `.bind()` for each SKU.

```ts
const productBySku = db.select().from(products)
  .where(eq(products.sku, sql.placeholder('sku')))
  .prepare('product_by_sku');

const [usb, hdmi] = await db.batch([
  productBySku.bind({ sku: 'usb-c-cable' }),
  productBySku.bind({ sku: 'hdmi-cable' }),
]);
```

## HTTP connections

- `drizzle-orm/postgres/http` supports `batch`, `pipeline`, and `parallel`. A pipeline sends its queries in one HTTP request. Parallel queries use a separate request per query.
- `drizzle-orm/postgres/neon-http` supports `batch` and `parallel`. Use `batch` to run the queries in a transaction.
- Use `drizzle-orm/postgres` for prepared queries, batch metrics, and the bulk methods shown above.

## Upgrading to v1

If you use `drizzle-orm/postgres/http/neon`, change the import to `drizzle-orm/postgres/neon-http`.
