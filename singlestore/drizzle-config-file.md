---
url: https://orm.drizzle.team/docs/singlestore/drizzle-config-file
title: "Drizzle Config File"
description: ""
access_date: 2026-08-03T18:54:19.022Z
current_date: 2026-08-03T18:54:19.022Z
---

## Drizzle Kit configuration file

This guide assumes familiarity with:

- Get started with Drizzle and `drizzle-kit` - [read here](https://orm.drizzle.team/docs/singlestore/get-started)
- Drizzle schema fundamentals - [read here](sql-schema-declaration.md)
- Database connection basics - [read here](connect-overview.md)
- Drizzle migrations fundamentals - [read here](migrations.md)
- Drizzle Kit [overview](kit-overview.md) and [config file](drizzle-config-file.md)

Drizzle Kit lets you declare configuration options in `TypeScript` or `JavaScript` configuration files.

```plaintext
📦 <project root>
 ├ ...
 ├ 📂 drizzle
 ├ 📂 src
 ├ 📜 drizzle.config.ts
 └ 📜 package.json
```

drizzle.config.ts

drizzle.config.js

```ts
import { defineConfig } from "drizzle-kit";

export default defineConfig({
  dialect: "singlestore",
  schema: "./src/schema.ts",
  out: "./drizzle",
});
```

Example of an extended config file

```ts
import { defineConfig } from "drizzle-kit";

export default defineConfig({
  out: "./drizzle",
  dialect: "singlestore",
  schema: "./src/schema.ts",

  dbCredentials: {
    url: "mysql://user:password@host:3306/dbname",
  },

  introspect: {
    casing: "camel",
  },

  migrations: {
    prefix: "timestamp",
    table: "__drizzle_migrations__",
  },

  breakpoints: true,
  strict: true,
  verbose: true,
});
```

### Multiple configuration files

You can have multiple config files in the project, it’s very useful when you have multiple database stages or multiple databases or different databases on the same project:

```shell
npx drizzle-kit generate --config=drizzle-dev.config.ts
npx drizzle-kit generate --config=drizzle-prod.config.ts
```

```plaintext
📦 <project root>
 ├ 📂 drizzle
 ├ 📂 src
 ├ 📜 .env
 ├ 📜 drizzle-dev.config.ts
 ├ 📜 drizzle-prod.config.ts
 ├ 📜 package.json
 └ 📜 tsconfig.json
```

### Migrations folder

`out` param lets you define folder for your migrations, it’s optional and `drizzle` by default.  
It’s very useful since you can have many separate schemas for different databases in the same project and have different migration folders for them.

Migration folder contains folders with `.sql` migration files which is used by `drizzle-kit`

```plaintext
📦 <project root>
 ├ ...
 ├ 📂 drizzle
 │ ├ 📂 20242409125510_premium_mister_fear
 │ ├ 📜 user.ts 
 │ ├ 📜 post.ts 
 │ └ 📜 comment.ts 
 ├ 📂 src
 ├ 📜 drizzle.config.ts
 └ 📜 package.json
```
```ts
import { defineConfig } from "drizzle-kit";

export default defineConfig({
  dialect: "singlestore", // "mysql" | "sqlite" | "postgresql" | "turso" | "singlestore" | "mssql"
  schema: "./src/schema/*",
  out: "./drizzle",
});
```

## \---

### dialect

Dialect of the database you’re using

|  |  |
| --- | --- |
| type | `postgresql` `mysql` `sqlite` `turso` `singlestore` `mssql` `cockroach` |
| default | — |
| commands | `generate` `migrate` `push` `pull` `check` `up` |

```ts
import { defineConfig } from "drizzle-kit";

export default defineConfig({
  dialect: "mysql", 
});
```

### schema

[`glob`](https://www.digitalocean.com/community/tools/glob?comments=true&glob=/**/*.js&matches=false&tests=//%20This%20will%20match%20as%20it%20ends%20with%20%27.js%27&tests=/hello/world.js&tests=//%20This%20won%27t%20match!&tests=/test/some/globs) based path to drizzle schema file(s) or folder(s) contaning schema files.

|  |  |
| --- | --- |
| type | `string` `string[]` |
| default | — |
| commands | `generate` `push` |

Example 1

Example 2

Example 3

Example 4

```plaintext
📦 <project root>
 ├ ...
 ├ 📂 drizzle
 ├ 📂 src
 │ ├ ...
 │ ├ 📜 index.ts
 │ └ 📜 schema.ts 
 ├ 📜 drizzle.config.ts
 └ 📜 package.json
```
```ts
import { defineConfig } from "drizzle-kit";

export default defineConfig({
  schema: "./src/schema.ts",
});
```

### out

Defines output folder of your SQL migration files, json snapshots of your schema and `schema.ts` from `drizzle-kit pull` command.

|  |  |
| --- | --- |
| type | `string` `string[]` |
| default | `drizzle` |
| commands | `generate` `migrate` `push` `pull` `check` `up` |

```ts
import { defineConfig } from "drizzle-kit";

export default defineConfig({
  out: "./drizzle", 
});
```

### driver

Drizzle Kit automatically picks an available SingleStore driver from your current project based on `dialect: "singlestore"`. SingleStore uses MySQL-compatible connection parameters, so you usually do not need a separate `driver` option.

|  |  |
| --- | --- |
| type | — |
| default | — |
| commands | `migrate` `push` `pull` |

## \---

### dbCredentials

Database connection credentials in a form of `url`, `user:password@host:port/db` params or exceptions drivers(`aws-data-api` `d1-http` `pglite` ) specific connection options.

|  |  |
| --- | --- |
| type | union of drivers connection options |
| default | — |
| commands | `migrate` `push` `pull` |

Connection string

Connection params

```ts
import { defineConfig } from 'drizzle-kit'

export default defineConfig({
  dialect: "singlestore",
  dbCredentials: {
    url: "mysql://user:password@host:3306/db",
  }
});
```

### migrations

When running `drizzle-kit migrate` - drizzle will records about successfully applied migrations in your database in log table named `__drizzle_migrations`.

`migrations` config options lets you change the migrations log `table` name.

|  |  |
| --- | --- |
| type | `{ table: string }` |
| default | `{ table: "__drizzle_migrations" }` |
| commands | `migrate` |

```ts
export default defineConfig({
  dialect: "singlestore",
  schema: "./src/schema.ts",
  migrations: {
    table: 'my-migrations-table', // \`__drizzle_migrations\` by default
  },
});
```

### introspect

Configuration for `drizzle-kit pull` command.

`casing` is responsible for in-code column keys casing

|  |  |
| --- | --- |
| type | `{ casing: "preserve" \| "camel" }` |
| default | `{ casing: "camel" }` |
| commands | `pull` |

camel

preserve

```ts
import * as p from "drizzle-orm/singlestore-core"

export const users = p.singlestoreTable("users", {
  id: p.int().primaryKey().autoincrement(),
  firstName: p.varchar("first-name", { length: 255 }),
  lastName: p.varchar("LastName", { length: 255 }),
  email: p.varchar("email", { length: 255 }),
  phoneNumber: p.varchar("phone_number", { length: 255 }),
});
```
```sql
SHOW COLUMNS FROM users;
```
```plaintext
Field        | Type
--------------+--------------
 id           | int
 first-name   | varchar(255)
 LastName     | varchar(255)
 email        | varchar(255)
 phone_number | varchar(255)
```

## \---

### tablesFilter

If you want to run multiple projects with one database - check out [our guide](goodies.md#multi-project-schema).

`drizzle-kit push` and `drizzle-kit pull` will manage all tables by default. You can configure the list of tables via `tablesFilter`.

`tablesFilter` option lets you specify [`glob`](https://www.digitalocean.com/community/tools/glob?comments=true&glob=/**/*.js&matches=false&tests=//%20This%20will%20match%20as%20it%20ends%20with%20%27.js%27&tests=/hello/world.js&tests=//%20This%20won%27t%20match!&tests=/test/some/globs) based table names filter, e.g. `["users", "user_info"]` or `"user*"`

|  |  |
| --- | --- |
| type | `string` `string[]` |
| default | — |
| commands | `generate` `push` `pull` |

```ts
import { defineConfig } from "drizzle-kit";

export default defineConfig({
  dialect: "singlestore",
  tablesFilter: ["users", "posts", "project1_*"],
});
```

## \---

### strict

Prompts confirmation to run printed SQL statements when running `drizzle-kit push` command.

|  |  |
| --- | --- |
| type | `boolean` |
| default | `false` |
| commands | `push` |

```ts
export default defineConfig({
  dialect: "singlestore",
  strict: false,
});
```

### verbose

Print all SQL statements during `drizzle-kit push` command.

|  |  |
| --- | --- |
| type | `boolean` |
| default | `true` |
| commands | `generate` `pull` |

```ts
export default defineConfig({
  dialect: "singlestore",
  verbose: false,
});
```

### breakpoints

Drizzle Kit will automatically embed `--> statement-breakpoint` into generated SQL migration files, that’s necessary for databases that do not support multiple DDL alternation statements in one transaction(MySQL and SQLite).

`breakpoints` option flag lets you switch it on and off

|  |  |
| --- | --- |
| type | `boolean` |
| default | `true` |
| commands | `generate` `pull` |

```ts
export default defineConfig({
  dialect: "singlestore",
  breakpoints: false,
});
```
