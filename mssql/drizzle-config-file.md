---
url: https://orm.drizzle.team/docs/mssql/drizzle-config-file
title: "Drizzle Config File"
description: ""
access_date: 2026-08-03T19:00:22.305Z
current_date: 2026-08-03T19:00:22.305Z
---

## Drizzle Kit configuration file

This guide assumes familiarity with:

- Get started with Drizzle and `drizzle-kit` - [read here](https://orm.drizzle.team/docs/mssql/get-started)
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
  dialect: "mssql",
  schema: "./src/schema.ts",
  out: "./drizzle",
});
```

Example of an extended config file

```ts
import { defineConfig } from "drizzle-kit";

export default defineConfig({
  out: "./drizzle",
  dialect: "mssql",
  schema: "./src/schema.ts",

  dbCredentials: {
    url: "mssql://user:password@host:3306/dbname",
  },

  schemaFilter: "dbo",
  tablesFilter: "*",

  introspect: {
    casing: "camel",
  },

  migrations: {
    table: "__drizzle_migrations__",
    schema: "drizzle",
  },

  breakpoints: true,
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
  dialect: "mssql",
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
| commands | `generate`, `push`, `pull`, `studio`, `migrate`, `up`, `export` |

```ts
import { defineConfig } from "drizzle-kit";

export default defineConfig({
  dialect: "mssql", 
});
```

### schema

[`glob`](https://www.digitalocean.com/community/tools/glob?comments=true&glob=/**/*.js&matches=false&tests=//%20This%20will%20match%20as%20it%20ends%20with%20%27.js%27&tests=/hello/world.js&tests=//%20This%20won%27t%20match!&tests=/test/some/globs) based path to drizzle schema file(s) or folder(s) contaning schema files.

|  |  |
| --- | --- |
| type | `string` `string[]` |
| default | — |
| commands | `generate`, `push`, `export`, `studio` |

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
| commands | `generate`, `pull`, `migrate`, `check`, `up` |

```ts
import { defineConfig } from "drizzle-kit";

export default defineConfig({
  out: "./drizzle", 
});
```

## \---

### dbCredentials

Database connection credentials in a form of `url` or `user:password@host:port/db` params.

|  |  |
| --- | --- |
| type | `postgresql` `mysql` `sqlite` `turso` `singlestore` `mssql` `cockroach` |
| default | — |
| commands | `push`, `pull`, `migrate`, `studio` |

```ts
import { defineConfig } from 'drizzle-kit'

export default defineConfig({
  dialect: "mssql",
  dbCredentials: {
    url: "mssql://user:password@host:3306/db",
  }
});
```

### migrations

When running `drizzle-kit migrate` - drizzle will records about successfully applied migrations in your database in log table named `__drizzle_migrations` in `drizzle` schema.

`migrations` config options lets you change both migrations log `table` name and `schema`.

|  |  |
| --- | --- |
| type | `{ table: string, schema: string }` |
| default | `{ table: "__drizzle_migrations", schema: "drizzle" }` |
| commands | `migrate`, `push`, `pull` |

```ts
export default defineConfig({
  dialect: "mssql",
  schema: "./src/schema.ts",
  migrations: {
    table: 'my-migrations-table', // \`__drizzle_migrations\` by default
    schema: 'my_schema', // \`drizzle\` by default
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
import * as p from "drizzle-orm/mssql-core"

export const users = p.mssqlTable("users", {
  id: p.int().primaryKey().identity(),
  firstName: p.varchar("first-name", { length: 255 }),
  lastName: p.varchar("LastName", { length: 255 }),
  email: p.varchar("email", { length: 255 }),
  phoneNumber: p.varchar("phone_number", { length: 255 }),
});
```
```sql
SELECT COLUMN_NAME, DATA_TYPE FROM INFORMATION_SCHEMA.COLUMNS WHERE TABLE_NAME = 'users';
```
```plaintext
COLUMN_NAME   | DATA_TYPE        
---------------+--------------
 id            | int
 first-name    | varchar
 LastName      | varchar
 email         | varchar
 phone_number  | varchar
```

## \---

### tablesFilter

If you want to run multiple projects with one database - check out [our guide](goodies.md#multi-project-schema).

`drizzle-kit push` and `drizzle-kit pull` will manage all tables by default. You can configure list of tables and schemas via `tablesFilters` and `schemaFilter` options.

`tablesFilter` option lets you specify [`glob`](https://www.digitalocean.com/community/tools/glob?comments=true&glob=/**/*.js&matches=false&tests=//%20This%20will%20match%20as%20it%20ends%20with%20%27.js%27&tests=/hello/world.js&tests=//%20This%20won%27t%20match!&tests=/test/some/globs) based table names filter, e.g. `["users", "user_info"]` or `"user*"`

|  |  |
| --- | --- |
| type | `string` `string[]` |
| default | — |
| commands | `push` `pull` |

```ts
import { defineConfig } from "drizzle-kit";

export default defineConfig({
  dialect: "mssql",
  tablesFilter: ["users", "posts", "project1_*"],
});
```

### schemaFilter

If you want to run multiple projects with one database - check out [our guide](goodies.md#multi-project-schema).

`drizzle-kit push` and `drizzle-kit pull` will by default manage all schemas.

`schemaFilter` option lets you specify [`glob`](https://www.digitalocean.com/community/tools/glob?comments=true&glob=/**/*.js&matches=false&tests=//%20This%20will%20match%20as%20it%20ends%20with%20%27.js%27&tests=/hello/world.js&tests=//%20This%20won%27t%20match!&tests=/test/some/globs) based schema names filter, e.g. `["dbo", "auth"]` or `"tenant_*"`

|  |  |
| --- | --- |
| type | `string[]` |
| commands | `push` `pull` |

```ts
export default defineConfig({
  dialect: "mssql",
  schemaFilter: ["dbo", "schema1", "schema2"],
});
```

## \---

### verbose

Print all SQL statements during `drizzle-kit push` command.

|  |  |
| --- | --- |
| type | `boolean` |
| default | `false` |
| commands | `pull` |

```ts
export default defineConfig({
  dialect: "mssql",
  verbose: false,
});
```

### breakpoints

Drizzle Kit will automatically embed `--> statement-breakpoint` into generated SQL migration files, that’s necessary for databases that do not support multiple DDL alternation statements in one transaction.

`breakpoints` option flag lets you switch it on and off

|  |  |
| --- | --- |
| type | `boolean` |
| default | `true` |
| commands | `generate` `pull` |

```ts
export default defineConfig({
  dialect: "mssql",
  breakpoints: false,
});
```
