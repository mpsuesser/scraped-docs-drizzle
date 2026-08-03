---
url: https://orm.drizzle.team/docs/singlestore/column-types
title: "Column Types"
description: ""
access_date: 2026-08-03T19:00:22.305Z
current_date: 2026-08-03T19:00:22.305Z
---

## SingleStore column types

We have native support for all of them, yet if that’s not enough for you, feel free to create **[custom types](custom-types.md)**.

important

All examples in this part of the documentation do not use database column name aliases, and column names are generated from TypeScript keys.

You can use database aliases in column names if you want, and you can also use the `casing` parameter to define a mapping strategy for Drizzle.

You can read more about it [here](sql-schema-declaration.md#shape-your-data-schema)

### integer

A signed integer, stored in `0`, `1`, `2`, `3`, `4`, `6`, or `8` bytes depending on the magnitude of the value.

```typescript
import { int, singlestoreTable } from "drizzle-orm/singlestore-core";

const table = singlestoreTable('table', {
    int: int()
});
```
```sql
CREATE TABLE \`table\` (
    \`int\` int
);
```

### tinyint

```typescript
import { tinyint, singlestoreTable } from "drizzle-orm/singlestore-core";

const table = singlestoreTable('table', {
    tinyint: tinyint()
});
```
```sql
CREATE TABLE \`table\` (
    \`tinyint\` tinyint
);
```

### smallint

```typescript
import { smallint, singlestoreTable } from "drizzle-orm/singlestore-core";

const table = singlestoreTable('table', {
    smallint: smallint()
});
```
```sql
CREATE TABLE \`table\` (
    \`smallint\` smallint
);
```

### mediumint

```typescript
import { mediumint, singlestoreTable } from "drizzle-orm/singlestore-core";

const table = singlestoreTable('table', {
    mediumint: mediumint()
});
```
```sql
CREATE TABLE \`table\` (
    \`mediumint\` mediumint
);
```

### bigint

```typescript
import { bigint, singlestoreTable } from "drizzle-orm/singlestore-core";

const table = singlestoreTable('table', {
    bigint: bigint({ mode: 'number' })
    bigintUnsigned: bigint({ mode: 'number', unsigned: true })
});

bigint('...', { mode: 'number' | 'bigint' });

// You can also specify unsigned option for bigint
bigint('...', { mode: 'number' | 'bigint', unsigned: true })
```
```sql
CREATE TABLE \`table\` (
    \`bigint\` bigint,
    \`bigintUnsigned\` bigint unsigned
);
```

We’ve omitted config of `M` in `bigint(M)`, since it indicates the display width of the numeric type

## \---

### real

```typescript
import { real, singlestoreTable } from "drizzle-orm/singlestore-core";

const table = singlestoreTable('table', {
    real: real()
});
```
```sql
CREATE TABLE \`table\` (
    \`real\` real
);
```

```typescript
import { real, singlestoreTable } from "drizzle-orm/singlestore-core";

const table = singlestoreTable('table', {
    realPrecision: real({ precision: 1,}),
    realPrecisionScale: real({ precision: 1, scale: 1,}),
});
```
```sql
CREATE TABLE \`table\` (
    \`realPrecision\` real(1),
    \`realPrecisionScale\` real(1, 1)
);
```

### decimal

```typescript
import { decimal, singlestoreTable } from "drizzle-orm/singlestore-core";

const table = singlestoreTable('table', {
    decimal: decimal(),
    decimalNum: decimal({ scale: 30, mode: 'number' }),
    decimalBig: decimal({ scale: 30, mode: 'bigint' }),
});
```
```sql
CREATE TABLE \`table\` (
    \`decimal\` decimal,
    \`decimalNum\` decimal(30),
    \`decimalBig\` decimal(30)
);
```

```typescript
import { decimal, singlestoreTable } from "drizzle-orm/singlestore-core";

const table = singlestoreTable('table', {
    decimalPrecision: decimal({ precision: 1,}),
    decimalPrecisionScale: decimal({ precision: 1, scale: 1,}),
});
```
```sql
CREATE TABLE \`table\` (
    \`decimalPrecision\` decimal(1),
    \`decimalPrecisionScale\` decimal(1, 1)
);
```

### double

```typescript
import { double, singlestoreTable } from "drizzle-orm/singlestore-core";

const table = singlestoreTable('table', {
    double: double('double')
});
```
```sql
CREATE TABLE \`table\` (
    \`double\` double
);
```

```typescript
import { double, singlestoreTable } from "drizzle-orm/singlestore-core";

const table = singlestoreTable('table', {
    doublePrecision: double({ precision: 1,}),
    doublePrecisionScale: double({ precision: 1, scale: 1,}),
});
```
```sql
CREATE TABLE \`table\` (
    \`doublePrecision\` double(1),
    \`doublePrecisionScale\` double(1, 1)
);
```

### float

```typescript
import { float, singlestoreTable } from "drizzle-orm/singlestore-core";

const table = singlestoreTable('table', {
    float: float()
});
```
```sql
CREATE TABLE \`table\` (
    \`float\` float
);
```

## \---

### serial

`SERIAL` is an alias for `BIGINT UNSIGNED NOT NULL AUTO_INCREMENT UNIQUE`.

```typescript
import { serial, singlestoreTable } from "drizzle-orm/singlestore-core";

const table = singlestoreTable('table', {
    serial: serial()
});
```
```sql
CREATE TABLE \`table\` (
    \`serial\` serial AUTO_INCREMENT
);
```

## \---

### binary

```typescript
import { binary, singlestoreTable } from "drizzle-orm/singlestore-core";

const table = singlestoreTable('table', {
    binary: binary()
});
```
```sql
CREATE TABLE \`table\` (
    \`binary\` binary
);
```

### varbinary

```typescript
import { varbinary, singlestoreTable } from "drizzle-orm/singlestore-core";

const table = singlestoreTable('table', {
    varbinary: varbinary({ length: 2}),
});
```
```sql
CREATE TABLE \`table\` (
    \`varbinary\` varbinary(2)
);
```

## \---

### char

```typescript
import { char, singlestoreTable } from "drizzle-orm/singlestore-core";

const table = singlestoreTable('table', {
    char: char(),
});
```
```sql
CREATE TABLE \`table\` (
    \`char\` char
);
```

### varchar

You can define `{ enum: ["value1", "value2"] }` config to infer `insert` and `select` types, it **won’t** check runtime values.

```typescript
import { varchar, singlestoreTable } from "drizzle-orm/singlestore-core";

const table = singlestoreTable('table', {
    varchar: varchar({ length: 2 }),
});

// will be inferred as text: "value1" | "value2" | null
varchar: varchar({ length: 6, enum: ["value1", "value2"] })
```
```sql
CREATE TABLE \`table\` (
    \`varchar\` varchar(2)
);
```

### text

You can define `{ enum: ["value1", "value2"] }` config to infer `insert` and `select` types, it **won’t** check runtime values.

```typescript
import { text, singlestoreTable } from "drizzle-orm/singlestore-core";

const table = singlestoreTable('table', {
    text: text(),
});

// will be inferred as text: "value1" | "value2" | null
text: text({ enum: ["value1", "value2"] });
```
```sql
CREATE TABLE \`table\` (
    \`text\` text
);
```

## \---

### boolean

```typescript
import { boolean, singlestoreTable } from "drizzle-orm/singlestore-core";

const table = singlestoreTable('table', {
    boolean: boolean(),
});
```
```sql
CREATE TABLE \`table\` (
    \`boolean\` boolean
);
```

## \---

```typescript
import { boolean, singlestoreTable } from "drizzle-orm/singlestore-core";

const table = singlestoreTable('table', {
    date: date(),
});
```
```sql
CREATE TABLE \`table\` (
    \`date\` date
);
```

### datetime

```typescript
import { datetime, singlestoreTable } from "drizzle-orm/singlestore-core";

const table = singlestoreTable('table', {
    datetime: datetime(),
});

datetime('...', { mode: 'date' | "string"}),
```
```sql
CREATE TABLE \`table\` (
    \`datetime\` datetime
);
```

### time

```typescript
import { time, singlestoreTable } from "drizzle-orm/singlestore-core";

const table = singlestoreTable('table', {
    time: time(),
});
```
```sql
CREATE TABLE \`table\` (
    \`time\` time
);
```

### year

```typescript
import { year, singlestoreTable } from "drizzle-orm/singlestore-core";

const table = singlestoreTable('table', {
    year: year(),
});
```
```sql
CREATE TABLE \`table\` (
    \`year\` year
);
```

### timestamp

```typescript
import { timestamp, singlestoreTable } from "drizzle-orm/singlestore-core";

const table = singlestoreTable('table', {
    timestamp: timestamp(),
});

timestamp('...', { mode: 'date' | "string"}),
```
```sql
CREATE TABLE \`table\` (
    \`timestamp\` timestamp
);
```

```typescript
import { timestamp, singlestoreTable } from "drizzle-orm/singlestore-core";

const table = singlestoreTable('table', {
    timestamp: timestamp().defaultNow(),
});
```
```sql
CREATE TABLE \`table\` (
    \`timestamp\` timestamp DEFAULT (now())
);
```

## \---

### json

```typescript
import { json, singlestoreTable } from "drizzle-orm/singlestore-core";

const table = singlestoreTable('table', {
    json: json(),
});
```
```sql
CREATE TABLE \`table\` (
    \`json\` json
);
```

You can specify `.$type<..>()` for json object inference, it **won’t** check runtime values. It provides compile time protection for default values, insert and select schemas.

```typescript
// will be inferred as { foo: string }
json: json().$type<{ foo: string }>();

// will be inferred as string[]
json: json().$type<string[]>();

// won't compile
json: json().$type<string[]>().default({});
```

## \---

### enum

```typescript
import { singlestoreEnum, singlestoreTable } from "drizzle-orm/singlestore-core";

const table = singlestoreTable('table', {
    popularity: singlestoreEnum(['unknown', 'known', 'popular']),
});
```
```sql
CREATE TABLE \`table\` (
    \`popularity\` enum('unknown','known','popular')
);
```

## \---

### Customizing data type

Every column builder has a `.$type()` method, which allows you to customize the data type of the column. This is useful, for example, with unknown or branded types.

```ts
type UserId = number & { __brand: 'user_id' };
type Data = {
    foo: string;
    bar: number;
};

const users = singlestoreTable('users', {
  id: int().$type<UserId>().primaryKey(),
  jsonField: json().$type<Data>(),
});
```

### Not null

`NOT NULL` constraint dictates that the associated column may not contain a `NULL` value.

```typescript
import { int, singlestoreTable } from "drizzle-orm/singlestore-core";

const table = singlestoreTable('table', {
    int: int().notNull(),
});
```
```sql
CREATE TABLE \`table\` (
    \`int\` int NOT NULL
);
```

### Default value

The `DEFAULT` clause specifies a default value to use for the column if no value is explicitly provided by the user when doing an `INSERT`. If there is no explicit `DEFAULT` clause attached to a column definition, then the default value of the column is `NULL`.

An explicit `DEFAULT` clause may specify that the default value is `NULL`, a string constant, a blob constant, a signed-number, or any constant expression enclosed in parentheses.

```typescript
import { int, singlestoreTable } from "drizzle-orm/singlestore-core";

const table = singlestoreTable('table', {
    int: int().default(3),
});
```
```sql
CREATE TABLE \`table\` (
    \`int\` int DEFAULT 3
);
```

When using `$default()` or `$defaultFn()`, which are simply different aliases for the same function, you can generate defaults at runtime and use these values in all insert queries. These functions can assist you in utilizing various implementations such as `uuid`, `cuid`, `cuid2`, and many more.

Note: This value does not affect the `drizzle-kit` behavior, it is only used at runtime in `drizzle-orm`

```ts
import { varchar, singlestoreTable } from "drizzle-orm/singlestore-core";
import { createId } from '@paralleldrive/cuid2';

const table = singlestoreTable('table', {
    id: varchar({ length: 128 }).$defaultFn(() => createId()),
});
```

When using `$onUpdate()` or `$onUpdateFn()`, which are simply different aliases for the same function, you can generate defaults at runtime and use these values in all update queries.

Adds a dynamic update value to the column. The function will be called when the row is updated, and the returned value will be used as the column value if none is provided. If no default (or $defaultFn) value is provided, the function will be called when the row is inserted as well, and the returned value will be used as the column value.

Note: This value does not affect the `drizzle-kit` behavior, it is only used at runtime in `drizzle-orm`

```ts
import { text, singlestoreTable } from "drizzle-orm/singlestore-core";

const table = singlestoreTable('table', {
    alwaysNull: text().$type<string | null>().$onUpdate(() => null),
});
```

### Primary key

```typescript
import { int, singlestoreTable } from "drizzle-orm/singlestore-core";

const table = singlestoreTable('table', {
    int: int().primaryKey(),
});
```
```sql
CREATE TABLE \`table\` (
    \`int\` int PRIMARY KEY NOT NULL
);
```

### Auto increment

```typescript
import { int, singlestoreTable } from "drizzle-orm/singlestore-core";

const table = singlestoreTable('table', {
    int: int().autoincrement(),
});
```
```sql
CREATE TABLE \`table\` (
    \`int\` int AUTO_INCREMENT
);
```
