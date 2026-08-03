---
url: https://orm.drizzle.team/docs/mysql/drizzle-kit-pull
title: "Drizzle Kit Pull"
description: ""
access_date: 2026-08-03T18:54:19.022Z
current_date: 2026-08-03T18:54:19.022Z
---

## drizzle-kit pull

This guide assumes familiarity with:

- Get started with Drizzle and `drizzle-kit` - [read here](https://orm.drizzle.team/docs/mysql/get-started)
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
┌────────────────────────┐      ┌─────────────────────────┐ 
                                  │                        │ <---  CREATE TABLE \`users\` (
┌──────────────────────────┐      │                        │        \`id\` int auto_increment primary key,
│ ~ drizzle-kit pull       │      │                        │        \`name\` varchar(255),
└─┬────────────────────────┘      │        DATABASE        │        \`email\` varchar(255) unique
  │                               │                        │       );
  └ Pull datatabase schema -----> │                        │
  ┌ Generate Drizzle       <----- │                        │
  │ schema TypeScript file        └────────────────────────┘
  │
  v
```
```typescript
import * as p from "drizzle-orm/mysql-core";

export const users = p.mysqlTable(
  "users",
  {
    id: p.int().autoincrement().primaryKey(),
    name: p.varchar({ length: 255 }),
    email: p.varchar({ length: 255 }),
  },
  (table) => [p.uniqueIndex("email").on(table.email)]
);
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
  dialect: "mysql",
  dbCredentials: {
    url: "mysql://user:password@host:3306/dbname",
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

### Including tables

`drizzle-kit pull` will introspect all MySQL tables by default. You can configure the list of tables via `tablesFilter`.

|  |  |
| --- | --- |
| `tablesFilter` | `glob` based table names filter, e.g. `["users", "user_info"]` or `"user*"`. Default is `"*"` |

Let’s configure drizzle-kit to introspect **all tables** in the connected MySQL database.

```ts
import { defineConfig } from "drizzle-kit";

export default defineConfig({
  dialect: "mysql",
  dbCredentials: {
    url: "mysql://user:password@host:3306/dbname",
  },
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

```shell
npx drizzle-kit pull --dialect=mysql --url=mysql://user:password@host:3306/dbname
npx drizzle-kit pull --dialect=mysql --tablesFilter=‘user*’ url=mysql://user:password@host:3306/dbname
```

![](https://orm.drizzle.team/_astro/introspect_mysql.Hk8acObY_Z20dDww.webp)
