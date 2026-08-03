---
url: https://orm.drizzle.team/docs/mysql/operators
title: "Operators"
description: ""
access_date: 2026-08-03T19:00:22.305Z
current_date: 2026-08-03T19:00:22.305Z
---

## Filter and conditional operators

We natively support all dialect specific filter and conditional operators.

You can import all filter & conditional from `drizzle-orm`:

```typescript
import { eq, ne, gt, gte, ... } from "drizzle-orm";
```

### eq

Value equal to `n`

```typescript
import { eq } from "drizzle-orm";

db.select().from(table).where(eq(table.column, 5));
```
```sql
SELECT * FROM \`table\` WHERE \`table\`.\`column\` = 5
```

```typescript
import { eq } from "drizzle-orm";

db.select().from(table).where(eq(table.column1, table.column2));
```
```sql
SELECT * FROM \`table\` WHERE \`table\`.\`column1\` = \`table\`.\`column2\`
```

### ne

Value is not equal to `n`

```typescript
import { ne } from "drizzle-orm";

db.select().from(table).where(ne(table.column, 5));
```
```sql
SELECT * FROM \`table\` WHERE \`table\`.\`column\` <> 5
```

```typescript
import { ne } from "drizzle-orm";

db.select().from(table).where(ne(table.column1, table.column2));
```
```sql
SELECT * FROM \`table\` WHERE \`table\`.\`column1\` <> \`table\`.\`column2\`
```

## \---

### gt

Value is greater than `n`

```typescript
import { gt } from "drizzle-orm";

db.select().from(table).where(gt(table.column, 5));
```
```sql
SELECT * FROM \`table\` WHERE \`table\`.\`column\` > 5
```

```typescript
import { gt } from "drizzle-orm";

db.select().from(table).where(gt(table.column1, table.column2));
```
```sql
SELECT * FROM \`table\` WHERE \`table\`.\`column1\` > \`table\`.\`column2\`
```

### gte

Value is greater than or equal to `n`

```typescript
import { gte } from "drizzle-orm";

db.select().from(table).where(gte(table.column, 5));
```
```sql
SELECT * FROM \`table\` WHERE \`table\`.\`column\` >= 5
```

```typescript
import { gte } from "drizzle-orm";

db.select().from(table).where(gte(table.column1, table.column2));
```
```sql
SELECT * FROM \`table\` WHERE \`table\`.\`column1\` >= \`table\`.\`column2\`
```

### lt

Value is less than `n`

```typescript
import { lt } from "drizzle-orm";

db.select().from(table).where(lt(table.column, 5));
```
```sql
SELECT * FROM \`table\` WHERE \`table\`.\`column\` < 5
```

```typescript
import { lt } from "drizzle-orm";

db.select().from(table).where(lt(table.column1, table.column2));
```
```sql
SELECT * FROM \`table\` WHERE \`table\`.\`column1\` < \`table\`.\`column2\`
```

### lte

Value is less than or equal to `n`.

```typescript
import { lte } from "drizzle-orm";

db.select().from(table).where(lte(table.column, 5));
```
```sql
SELECT * FROM \`table\` WHERE \`table\`.\`column\` <= 5
```

```typescript
import { lte } from "drizzle-orm";

db.select().from(table).where(lte(table.column1, table.column2));
```
```sql
SELECT * FROM \`table\` WHERE \`table\`.\`column1\` <= \`table\`.\`column2\`
```

## \---

### exists

Value exists

```typescript
import { exists } from "drizzle-orm";

const query = db.select().from(table2)
db.select().from(table).where(exists(query));
```
```sql
SELECT * FROM \`table\` WHERE EXISTS (SELECT * FROM \`table2\`)
```

### notExists

```typescript
import { notExists } from "drizzle-orm";

const query = db.select().from(table2)
db.select().from(table).where(notExists(query));
```
```sql
SELECT * FROM \`table\` WHERE NOT EXISTS (SELECT * FROM \`table2\`)
```

