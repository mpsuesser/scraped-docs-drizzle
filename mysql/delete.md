---
url: https://orm.drizzle.team/docs/mysql/delete
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

### Limit

Use `.limit()` to add `limit` clause to the query - for example:

```typescript
await db.delete(users).where(eq(users.name, 'Dan')).limit(2);
```
```sql
delete from \`users\` where \`users\`.\`name\` = 'Dan' limit 2;
```

### Order By

Use `.orderBy()` to add `order by` clause to the query, sorting the results by the specified fields:

```typescript
import { asc, desc } from 'drizzle-orm';

await db.delete(users).where(eq(users.name, 'Dan')).orderBy(users.name);
await db.delete(users).where(eq(users.name, 'Dan')).orderBy(desc(users.name));

// order by multiple fields
await db.delete(users).where(eq(users.name, 'Dan')).orderBy(users.name, users.name2);
await db.delete(users).where(eq(users.name, 'Dan')).orderBy(asc(users.name), desc(users.name2));
```
```sql
delete from \`users\` where \`users\`.\`name\` = 'Dan' order by \`users\`.\`name\`;
delete from \`users\` where \`users\`.\`name\` = 'Dan' order by \`users\`.\`name\` desc;

delete from \`users\` where \`users\`.\`name\` = 'Dan' order by \`users\`.\`name\`, \`users\`.\`name2\`;
delete from \`users\` where \`users\`.\`name\` = 'Dan' order by \`users\`.\`name\` asc, \`users\`.\`name2\` desc;
```
