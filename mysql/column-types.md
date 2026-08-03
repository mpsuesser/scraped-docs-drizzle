---
url: https://orm.drizzle.team/docs/mysql/column-types
title: "Column Types"
description: ""
access_date: 2026-08-03T19:38:26.356Z
current_date: 2026-08-03T19:38:26.356Z
---

## MySQL column types

We have native support for all of them, yet if that’s not enough for you, feel free to create **[custom types](custom-types.md)**.

important

All examples in this part of the documentation do not use database column name aliases, and column names are generated from TypeScript keys.

You can use database aliases in column names if you want, and you can also use the `casing API` to define a mapping strategy for Drizzle.

You can read more about it [here](sql-schema-declaration.md#shape-your-data-schema)

important

All integer types (`int`, `tinyint`, `smallint`, `mediumint`, `bigint`) are `signed` by default, allowing both negative and positive values. Adding `{ unsigned: true }` restricts the column to non-negative values only but doubles the upper range.

### integer

A `4` -byte integer with a range of `-2,147,483,648` to `2,147,483,647` signed, or `0` to `4,294,967,295` unsigned

For more info please refer to the official MySQL **[docs.](https://dev.mysql.com/doc/refman/8.0/en/integer-types.html)**

```typescript
import { int, mysqlTable } from "drizzle-orm/mysql-core";

export const table = mysqlTable('table', {
    int: int(),
    int2: int({ unsigned: true })
});
```
```sql
CREATE TABLE \`table\` (
    \`int\` int,
    \`int2\` int unsigned
);
```

### tinyint

A `1` -byte integer with a range of `-128` to `127` signed, or `0` to `255` unsigned

For more info please refer to the official MySQL **[docs.](https://dev.mysql.com/doc/refman/8.0/en/integer-types.html)**

```typescript
import { tinyint, mysqlTable } from "drizzle-orm/mysql-core";

export const table = mysqlTable('table', {
    tinyint: tinyint(),
    tinyint2: tinyint({ unsigned: true })
});
```
```sql
CREATE TABLE \`table\` (
    \`tinyint\` tinyint,
    \`tinyint2\` tinyint unsigned
);
```

### smallint

A `2` -byte integer with a range of `-32,768` to `32,767` signed, or `0` to `65,535` unsigned

For more info please refer to the official MySQL **[docs.](https://dev.mysql.com/doc/refman/8.0/en/integer-types.html)**

```typescript
import { smallint, mysqlTable } from "drizzle-orm/mysql-core";

export const table = mysqlTable('table', {
    smallint: smallint(),
    smallint2: smallint({ unsigned: true }),
});
```
```sql
CREATE TABLE \`table\` (
    \`smallint\` smallint,
    \`smallint2\` smallint unsigned
);
```

### mediumint

A `3` -byte integer with a range of `-8,388,608` to `8,388,607` signed, or `0` to `16,777,215` unsigned

For more info please refer to the official MySQL **[docs.](https://dev.mysql.com/doc/refman/8.0/en/integer-types.html)**

```typescript
import { mediumint, mysqlTable } from "drizzle-orm/mysql-core";

export const table = mysqlTable('table', {
    mediumint: mediumint(),
    mediumint2: mediumint({ unsigned: true })
});
```
```sql
CREATE TABLE \`table\` (
    \`mediumint\` mediumint,
    \`mediumint2\` mediumint unsigned
);
```

### bigint

An `8` -byte integer with a range of `-2^63` to `2^63-1` signed, or `0` to `2^64-1` unsigned

For more info please refer to the official MySQL **[docs.](https://dev.mysql.com/doc/refman/8.0/en/integer-types.html)**

```typescript
import { bigint, mysqlTable } from "drizzle-orm/mysql-core";

export const table = mysqlTable('table', {
    bigint: bigint({ mode: 'number' }),
    bigintUnsigned: bigint({ mode: 'number', unsigned: true })
});

bigint("...", { mode: 'number' | 'bigint' | 'string' });

// You can also specify unsigned option for bigint
bigint("...", { mode: 'number' | 'bigint' | 'string', unsigned: true });
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

A floating-point number

For more info please refer to the official MySQL **[docs.](https://dev.mysql.com/doc/refman/8.0/en/floating-point-types.html)**

```typescript
import { real, mysqlTable } from "drizzle-orm/mysql-core";

export const table = mysqlTable('table', {
    real: real()
});
```
```sql
CREATE TABLE \`table\` (
    \`real\` real
);
```

```typescript
import { real, mysqlTable } from "drizzle-orm/mysql-core";

export const table = mysqlTable('table', {
    realPrecision: real({ precision: 1 }),
    realPrecisionScale: real({ precision: 1, scale: 1 }),
});
```
```sql
CREATE TABLE \`table\` (
    \`realPrecision\` real(1),
    \`realPrecisionScale\` real(1, 1)
);
```

### decimal

An exact fixed-point number. `DECIMAL(M, D)` stores up to `M` total digits with `D` digits after the decimal point. Maximum precision is `65` digits

For more info please refer to the official MySQL **[docs.](https://dev.mysql.com/doc/refman/8.0/en/fixed-point-types.html)**

```typescript
import { decimal, mysqlTable } from "drizzle-orm/mysql-core";

export const table = mysqlTable('table', {
    decimal: decimal(),
    decimalNum: decimal({ precision: 30, mode: 'number' }),
    decimalBig: decimal({ precision: 30, mode: 'bigint' }),
    decimal2: decimal({ precision: 30, scale: 10 }),
});
```
```sql
CREATE TABLE \`table\` (
    \`decimal\` decimal,
    \`decimalNum\` decimal(30),
    \`decimalBig\` decimal(30),
    \`decimal2\` decimal(30,10)
);
```

### double

An `8` -byte double-precision floating-point number representing approximate numeric data values

For more info please refer to the official MySQL **[docs.](https://dev.mysql.com/doc/refman/8.0/en/floating-point-types.html)**

```typescript
import { double, mysqlTable } from "drizzle-orm/mysql-core";

export const table = mysqlTable('table', {
    double: double()
});
```
```sql
CREATE TABLE \`table\` (
    \`double\` double
);
```

```typescript
import { double, mysqlTable } from "drizzle-orm/mysql-core";

export const table = mysqlTable('table', {
    doublePrecision: double({ precision: 1 }),
    doublePrecisionScale: double({ precision: 1, scale: 1 })
});
```
```sql
CREATE TABLE \`table\` (
    \`doublePrecision\` double(1),
    \`doublePrecisionScale\` double(1,1)
);
```

### float

A `4` -byte single-precision floating-point number representing approximate numeric data values

For more info please refer to the official MySQL **[docs.](https://dev.mysql.com/doc/refman/8.0/en/floating-point-types.html)**

```typescript
import { float, mysqlTable } from "drizzle-orm/mysql-core";

export const table = mysqlTable('table', {
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

For more info please refer to the official MySQL **[docs.](https://dev.mysql.com/doc/refman/8.0/en/numeric-type-syntax.html)**

```typescript
import { serial, mysqlTable } from "drizzle-orm/mysql-core";

export const table = mysqlTable('table', {
    serial: serial()
});
```
```sql
CREATE TABLE \`table\` (
    \`serial\` serial
);
```

## \---

### binary

`BINARY(M)` stores a fixed-length byte string of exactly M bytes (If omitted, M defaults to 1).  
On insert, shorter values are right-padded with `0x00` bytes to reach M bytes; on retrieval, no padding is stripped. All bytes are significant in comparisons, including `ORDER BY` and `DISTINCT` operations

For more info please refer to the official MySQL **[docs.](https://dev.mysql.com/doc/refman/8.0/en/binary-varbinary.html)**

```typescript
import { binary, mysqlTable } from "drizzle-orm/mysql-core";

export const table = mysqlTable('table', {
    binary: binary(),
    binary2: binary({ length: 10 }),
});
```
```sql
CREATE TABLE \`table\` (
    \`binary\` binary,
    \`binary2\` binary(10)
);
```

### varbinary

`VARBINARY(M)` stores a variable-length byte string of up to M bytes (M as required).  
For `VARBINARY`, there is no padding for inserts and no bytes are stripped for retrievals. All bytes are significant in comparisons, including `ORDER BY` and `DISTINCT` operations

For more info please refer to the official MySQL **[docs.](https://dev.mysql.com/doc/refman/8.0/en/binary-varbinary.html)**

```typescript
import { varbinary, mysqlTable } from "drizzle-orm/mysql-core";

export const table = mysqlTable('table', {
    varbinary: varbinary({ length: 2 }),
});
```
```sql
CREATE TABLE \`table\` (
    \`varbinary\` varbinary(2)
);
```

## \---

### blob

A `BLOB` is a binary large object that can hold a variable amount of data with a maximum length of `65,535` bytes (`64` KB)

For more info please refer to the official MySQL **[docs.](https://dev.mysql.com/doc/refman/8.0/en/blob.html)**

```typescript
import { blob, mysqlTable } from "drizzle-orm/mysql-core";

export const table = mysqlTable('table', {
    blob: blob()
});
```
```sql
CREATE TABLE \`table\` (
    \`blob\` blob
);
```

You can specify either `buffer` or `string` mode:

```typescript
// will be inferred as \`Buffer\`
blob: blob({ mode: "buffer" }),

// will be inferred as \`string\`
blob: blob({ mode: "string" }),
```

> The `buffer` mode is the default way to work with blobs. Drizzle will represent the blob value as a `Buffer` instance

> The `string` mode represents blob values as UTF-8 encoded strings, which can be useful when storing text data in blob columns

### tinyblob

A `BLOB` column with a maximum length of `255` bytes

For more info please refer to the official MySQL **[docs.](https://dev.mysql.com/doc/refman/8.0/en/blob.html)**

```typescript
import { tinyblob, mysqlTable } from "drizzle-orm/mysql-core";

export const table = mysqlTable('table', {
    tinyblob: tinyblob()
});
```
```sql
CREATE TABLE \`table\` (
    \`tinyblob\` tinyblob
);
```

You can specify either `buffer` or `string` mode:

```typescript
// will be inferred as \`Buffer\`
tinyblob: tinyblob({ mode: "buffer" }),

// will be inferred as \`string\`
tinyblob: tinyblob({ mode: "string" }),
```

> The `buffer` mode is the default way to work with blobs. Drizzle will represent the blob value as a `Buffer` instance

> The `string` mode represents blob values as UTF-8 encoded strings, which can be useful when storing text data in blob columns

### mediumblob

A `BLOB` column with a maximum length of `16,777,215` bytes (`16` MB)

For more info please refer to the official MySQL **[docs.](https://dev.mysql.com/doc/refman/8.0/en/blob.html)**

```typescript
import { mediumblob, mysqlTable } from "drizzle-orm/mysql-core";

export const table = mysqlTable('table', {
    mediumblob: mediumblob()
});
```
```sql
CREATE TABLE \`table\` (
    \`mediumblob\` mediumblob
);
```

You can specify either `buffer` or `string` mode:

```typescript
// will be inferred as \`Buffer\`
mediumblob: mediumblob({ mode: "buffer" }),

// will be inferred as \`string\`
mediumblob: mediumblob({ mode: "string" }),
```

> The `buffer` mode is the default way to work with blobs. Drizzle will represent the blob value as a `Buffer` instance

> The `string` mode represents blob values as UTF-8 encoded strings, which can be useful when storing text data in blob columns

### longblob

A `BLOB` column with a maximum length of `4,294,967,295` bytes (`4` GB)

For more info please refer to the official MySQL **[docs.](https://dev.mysql.com/doc/refman/8.0/en/blob.html)**

```typescript
import { longblob, mysqlTable } from "drizzle-orm/mysql-core";

export const table = mysqlTable('table', {
    longblob: longblob()
});
```
```sql
CREATE TABLE \`table\` (
    \`longblob\` longblob
);
```

You can specify either `buffer` or `string` mode:

```typescript
// will be inferred as \`Buffer\`
longblob: longblob({ mode: "buffer" }),

// will be inferred as \`string\`
longblob: longblob({ mode: "string" }),
```

> The `buffer` mode is the default way to work with blobs. Drizzle will represent the blob value as a `Buffer` instance

> The `string` mode represents blob values as UTF-8 encoded strings, which can be useful when storing text data in blob columns

## \---

### char

The length of a `CHAR(M)` column is fixed to the length that you declare when you create the table. `M` can be any value from `0` to `255`, default is `1` When `CHAR` values are stored, they are right-padded with spaces to the specified length. When `CHAR` values are retrieved, trailing spaces are removed unless the [PAD\_CHAR\_TO\_FULL\_LENGTH](https://dev.mysql.com/doc/refman/9.7/en/sql-mode.html#sqlmode_pad_char_to_full_length) SQL mode is enabled.

For more info please refer to the official MySQL **[docs.](https://dev.mysql.com/doc/refman/8.0/en/char.html)**

You can define `{ enum: ["value1", "value2"] }` config to infer insert and select types, it won’t check runtime values.

```typescript
import { char, mysqlTable } from "drizzle-orm/mysql-core";

export const table = mysqlTable('table', {
    char: char(),
});

// will be inferred as char: "value1" | "value2" | null
char: char({ enum: ["value1", "value2"] })
```
```sql
CREATE TABLE \`table\` (
    \`char\` char
);
```

### varchar

Values in `VARCHAR` columns are variable-length strings.  
`VARCHAR(M)` stores up to `M` characters, where `M` ranges from `0` to `65,535` `VARCHAR` values are not padded when they are stored. Trailing spaces are retained when values are stored and retrieved, in conformance with standard SQL

For more info please refer to the official MySQL **[docs.](https://dev.mysql.com/doc/refman/8.0/en/char.html)**

You can define `{ enum: ["value1", "value2"] }` config to infer `insert` and `select` types, it **won’t** check runtime values.

```typescript
import { varchar, mysqlTable } from "drizzle-orm/mysql-core";

export const table = mysqlTable('table', {
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

A `TEXT` column with a maximum length of `65,535` characters (`64` KB)

For more info please refer to the official MySQL **[docs.](https://dev.mysql.com/doc/refman/8.0/en/blob.html)**

You can define `{ enum: ["value1", "value2"] }` config to infer `insert` and `select` types, it **won’t** check runtime values.

```typescript
import { text, mysqlTable } from "drizzle-orm/mysql-core";

export const table = mysqlTable('table', {
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

### tinytext

A `TEXT` column with a maximum length of `255` characters

For more info please refer to the official MySQL **[docs.](https://dev.mysql.com/doc/refman/8.0/en/blob.html)**

You can define `{ enum: ["value1", "value2"] }` config to infer `insert` and `select` types, it **won’t** check runtime values.

```typescript
import { tinytext, mysqlTable } from "drizzle-orm/mysql-core";

export const table = mysqlTable('table', {
    tinytext: tinytext(),
});

// will be inferred as tinytext: "value1" | "value2" | null
tinytext: tinytext({ enum: ["value1", "value2"] });
```
```sql
CREATE TABLE \`table\` (
    \`tinytext\` tinytext
);
```

### mediumtext

A `TEXT` column with a maximum length of `16,777,215` characters (`16` MB)

For more info please refer to the official MySQL **[docs.](https://dev.mysql.com/doc/refman/8.0/en/blob.html)**

You can define `{ enum: ["value1", "value2"] }` config to infer `insert` and `select` types, it **won’t** check runtime values.

```typescript
import { mediumtext, mysqlTable } from "drizzle-orm/mysql-core";

export const table = mysqlTable('table', {
    mediumtext: mediumtext(),
});

// will be inferred as mediumtext: "value1" | "value2" | null
mediumtext: mediumtext({ enum: ["value1", "value2"] });
```
```sql
CREATE TABLE \`table\` (
    \`mediumtext\` mediumtext
);
```

### longtext

A `TEXT` column with a maximum length of `4,294,967,295` characters (`4` GB)

For more info please refer to the official MySQL **[docs.](https://dev.mysql.com/doc/refman/8.0/en/blob.html)**

You can define `{ enum: ["value1", "value2"] }` config to infer `insert` and `select` types, it **won’t** check runtime values.

```typescript
import { longtext, mysqlTable } from "drizzle-orm/mysql-core";

export const table = mysqlTable('table', {
    longtext: longtext(),
});

// will be inferred as longtext: "value1" | "value2" | null
longtext: longtext({ enum: ["value1", "value2"] });
```
```sql
CREATE TABLE \`table\` (
    \`longtext\` longtext
);
```

## \---

### boolean

`BOOLEAN` is a synonym for `TINYINT(1)`. A value of `0` is considered false, non-zero values are considered true

For more info please refer to the official MySQL **[docs.](https://dev.mysql.com/doc/refman/8.0/en/numeric-type-syntax.html)**

```typescript
import { boolean, mysqlTable } from "drizzle-orm/mysql-core";

export const table = mysqlTable('table', {
    boolean: boolean(),
});
```
```sql
CREATE TABLE \`table\` (
    \`boolean\` boolean
);
```

## \---

A date value in `'YYYY-MM-DD'` format, with a range of `'1000-01-01'` to `'9999-12-31'`

For more info please refer to the official MySQL **[docs.](https://dev.mysql.com/doc/refman/8.0/en/datetime.html)**

```typescript
import { date, mysqlTable } from "drizzle-orm/mysql-core";

export const table = mysqlTable('table', {
    date: date(),
});
```
```sql
CREATE TABLE \`table\` (
    \`date\` date
);
```

You can specify either `date` or `string` infer modes:

```typescript
// will infer as date
date: date({ mode: "date" }),

// will infer as string
date: date({ mode: "string" }),
```

### datetime

A date and time value in `'YYYY-MM-DD hh:mm:ss'` format, with a range of `'1000-01-01 00:00:00'` to `'9999-12-31 23:59:59'`. Supports fractional seconds up to `6` digits

For more info please refer to the official MySQL **[docs.](https://dev.mysql.com/doc/refman/8.0/en/datetime.html)**

```typescript
import { datetime, mysqlTable } from "drizzle-orm/mysql-core";

export const table = mysqlTable('table', {
    datetime: datetime(),
});

datetime('...', { mode: 'date' | "string"}),
datetime('...', { fsp : 0..6}),
```
```sql
CREATE TABLE \`table\` (
    \`datetime\` datetime
);
```

```typescript
import { datetime, mysqlTable } from "drizzle-orm/mysql-core";

export const table = mysqlTable('table', {
    datetime: datetime({ mode: 'date', fsp: 6 }),
});
```
```sql
CREATE TABLE \`table\` (
    \`datetime\` datetime(6)
);
```

### time

A time value in `'hh:mm:ss'` format, with a range of `'-838:59:59'` to `'838:59:59'`. Can represent time of day or elapsed time. Supports fractional seconds up to `6` digits

For more info please refer to the official MySQL **[docs.](https://dev.mysql.com/doc/refman/8.0/en/time.html)**

```typescript
import { time, mysqlTable } from "drizzle-orm/mysql-core";

export const table = mysqlTable('table', {
    time: time(),
    timefsp: time({ fsp: 6 }),
});
    
time('...', { fsp: 0..6 }),
```
```sql
CREATE TABLE \`table\` (
    \`time\` time,
    \`timefsp\` time(6)
);
```

### year

A `1` -byte type for year values in `YYYY` format, with a range of `1901` to `2155` and `0000`

For more info please refer to the official MySQL **[docs.](https://dev.mysql.com/doc/refman/8.0/en/year.html)**

```typescript
import { year, mysqlTable } from "drizzle-orm/mysql-core";

export const table = mysqlTable('table', {
    year: year(),
});
```
```sql
CREATE TABLE \`table\` (
    \`year\` year
);
```

### timestamp

A date and time value in `'YYYY-MM-DD hh:mm:ss'` format, with a range of `'1970-01-01 00:00:01'` UTC to `'2038-01-19 03:14:07'` UTC. MySQL converts `TIMESTAMP` values from the current time zone to UTC for storage, and back from UTC to the current time zone for retrieval. Supports fractional seconds up to `6` digits

For more info please refer to the official MySQL **[docs.](https://dev.mysql.com/doc/refman/8.0/en/datetime.html)**

```typescript
import { timestamp, mysqlTable } from "drizzle-orm/mysql-core";

export const table = mysqlTable('table', {
    timestamp: timestamp(),
});

timestamp('...', { mode: 'date' | "string"}),
timestamp('...', { fsp : 0..6}),
```
```sql
CREATE TABLE \`table\` (
    \`timestamp\` timestamp
);
```

```typescript
import { timestamp, mysqlTable } from "drizzle-orm/mysql-core";

export const table = mysqlTable('table', {
    timestamp: timestamp({ mode: 'date', fsp: 6 }),
});
```
```sql
CREATE TABLE \`table\` (
    \`timestamp\` timestamp(6)
);
```

```typescript
import { timestamp, mysqlTable } from "drizzle-orm/mysql-core";

export const table = mysqlTable('table', {
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

A native `JSON` data type that enables efficient access to data in JSON documents. JSON documents are automatically validated and stored in an optimized binary format that permits quick read access to document elements

For more info please refer to the official MySQL **[docs.](https://dev.mysql.com/doc/refman/8.0/en/json.html)**

```typescript
import { json, mysqlTable } from "drizzle-orm/mysql-core";

export const table = mysqlTable('table', {
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

A string object with a value chosen from a list of permitted values that are enumerated at table creation time.

For more info please refer to the official MySQL **[docs.](https://dev.mysql.com/doc/refman/8.0/en/enum.html)**

```typescript
import { mysqlEnum, mysqlTable } from "drizzle-orm/mysql-core";

export const table = mysqlTable('table', {
    popularity: mysqlEnum(['unknown', 'known', 'popular']),
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
import { int, json, mysqlTable } from "drizzle-orm/mysql-core";

type UserId = number & { __brand: "user_id" };
type Data = {
  foo: string;
  bar: number;
};

export const users = mysqlTable("users", {
  id: int().$type<UserId>().primaryKey(),
  jsonField: json().$type<Data>(),
});
```

### Not null

`NOT NULL` constraint dictates that the associated column may not contain a `NULL` value.

```typescript
import { int, mysqlTable } from "drizzle-orm/mysql-core";

export const table = mysqlTable('table', {
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
import { sql } from "drizzle-orm";
import { int, mysqlTable } from "drizzle-orm/mysql-core";

export const table = mysqlTable("table", {
  int: int().default(3),
  int2: int().default(sql\`3\`),
});
```
```sql
CREATE TABLE \`table\` (
    \`int\` int DEFAULT 3,
    \`int2\` int DEFAULT (3)
);
```

When using `$default()` or `$defaultFn()`, which are simply different aliases for the same function, you can generate defaults at runtime and use these values in all insert queries. These functions can assist you in utilizing various implementations such as `uuid`, `cuid`, `cuid2`, and many more.

Note: This value does not affect the `drizzle-kit` behavior, it is only used at runtime in `drizzle-orm`

```ts
import { varchar, mysqlTable } from "drizzle-orm/mysql-core";
import { createId } from '@paralleldrive/cuid2';

export const table = mysqlTable('table', {
    id: varchar({ length: 128 }).$defaultFn(() => createId()),
});
```

When using `$onUpdate()` or `$onUpdateFn()`, which are simply different aliases for the same function, you can generate defaults at runtime and use these values in all update queries.

Adds a dynamic update value to the column. The function will be called when the row is updated, and the returned value will be used as the column value if none is provided. If no default (or $defaultFn) value is provided, the function will be called when the row is inserted as well, and the returned value will be used as the column value.

Note: This value does not affect the `drizzle-kit` behavior, it is only used at runtime in `drizzle-orm`

```ts
import { text, mysqlTable } from "drizzle-orm/mysql-core";

export const table = mysqlTable('table', {
    alwaysNull: text().$type<string | null>().$onUpdate(() => null),
});
```

### Primary key

```typescript
import { int, mysqlTable } from "drizzle-orm/mysql-core";

export const table = mysqlTable('table', {
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
import { int, mysqlTable } from "drizzle-orm/mysql-core";

export const table = mysqlTable('table', {
    int: int().autoincrement(),
});
```
```sql
CREATE TABLE \`table\` (
    \`int\` int AUTO_INCREMENT
);
```
