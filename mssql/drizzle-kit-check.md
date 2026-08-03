---
url: https://orm.drizzle.team/docs/mssql/drizzle-kit-check
title: "Drizzle Kit Check"
description: ""
access_date: 2026-08-03T19:00:22.305Z
current_date: 2026-08-03T19:00:22.305Z
---

## drizzle-kit check

This guide assumes familiarity with:

- Get started with Drizzle and `drizzle-kit` - [read here](https://orm.drizzle.team/docs/mssql/get-started)
- Drizzle schema fundamentals - [read here](sql-schema-declaration.md)
- Database connection basics - [read here](connect-overview.md)
- Drizzle migrations fundamentals - [read here](migrations.md)
- Drizzle Kit [overview](kit-overview.md) and [config file](drizzle-config-file.md)
- `drizzle-kit generate` command - [read here](drizzle-kit-generate.md)

`drizzle-kit check` command lets you check consistency of your generated SQL migrations history.

That’s extremely useful when you have multiple developers working on the project and altering database schema on different branches - read more about [migrations for teams](kit-migrations-for-teams.md).

---

`drizzle-kit check` command requires you to specify `dialect`, you can provide it either via [drizzle.config.ts](drizzle-config-file.md) config file or via CLI options

With config file

As CLI options

```ts
// drizzle.config.ts
import { defineConfig } from "drizzle-kit";

export default defineConfig({
  dialect: "mssql",
});
```
```shell
npx drizzle-kit check
```

### Multiple configuration files in one project

You can have multiple config files in the project, it’s very useful when you have multiple database stages or multiple databases on the same project:

```shell
npx drizzle-kit check --config=drizzle-dev.config.ts
npx drizzle-kit check --config=drizzle-prod.config.ts
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

### Ignore conflicts

In case you need `check` command to skip commutativity checks and bypass it, you can use `--ignore-conflicts`. If there is a situation you want to use it, then there is a big chance that `drizzle-kit` didn’t check migrations right and it’s a bug. Please report us your case, so we can fix it

```shell
drizzle-kit check --ignore-conflicts
```

### Extended list of configurations

We recommend configuring `drizzle-kit` through [drizzle.config.ts](drizzle-config-file.md) file, yet you can provide all configuration options through CLI if necessary, e.g. in CI/CD pipelines, etc.

|  |  |  |
| --- | --- | --- |
| `dialect` | `required` | Database dialect you are using. Can be one of `postgresql` `mysql` `sqlite` `turso` `singlestore` `mssql` `cockroach` |
| `out` |  | Migrations folder, default=`./drizzle` |
| `config` |  | Configuration file path, default= `drizzle.config.ts` |

```shell
npx drizzle-kit check --dialect=mssql
npx drizzle-kit check --dialect=mssql --out=./migrations-folder
```
