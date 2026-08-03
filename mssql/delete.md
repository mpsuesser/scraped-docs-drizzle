---
url: https://orm.drizzle.team/docs/mssql/delete
title: "Delete"
description: ""
access_date: 2026-08-03T19:43:09.006Z
current_date: 2026-08-03T19:43:09.006Z
---

## SQL Delete

You can delete all rows in the table:

```typescript
await db.delete(users);
```

And you can delete with filters and conditions:

```typescript
await db.delete(users).where(eq(users.name, 'Dan'));
```

### Output

You can delete a row and get it back in PostgreSQL:

```typescript
const deletedUser = await db.delete(users)
  .where(eq(users.name, 'Dan'))
  .output();

// partial return
const deletedUserId = await db.delete(users)
  .where(eq(users.name, "Dan"))
  .output({ deletedId: users.id });

// deletedUserId: { deletedId: number }[]
```
```sql
delete from [users] output DELETED.[id], DELETED.[name], DELETED.[age] where [users].[name] = 'Dan'

delete from [users] output DELETED.[id] where [users].[name] = 'Dan'
```
