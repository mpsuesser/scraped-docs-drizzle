---
url: https://orm.drizzle.team/docs/mysql/drizzle-kit-export
title: "Drizzle Kit Export"
description: ""
access_date: 2026-08-03T18:54:19.022Z
current_date: 2026-08-03T18:54:19.022Z
---

## drizzle-kit export

This guide assumes familiarity with:

- Get started with Drizzle and `drizzle-kit` - [read here](https://orm.drizzle.team/docs/mysql/get-started)
- Drizzle schema fundamentals - [read here](sql-schema-declaration.md)
- Database connection basics - [read here](connect-overview.md)
- Drizzle migrations fundamentals - [read here](migrations.md)
- Drizzle Kit [overview](kit-overview.md) and [config file](drizzle-config-file.md)

`drizzle-kit export` lets you export SQL representation of Drizzle schema and print in console SQL DDL representation on it.

How it works under the hood?

Drizzle Kit `export` command triggers a sequence of events:

1. It will read through your Drizzle schema file(s) and compose a json snapshot of your schema
2. Based on the current json snapshot it will generate SQL DDL statements
3. Output SQL DDL statements to console

It’s designed to cover [codebase first](migrations.md) approach of managing Drizzle migrations. You can export the SQL representation of the Drizzle schema, allowing external tools like Atlas to handle all the migrations for you

`drizzle-kit export` command requires you to provide both `dialect` and `schema` path options, you can set them either via [drizzle.config.ts](drizzle-config-file.md) config file or via CLI options

With config file

As CLI options

```ts
// drizzle.config.ts
import { defineConfig } from "drizzle-kit";

export default defineConfig({
  dialect: "mysql",
  schema: "./src/schema.ts",
});
```
```shell
npx drizzle-kit export
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
npx drizzle-kit export --config=drizzle-dev.config.ts
npx drizzle-kit export --config=drizzle-prod.config.ts
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

### Extended list of available configurations

`drizzle-kit export` has a list of cli-only options

|  |  |
| --- | --- |
| `--sql` | generating SQL representation of Drizzle Schema |

By default, Drizzle Kit outputs SQL files, but in the future, we want to support different formats

```shell
npx drizzle-kit export --sql=true
```

---

We recommend configuring `drizzle-kit` through [drizzle.config.ts](drizzle-config-file.md) file, yet you can provide all configuration options through CLI if necessary, e.g. in CI/CD pipelines, etc.

|  |  |  |
| --- | --- | --- |
| `dialect` | `required` | Database dialect, one of `postgresql` `mysql` `sqlite` `turso` `singlestore` `mssql` `cockroach` |
| `schema` | `required` | Path to typescript schema file(s) or folder(s) with multiple schema files |
| `config` |  | Configuration file path, default is `drizzle.config.ts` |

### Example

Example of how to export drizzle schema to console with Drizzle schema located in `./src/schema.ts`

We will also place drizzle config file in the `configs` folder.

Let’s create config file:

```plaintext
📦 <project root>
 ├ 📂 configs
 │ └ 📜 drizzle.config.ts
 ├ 📂 src
 │ └ 📜 schema.ts
 └ …
```
```ts
import { defineConfig } from "drizzle-kit";

export default defineConfig({
  dialect: "mysql",
  schema: "./src/schema.ts",
});
```
```ts
import { mysqlTable, int, text } from 'drizzle-orm/mysql-core'

export const users = mysqlTable('users', {
    id: int('id').primaryKey().autoincrement(),
    email: text('email').notNull(),
    name: text('name')
});
```

Now let’s run

```shell
npx drizzle-kit export --config=./configs/drizzle.config.ts
```

And it will successfully output SQL representation of drizzle schema

```bash
CREATE TABLE "users" (
        \`id\` int AUTO_INCREMENT PRIMARY KEY,
        \`email\` varchar(255) NOT NULL,
        "name" text
);
```
