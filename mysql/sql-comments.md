---
url: https://orm.drizzle.team/docs/mysql/sql-comments
title: "Sql Comments"
description: ""
access_date: 2026-08-03T19:43:09.006Z
current_date: 2026-08-03T19:43:09.006Z
---

## SQL Comments

You can add custom tags to any query using the `.comment()` method. These tags are appended as [sqlcommenter](https://google.github.io/sqlcommenter) -formatted comments at the end of each query, helping you attach metadata for query tracking, debugging and database traffic control.

The `.comment()` method is available on `select`, `insert`, `update` and `delete` queries.

### String comments

You can pass a raw string as a comment:

```ts
db.select().from(users).comment("my_first_tag");
```
```sql
select \`id\`, \`name\` from \`users\` /*my_first_tag*/
```

```ts
db.select().from(users).comment("key='val'");
```
```sql
select \`id\`, \`name\` from \`users\` /*key='val'*/
```

### Object comments

You can pass an object with key-value pairs for structured tags:

```ts
db.select().from(users).comment({ priority: 'high', category: 'analytics' });
```
```sql
select \`id\`, \`name\` from \`users\` /*priority='high',category='analytics'*/
```

```ts
db.select().from(users).comment({ trace: true, route: '/api/users', version: 2 });
```
```sql
select \`id\`, \`name\` from \`users\` /*route='%2Fapi%2Fusers',trace='true',version='2'*/
```

### Usage with insert, update, delete

```ts
db.insert(users).values({ name: 'Dan' }).comment({ operation: 'seed' });
```
```sql
insert into \`users\` (\`name\`) values ('Dan') /*operation='seed'*/
```

```ts
db.update(users).set({ name: 'Dan' }).where(eq(users.id, 1)).comment({ operation: 'update' });
```
```sql
update \`users\` set \`name\` = 'Dan' where \`users\`.\`id\` = 1 /*operation='update'*/
```

```ts
db.delete(users).where(eq(users.id, 1)).comment({ operation: 'cleanup' });
```
```sql
delete from \`users\` where \`users\`.\`id\` = 1 /*operation='cleanup'*/
```

### Limitations

You can’t use `.comment()` after the statement has been prepared. Prepared statements compile the SQL query once and reuse it across executions, so the query string is fixed at preparation time and can’t be modified afterwards — including appending comments:

```ts
// can't be used
const p = db.select().from(users).prepare();
// ❌
p.comment({ key: 'val' }).execute();
```

Instead, add the comment before preparing the statement:

```ts
// ✅
db.select().from(users).comment({ key: "val" }).prepare();
```
