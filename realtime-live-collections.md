---
url: https://orm.drizzle.team/docs/realtime-live-collections
title: "Realtime Live Collections"
description: ""
access_date: 2026-11-10T15:42:53.848Z
current_date: 2026-11-10T15:42:53.848Z
---

## Live collections and CDC event streams

This guide assumes familiarity with:

- [Drizzle Pulse overview](realtime-overview.md)
- [Queries](realtime-queries.md)
- A started runtime. See [Server setup](realtime-server-setup.md).

For each query that you pass to `createRuntime()`, the runtime gives you two ways to get its data:

1. `runtime.client.<queryName>()` creates a [live collection](#live-collections): the current rows of the query, kept up to date in memory.
2. `runtime.events.<queryName>()` subscribes to a [CDC event stream](#cdc-event-streams): each row change of the query, one event at a time.

The examples on this page use the `allOrders` and `ordersByStatus` queries from [Server setup](realtime-server-setup.md#create-and-start-the-runtime).

## Live collections

A live collection is like the result of a Drizzle `select` that stays up to date by itself. The runtime reads the rows that match the query once, keeps them in memory, and updates them after each commit that changes them. Use it when your code needs the current rows, for example to render a list or to answer requests from memory.

### Create a live collection

Create a live collection with `runtime.client.<queryName>()`. If the query has arguments, pass them as the only parameter. The call returns a promise that resolves when the live collection has loaded its first rows. The promise rejects if the runtime is not started.

```ts
const orders = await runtime.client.ordersByStatus({ status: 'accepted' });
const everyOrder = await runtime.client.allOrders();
```

Each call creates a new, independent live collection. Create a live collection once, keep it for as long as you need it, and then dispose it.

### Read rows

`list()` returns the current rows synchronously. The rows are sorted by primary key, in the direction that the query’s `.order()` sets. Call it whenever you need the rows, for example in a request handler.

```ts
const rows = orders.list();
```

### Change notifications

`onChange()` calls your listener after each change to the rows. The listener gets three fields:

1. `events`: the changes that the runtime applied. Each change has `op` set to `'insert'`, `'update'`, or `'delete'`, as described in [Events](#events).
2. `state`: the rows after the change. It is the same as `list()`.
3. `lsn`: the log sequence number of the commit.
```ts
orders.onChange(({ events, state, lsn }) => {
  console.log(\`${events.length} changes, ${state.length} rows at ${lsn}\`);
});
```

`onChange()` returns a function that removes the listener.

IMPORTANT

The runtime calls listeners synchronously and does not wait for a promise that a listener returns. If a listener is `async`, catch its errors inside the listener. A rejected promise that nobody catches is an unhandled rejection, and Node.js stops the process on it by default. This applies to all listeners on this page, including the callbacks of CDC event streams.

### Resync

After a connection loss, the live collection loads its rows again. The runtime does not know which rows changed during the connection loss, so `onChange()` does not fire for those changes. Instead, the live collection calls the listeners that you register with `onResync()`. The listener gets the new `state` and the `lsn` from which the live collection continues.

```ts
orders.onResync(({ state }) => {
  // Send the full list again, for example to connected clients.
  broadcast(state);
});
```

IMPORTANT

If your code uses `state` from `onChange()`, for example to render a list, register the same handling in `onResync()` too. Otherwise your code keeps the rows from before the connection loss until the next change.

`onResync()` also returns a function that removes the listener.

### Errors

`onError()` calls your listener when the live collection fails after you create it. This happens in two cases:

1. The live collection cannot load its rows again after a reconnect.
2. Replication [fails permanently](realtime-server-setup.md#fatal-errors).

`onError()` also returns a function that removes the listener.

```ts
orders.onError((error) => {
  console.error('Live collection failed', error);
});
```

### Dispose

`dispose()` stops the live collection. After that, the runtime calls none of its listeners. You can call it more than once.

```ts
orders.dispose();
```

## CDC event streams

A CDC event stream delivers the row changes for one query with its arguments, one by one. Use it when you want to react to changes, for example to send a notification or to update another system. The stream keeps no rows and has no first load. It reports only changes that happen after you subscribe.
