---
url: https://orm.drizzle.team/docs/mssql/drizzle-kit-pull
title: "Drizzle Kit Pull"
description: ""
access_date: 2026-08-03T19:43:09.006Z
current_date: 2026-08-03T19:43:09.006Z
---

## drizzle-kit pull

This guide assumes familiarity with:

- Get started with Drizzle and `drizzle-kit` - [read here](https://orm.drizzle.team/docs/mssql/get-started)
- Drizzle schema fundamentals - [read here](sql-schema-declaration.md)
- Database connection basics - [read here](connect-overview.md)
- Drizzle migrations fundamentals - [read here](migrations.md)
- Drizzle Kit [overview](kit-overview.md) and [config file](drizzle-config-file.md) docs

`drizzle-kit pull` lets you literally pull(introspect) your existing database schema and generate `schema.ts` drizzle schema file, it is designed to cover [database first](migrations.md) approach of Drizzle migrations.

How it works under the hood?

When you run Drizzle Kit `pull` command it will:

1. Pull database schema(DDL) from your existing database
2. Generate `schema.ts` drizzle schema file and save it to `out` folder

```plaintext
┌────────────────────────┐      ┌───────────────────────────────┐ 
                                  │                        │ <---  CREATE TABLE [users] (
┌──────────────────────────┐      │                        │        [id] int identity primary key,
│ ~ drizzle-kit pull       │      │                        │        [name] varchar(255),
└─┬────────────────────────┘      │        DATABASE        │        [email] varchar(255) unique
  │                               │                        │       );
  └ Pull datatabase schema -----> │                        │
  ┌ Generate Drizzle       <----- │                        │
  │ schema TypeScript file        └────────────────────────┘
  │
  v
```
```typescript
import * as p from "drizzle-orm/mssql-core";

export const users = p.mssqlTable("users", {
    id: p.int().identity({ seed: 1 ,increment: 1 }),
    name: p.varchar({ length: 255 }),
    email: p.varchar({ length: 255 }),
}, (table) => [
    p.primaryKey({ columns: [table.id], name: "PK__users__3213E83F9C94E8E5"}),
    p.unique("UQ__users__AB6E6164167EE806").on(table.email)
]);
```

It is a great approach if you need to manage database schema outside of your TypeScript project or you’re using database, which is managed by somebody else.

---

`drizzle-kit pull` requires you to specify `dialect` and either database connection `url` or `user:password@host:port/db` params, you can provide them either via [drizzle.config.ts](drizzle-config-file.md) config file or via CLI options:

With config file

With CLI options

```ts
// drizzle.config.ts
import { defineConfig } from "drizzle-kit";

export default defineConfig({
  dialect: "mssql",
  dbCredentials: {
    url: "mssql://user:password@host:port/dbname",
  },
});
```
```shell
npx drizzle-kit pull
```

### Multiple configuration files in one project

You can have multiple config files in the project, it’s very useful when you have multiple database stages or multiple databases or different databases on the same project:

```shell
npx drizzle-kit pull --config=drizzle-dev.config.ts
npx drizzle-kit pull --config=drizzle-prod.config.ts
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

### Initial pull

You can use the `--init` flag to mark the pulled schema as an applied migration in your database, so that all subsequent migrations are diffed against the initial one

```shell
npx drizzle-kit pull --init
```

### Including tables and schemas

`drizzle-kit pull` will by default manage all tables in all schemas. You can configure list of tables and schemas via `tablesFilter` and `schemaFilter` options.

|  |  |
| --- | --- |
| `tablesFilter` | `glob` based table names filter, e.g. `["users", "user_info"]` or `"user*"`. Default is `"*"` |
| `schemaFilter` | `glob` based schema names filter, e.g. `["dbo", "drizzle"]` or `"drizzle*"`. Default is `"*"` |

Let’s configure drizzle-kit to only operate with **all tables** in **dbo** schema.

```ts
import { defineConfig } from "drizzle-kit";

export default defineConfig({
  dialect: "mssql",
  schema: "./src/schema.ts",
  dbCredentials: {
    url: "mssql://user:password@host:port/dbname",
  },
  schemaFilter: ["dbo"],
  tablesFilter: ["*"],
});
```
```shell
npx drizzle-kit pull
```

### Extended list of configurations

We recommend configuring `drizzle-kit` through [drizzle.config.ts](drizzle-config-file.md) file, yet you can provide all configuration options through CLI if necessary, e.g. in CI/CD pipelines, etc.

|  |  |  |
| --- | --- | --- |
| `dialect` | `required` | Database dialect, one of `postgresql` `mysql` `sqlite` `turso` `singlestore` `mssql` `cockroach` |
| `out` |  | Migrations output folder path, default is `./drizzle` |
| `url` |  | Database connection string |
| `user` |  | Database user |
| `password` |  | Database password |
| `host` |  | Host |
| `port` |  | Port |
| `database` |  | Database name |
| `config` |  | Configuration file path, default is `drizzle.config.ts` |
| `introspect-casing` |  | Strategy for JS keys creation in columns, tables, etc. `preserve` `camel` |
| `tablesFilter` |  | Table name filter |
| `schemaFilter` |  | Schema name filter. Default: `["*"]` |

```shell
npx drizzle-kit pull --dialect=mssql --url=mssql://user:password@host:port/dbname
npx drizzle-kit pull --dialect=mssql --tablesFilter=‘user*’ url=mssql://user:password@host:port/dbname
```

![](https://orm.drizzle.team/_astro/introspect_mysql.Hk8acObY_Z20dDww.webp)
