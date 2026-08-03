---
url: https://orm.drizzle.team/docs/mssql/kit-overview
title: "Kit Overview"
description: ""
access_date: 2026-08-03T19:43:09.006Z
current_date: 2026-08-03T19:43:09.006Z
---

## Migrations with Drizzle Kit

This guide assumes familiarity with:

- Get started with Drizzle and `drizzle-kit` - [read here](https://orm.drizzle.team/docs/mssql/get-started)
- Drizzle schema fundamentals - [read here](sql-schema-declaration.md)
- Database connection basics - [read here](connect-overview.md)
- Drizzle migrations fundamentals - [read here](migrations.md)

**Drizzle Kit** is a CLI tool for managing SQL database migrations with Drizzle.

```shell
npm i -D drizzle-kit
```

IMPORTANT

Make sure to first go through Drizzle [get started](https://orm.drizzle.team/docs/mssql/get-started) and [migration fundamentals](migrations.md) and pick SQL migration flow that suits your business needs best.

Based on your schema, Drizzle Kit let’s you generate and run SQL migration files, push schema directly to the database, pull schema from database, spin up drizzle studio and has a couple of utility commands.

```shell
npx drizzle-kit generate
npx drizzle-kit migrate
npx drizzle-kit push
npx drizzle-kit pull
npx drizzle-kit check
npx drizzle-kit up
npx drizzle-kit studio
npx drizzle-kit export
```

|  |  |
| --- | --- |
| [`drizzle-kit generate`](drizzle-kit-generate.md) | lets you generate SQL migration files based on your Drizzle schema either upon declaration or on subsequent changes, [see here](drizzle-kit-generate.md). |
| [`drizzle-kit migrate`](drizzle-kit-migrate.md) | lets you apply generated SQL migration files to your database, [see here](drizzle-kit-migrate.md). |
| [`drizzle-kit pull`](drizzle-kit-pull.md) | lets you pull(introspect) database schema, convert it to Drizzle schema and save it to your codebase, [see here](drizzle-kit-pull.md) |
| [`drizzle-kit push`](drizzle-kit-push.md) | lets you push your Drizzle schema to database either upon declaration or on subsequent schema changes, [see here](drizzle-kit-push.md) |
| [`drizzle-kit studio`](drizzle-kit-studio.md) | will connect to your database and spin up proxy server for Drizzle Studio which you can use for convenient database browsing, [see here](drizzle-kit-studio.md) |
| [`drizzle-kit check`](drizzle-kit-check.md) | will walk through all generate migrations and check for any race conditions(collisions) of generated migrations, [see here](drizzle-kit-check.md) |
| [`drizzle-kit up`](drizzle-kit-up.md) | used to upgrade snapshots of previously generated migrations, [see here](drizzle-kit-up.md) |
| [`drizzle-kit export`](drizzle-kit-export.md) | used to convert a TypeScript schema into raw SQL DDL and print it out [see here](drizzle-kit-export.md) |

Drizzle Kit is configured through [drizzle.config.ts](drizzle-config-file.md) configuration file or via CLI params.  
It’s required to at least provide SQL `dialect` and `schema` path for Drizzle Kit to know how to generate migrations.

```plaintext
📦 <project root>
 ├ 📂 drizzle
 ├ 📂 src
 ├ 📜 .env
 ├ 📜 drizzle.config.ts  <--- Drizzle config file
 ├ 📜 package.json
 └ 📜 tsconfig.json
```

```ts
import { defineConfig } from "drizzle-kit";

export default defineConfig({
  dialect: "mssql",
  schema: "./src/schema.ts",
});
```

You can provide Drizzle Kit config path via CLI param, it’s very useful when you have multiple database stages or multiple databases or different databases on the same project:

```shell
npx drizzle-kit push --config=drizzle-dev.drizzle.config
npx drizzle-kit push --config=drizzle-prod.drizzle.config
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
