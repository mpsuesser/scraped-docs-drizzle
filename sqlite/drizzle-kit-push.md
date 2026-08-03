---
url: https://orm.drizzle.team/docs/sqlite/drizzle-kit-push
title: "Drizzle Kit Push"
description: ""
access_date: 2026-08-03T19:38:26.356Z
current_date: 2026-08-03T19:38:26.356Z
---

## drizzle-kit push

This guide assumes familiarity with:

- Get started with Drizzle and `drizzle-kit` - [read here](https://orm.drizzle.team/docs/sqlite/get-started)
- Drizzle schema fundamentals - [read here](sql-schema-declaration.md)
- Database connection basics - [read here](connect-overview.md)
- Drizzle migrations fundamentals - [read here](migrations.md)
- Drizzle Kit [overview](kit-overview.md) and [config file](drizzle-config-file.md) docs

`drizzle-kit push` lets you literally push your schema and subsequent schema changes directly to the database while omitting SQL files generation, it’s designed to cover [code first](migrations.md) approach of Drizzle migrations.

How it works under the hood?

When you run Drizzle Kit `push` command it will:

1. Read through your Drizzle schema file(s) and compose a json snapshot of your schema
2. Pull(introspect) database schema
3. Based on differences between those two it will generate SQL migrations
4. Apply SQL migrations to the database

```typescript
import * as p from "drizzle-orm/sqlite-core";

export const users = p.sqliteTable("users", {
  id: p.integer().primaryKey({ autoIncrement: true }),
  name: p.text(),
});
```
```plaintext
┌─────────────────────┐                  
│ ~ drizzle-kit push  │                  
└─┬───────────────────┘                  
  │                                           ┌──────────────────────────┐
  └ Pull current datatabase schema ---------> │                          │
                                              │                          │
  ┌ Generate alternations based on diff <---- │         DATABASE         │
  │                                           │                          │
  └ Apply migrations to the database -------> │                          │
                                       │      └──────────────────────────┘
                                       │
  ┌────────────────────────────────────┴───────────────────────────────────────┐
   CREATE TABLE \`users\` (\`id\` integer PRIMARY KEY AUTOINCREMENT, \`name\` text );
```

It’s the best approach for rapid prototyping and we’ve seen dozens of teams and solo developers successfully using it as a primary migrations flow in their production applications. It pairs exceptionally well with blue/green deployment strategy and serverless databases like [Planetscale](https://planetscale.com/), [Neon](https://neon.tech/), [Turso](https://turso.tech/drizzle) and others.

---

`drizzle-kit push` requires you to specify `dialect`, path to the `schema` file(s) and either database connection `url` or `user:password@host:port/db` params, you can provide them either via [drizzle.config.ts](drizzle-config-file.md) config file or via CLI options:

With config file

With CLI options

```ts
// drizzle.config.ts
import { defineConfig } from "drizzle-kit";

export default defineConfig({
  dialect: "sqlite",
  schema: "./src/schema.ts",
  dbCredentials: {
    url: "./sqlite.db",
  },
});
```
```shell
npx drizzle-kit push
```

### Schema files path

You can have a single `schema.ts` file or as many schema files as you want spread out across the project. Drizzle Kit requires you to specify path(s) to them as a [glob](https://www.digitalocean.com/community/tools/glob?comments=true&glob=/**/*.js&matches=false&tests=//%20This%20will%20match%20as%20it%20ends%20with%20%27.js%27&tests=/hello/world.js&tests=//%20This%20won%27t%20match!&tests=/test/some/globs) via `schema` configuration option.

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

### Multiple configuration files in one project

You can have multiple config files in the project, it’s very useful when you have multiple database stages or multiple databases or different databases on the same project:

```shell
npx drizzle-kit push --config=drizzle-dev.config.ts
npx drizzle-kit push --config=drizzle-prod.config.ts
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

### Specifying database driver

IMPORTANT

**Expo SQLite** and **OP SQLite** are on-device(per-user) databases, there’s no way to `push` migrations there.  
For embedded databases Drizzle provides **embedded migrations** - check out our [get started](../get-started/expo-new.md) guide.

Drizzle Kit does not come with a pre-bundled database driver, it will automatically pick an available SQLite driver from your current project based on the `dialect` - [see discussion](https://github.com/drizzle-team/drizzle-orm/discussions/2203).

For Cloudflare D1 HTTP, specify `driver: "d1-http"` explicitly.

```ts
import { defineConfig } from "drizzle-kit";

export default defineConfig({
  dialect: "sqlite",
  driver: "d1-http",
  dbCredentials: {
    accountId: "accountId",
    databaseId: "databaseId",
    token: "token",
  },
});
```

### Including tables

`drizzle-kit push` will manage all SQLite tables by default. You can configure the list of tables via `tablesFilter`.

|  |  |
| --- | --- |
| `tablesFilter` | `glob` based table names filter, e.g. `["users", "user_info"]` or `"user*"`. Default is `"*"` |

Let’s configure drizzle-kit to operate with **all tables** in the SQLite database.

```ts
import { defineConfig } from "drizzle-kit";

export default defineConfig({
  dialect: "sqlite",
  schema: "./src/schema.ts",
  dbCredentials: {
    url: "./sqlite.db",
  },
  tablesFilter: ["*"],
});
```
```shell
npx drizzle-kit push
```

### Extended list of configurations

`drizzle-kit push` has a list of cli-only options

|  |  |
| --- | --- |
| `verbose` | print all SQL statements prior to execution |
| `explain` | print the planned SQL changes, without applying them (dry run) |
| `force` | auto-accept all data-loss statements |

```shell
npx drizzle-kit push --explain --verbose --force
```

---

We recommend configuring `drizzle-kit` through [drizzle.config.ts](drizzle-config-file.md) file, yet you can provide all configuration options through CLI if necessary, e.g. in CI/CD pipelines, etc.

|  |  |  |
| --- | --- | --- |
| `dialect` | `required` | Database dialect, one of `postgresql` `mysql` `sqlite` `turso` `singlestore` `mssql` `cockroach` |
| `schema` | `required` | Path to typescript schema file(s) or folder(s) with multiple schema files |
| `driver` |  | Driver exception, for example `d1-http` |
| `tablesFilter` |  | Table name filter |
| `url` |  | Database connection string |
| `user` |  | Database user |
| `password` |  | Database password |
| `host` |  | Host |
| `port` |  | Port |
| `database` |  | Database name |
| `config` |  | Configuration file path, default= `drizzle.config.ts` |

```shell
npx drizzle-kit push dialect=sqlite schema=src/schema.ts url=./sqlite.db
npx drizzle-kit push dialect=sqlite schema=src/schema.ts --tablesFilter=‘user*’ url=./sqlite.db
```

### Extended example

Let’s declare drizzle schema in the project and push it to the database via `drizzle-kit push` command

```plaintext
📦 <project root>
 ├ 📂 src
 │ ├ 📜 schema.ts
 │ └ 📜 index.ts
 ├ 📜 drizzle.config.ts
 └ …
```

```ts
import { defineConfig } from "drizzle-kit";

export default defineConfig({
  dialect: "sqlite",
  schema: "./src/schema.ts",
  dbCredentials: {
    url: "./sqlite.db"
  },
});
```

Now let’s run

```shell
npx drizzle-kit push
```

it will pull existing(empty) schema from the database and generate SQL migration and apply it under the hood

```sql
CREATE TABLE \`users\`(
  \`id\` integer primary key autoincrement,
  \`name\` text
)
```

DONE ✅
