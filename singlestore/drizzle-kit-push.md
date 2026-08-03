---
url: https://orm.drizzle.team/docs/singlestore/drizzle-kit-push
title: "Drizzle Kit Push"
description: ""
access_date: 2026-08-03T18:54:19.022Z
current_date: 2026-08-03T18:54:19.022Z
---

## drizzle-kit push

This guide assumes familiarity with:

- Get started with Drizzle and `drizzle-kit` - [read here](https://orm.drizzle.team/docs/singlestore/get-started)
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
import * as p from "drizzle-orm/singlestore-core";

export const users = p.singlestoreTable("users", {
  id: p.int().primaryKey().autoincrement(),
  name: p.varchar({ length: 255 }),
};
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
  ┌────────────────────────────────────┴────────────────┐
   create table users(\`id\` int auto_increment primary key, \`name\` varchar(255));
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
  dialect: "singlestore",
  schema: "./src/schema.ts",
  dbCredentials: {
    url: "mysql://user:password@host:3306/dbname",
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

Drizzle Kit does not come with a pre-bundled database driver, it will automatically pick an available SingleStore driver from your current project based on `dialect: "singlestore"` - [see discussion](https://github.com/drizzle-team/drizzle-orm/discussions/2203).

SingleStore uses MySQL-compatible connection parameters, so you usually only need to provide `dbCredentials`.

### Including tables

`drizzle-kit push` will manage all SingleStore tables by default. You can configure the list of tables via `tablesFilter`.

|  |  |
| --- | --- |
| `tablesFilter` | `glob` based table names filter, e.g. `["users", "user_info"]` or `"user*"`. Default is `"*"` |

Let’s configure drizzle-kit to operate with **all tables** in the connected SingleStore database.

```ts
import { defineConfig } from "drizzle-kit";

export default defineConfig({
  dialect: "singlestore",
  schema: "./src/schema.ts",
  dbCredentials: {
    url: "mysql://user:password@host:3306/dbname",
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
| `strict` | always ask for approval before executing SQL statements |
| `force` | auto-accept all data-loss statements |

```shell
npx drizzle-kit push --strict --verbose --force
```

---

We recommend configuring `drizzle-kit` through [drizzle.config.ts](drizzle-config-file.md) file, yet you can provide all configuration options through CLI if necessary, e.g. in CI/CD pipelines, etc.

|  |  |  |
| --- | --- | --- |
| `dialect` | `required` | Database dialect, one of `postgresql` `mysql` `sqlite` `turso` `singlestore` `mssql` `cockroach` |
| `schema` | `required` | Path to typescript schema file(s) or folder(s) with multiple schema files |
| `tablesFilter` |  | Table name filter |
| `url` |  | Database connection string |
| `user` |  | Database user |
| `password` |  | Database password |
| `host` |  | Host |
| `port` |  | Port |
| `database` |  | Database name |
| `config` |  | Configuration file path, default= `drizzle.config.ts` |

```shell
npx drizzle-kit push dialect=singlestore schema=src/schema.ts url=mysql://user:password@host:3306/dbname
npx drizzle-kit push dialect=singlestore schema=src/schema.ts --tablesFilter=‘user*’ url=mysql://user:password@host:3306/dbname
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
  dialect: "singlestore",
  schema: "./src/schema.ts",
  dbCredentials: {
    url: "mysql://user:password@host:3306/dbname"
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
  \`id\` int auto_increment primary key,
  \`name\` varchar(255)
)
```

DONE ✅
