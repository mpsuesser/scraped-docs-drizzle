---
url: https://orm.drizzle.team/docs/sqlite/transactions
title: "Transactions"
description: ""
access_date: 2026-08-03T19:00:22.305Z
current_date: 2026-08-03T19:00:22.305Z
---

## Transactions

SQL transaction is a grouping of one or more SQL statements that interact with a database. A transaction in its entirety can commit to a database as a single logical unit or rollback (become undone) as a single logical unit.

Drizzle ORM provides APIs to run SQL statements in transactions:

```ts
const db = drizzle(...)

await db.transaction(async (tx) => {
  await tx.update(accounts).set({ balance: sql\`${accounts.balance} - 100.00\` }).where(eq(users.name, 'Dan'));
  await tx.update(accounts).set({ balance: sql\`${accounts.balance} + 100.00\` }).where(eq(users.name, 'Andrew'));
});
```

Drizzle ORM supports `savepoints` with nested transactions API:

```ts
const db = drizzle(...)

await db.transaction(async (tx) => {
  await tx.update(accounts).set({ balance: sql\`${accounts.balance} - 100.00\` }).where(eq(users.name, 'Dan'));
  await tx.update(accounts).set({ balance: sql\`${accounts.balance} + 100.00\` }).where(eq(users.name, 'Andrew'));

  await tx.transaction(async (tx2) => {
    await tx2.update(users).set({ name: "Mr. Dan" }).where(eq(users.name, "Dan"));
  });
});
```

You can embed business logic to the transaction and rollback whenever needed:

```ts
const db = drizzle(...)

await db.transaction(async (tx) => {
  const [account] = await tx.select({ balance: accounts.balance }).from(accounts).where(eq(users.name, 'Dan'));
  if (account.balance < 100) {
    // This throws an exception that rollbacks the transaction.
    tx.rollback()
  }

  await tx.update(accounts).set({ balance: sql\`${accounts.balance} - 100.00\` }).where(eq(users.name, 'Dan'));
  await tx.update(accounts).set({ balance: sql\`${accounts.balance} + 100.00\` }).where(eq(users.name, 'Andrew'));
});
```

You can return values from the transaction:

```ts
const db = drizzle(...)

const newBalance: number = await db.transaction(async (tx) => {
  await tx.update(accounts).set({ balance: sql\`${accounts.balance} - 100.00\` }).where(eq(users.name, 'Dan'));
  await tx.update(accounts).set({ balance: sql\`${accounts.balance} + 100.00\` }).where(eq(users.name, 'Andrew'));

  const [account] = await tx.select({ balance: accounts.balance }).from(accounts).where(eq(users.name, 'Dan'));
  return account.balance;
});
```

You can use transactions with **[relational queries](rqb.md)**:

```ts
const db = drizzle({ schema })

await db.transaction(async (tx) => {
  await tx.query.users.findMany({
    with: {
      accounts: true
    }
  });
});
```

We provide dialect-specific transaction configuration APIs:

```ts
await db.transaction(
  async (tx) => {
    await tx.update(accounts).set({ balance: sql\`${accounts.balance} - 100.00\` }).where(eq(users.name, "Dan"));
    await tx.update(accounts).set({ balance: sql\`${accounts.balance} + 100.00\` }).where(eq(users.name, "Andrew"));
  }, {
    behavior: "deferred",
  }
);

interface SQLiteTransactionConfig {
    behavior?: 'deferred' | 'immediate' | 'exclusive';
}
```