### isNull

Value is `null`

```typescript
import { isNull } from "drizzle-orm";

db.select().from(table).where(isNull(table.column));
```
```sql
SELECT * FROM \`table\` WHERE \`table\`.\`column\` IS NULL
```

### isNotNull

Value is not `null`

```typescript
import { isNotNull } from "drizzle-orm";

db.select().from(table).where(isNotNull(table.column));
```
```sql
SELECT * FROM \`table\` WHERE \`table\`.\`column\` IS NOT NULL
```

## \---

### inArray

Value is in array of values

```typescript
import { inArray } from "drizzle-orm";

db.select().from(table).where(inArray(table.column, [1, 2, 3, 4]));
```
```sql
SELECT * FROM \`table\` WHERE \`table\`.\`column\` IN (1, 2, 3, 4)
```

```typescript
import { inArray } from "drizzle-orm";

const query = db.select({ data: table2.column }).from(table2);
db.select().from(table).where(inArray(table.column, query));
```
```sql
SELECT * FROM \`table\` WHERE \`table\`.\`column\` IN (SELECT \`table2\`.\`column\` FROM \`table2\`)
```

### notInArray

Value is not in array of values

```typescript
import { notInArray } from "drizzle-orm";

db.select().from(table).where(notInArray(table.column, [1, 2, 3, 4]));
```
```sql
SELECT * FROM \`table\` WHERE \`table\`.\`column\` NOT IN (1, 2, 3, 4)
```

```typescript
import { notInArray } from "drizzle-orm";

const query = db.select({ data: table2.column }).from(table2);
db.select().from(table).where(notInArray(table.column, query));
```
```sql
SELECT * FROM \`table\` WHERE \`table\`.\`column\` NOT IN (SELECT \`table2\`.\`column\` FROM \`table2\`)
```

## \---

### between

Value is between two values

```typescript
import { between } from "drizzle-orm";

db.select().from(table).where(between(table.column, 2, 7));
```
```sql
SELECT * FROM \`table\` WHERE \`table\`.\`column\` BETWEEN 2 AND 7
```

### notBetween

Value is not between two value

```typescript
import { notBetween } from "drizzle-orm";

db.select().from(table).where(notBetween(table.column, 2, 7));
```
```sql
SELECT * FROM \`table\` WHERE \`table\`.\`column\` NOT BETWEEN 2 AND 7
```

## \---

### like

Value is like other value, case sensitive

```typescript
import { like } from "drizzle-orm";

db.select().from(table).where(like(table.column, "%llo wor%"));
```
```sql
SELECT * FROM \`table\` WHERE \`table\`.\`column\` LIKE '%llo wor%'
```

### notLike

Value is not like some other value, case insensitive

```typescript
import { notLike } from "drizzle-orm";

db.select().from(table).where(notLike(table.column, "%llo wor%"));
```
```sql
SELECT * FROM \`table\` WHERE \`table\`.\`column\` NOT LIKE '%llo wor%'
```

## \---

### not

All conditions must return `false`.

```typescript
import { eq, not } from "drizzle-orm";

db.select().from(table).where(not(eq(table.column, 5)));
```
```sql
SELECT * FROM \`table\` WHERE NOT (\`table\`.\`column\` = 5)
```

### and

All conditions must return `true`.

```typescript
import { gt, lt, and } from "drizzle-orm";

db.select().from(table).where(and(gt(table.column, 5), lt(table.column, 7)));
```
```sql
SELECT * FROM \`table\` WHERE (\`table\`.\`column\` > 5 AND \`table\`.\`column\` < 7)
```

### or

One or more conditions must return `true`.

```typescript
import { gt, lt, or } from "drizzle-orm";

db.select().from(table).where(or(gt(table.column, 5), lt(table.column, 7)));
```
```sql
SELECT * FROM \`table\` WHERE (\`table\`.\`column\` > 5 OR \`table\`.\`column\` < 7)
```
