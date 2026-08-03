---
url: https://orm.drizzle.team/docs/cockroach/drizzle-config-file
title: "Drizzle Config File"
description: ""
access_date: 2026-08-03T19:43:09.006Z
current_date: 2026-08-03T19:43:09.006Z
---

## Drizzle Kit configuration file

This guide assumes familiarity with:

- Get started with Drizzle and `drizzle-kit` - [read here](https://orm.drizzle.team/docs/cockroach/get-started)
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
  dialect: "cockroach",
  schema: "./src/schema.ts",
  out: "./drizzle",
});
```

Example of an extended config file

```ts
import { defineConfig } from "drizzle-kit";

export default defineConfig({
  out: "./drizzle",
  dialect: "cockroach",
  schema: "./src/schema.ts",

  dbCredentials: {
    url: "./database/",
  },

  schemaFilter: "public",
  tablesFilter: "*",

  introspect: {
    casing: "camel",
  },

  migrations: {
    table: "__drizzle_migrations__",
    schema: "drizzle",
  },

  entities: {
    roles: {
      provider: '',
      exclude: [],
      include: []
    }
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
  dialect: "cockroach",
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
  dialect: "cockroach", 
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

Database connection credentials in a form of `url`, `user:password@host:port/db` params or exceptions drivers(`aws-data-api` `d1-http` `pglite` ) specific connection options.

|  |  |
| --- | --- |
| type | `postgresql` `mysql` `sqlite` `turso` `singlestore` `mssql` `cockroach` |
| default | — |
| commands | `push`, `pull`, `migrate`, `studio` |

```ts
import { defineConfig } from 'drizzle-kit'

export default defineConfig({
  dialect: "cockroach",
  dbCredentials: {
    url: "postgres://user:password@host:port/db",
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
  dialect: "cockroach",
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
import * as p from "drizzle-orm/cockroach-core"

export const users = p.cockroachTable("users", {
  id: p.int4(),
  firstName: p.string("first-name"),
  lastName: p.string("LastName"),
  email: p.string(),
  phoneNumber: p.string("phone_number"),
});
```
```sql
SELECT column_name as column_name, lower(crdb_sql_type) as data_type FROM information_schema.columns WHERE table_name = 'users';
```
```plaintext
column_name   | data_type        
---------------+------------------------
 id            | int4
 first-name    | string
 LastName      | string
 email         | string
 phone_number  | string
```

## \---

### tablesFilter

If you want to run multiple projects with one database - check out [our guide](goodies.md#multi-project-schema).

`drizzle-kit push` and `drizzle-kit pull` will manage all tables by default. You can configure list of tables, schemas and extensions via `tablesFilters`, `schemaFilter` and `extensionFilters` options.

`tablesFilter` option lets you specify [`glob`](https://www.digitalocean.com/community/tools/glob?comments=true&glob=/**/*.js&matches=false&tests=//%20This%20will%20match%20as%20it%20ends%20with%20%27.js%27&tests=/hello/world.js&tests=//%20This%20won%27t%20match!&tests=/test/some/globs) based table names filter, e.g. `["users", "user_info"]` or `"user*"`

|  |  |
| --- | --- |
| type | `string` `string[]` |
| default | — |
| commands | `push` `pull` |

```ts
import { defineConfig } from "drizzle-kit";

export default defineConfig({
  dialect: "cockroach",
  tablesFilter: ["users", "posts", "project1_*"],
});
```

### schemaFilter

If you want to run multiple projects with one database - check out [our guide](goodies.md#multi-project-schema).

`drizzle-kit push` and `drizzle-kit pull` will by default manage all schemas.

`schemaFilter` option lets you specify [`glob`](https://www.digitalocean.com/community/tools/glob?comments=true&glob=/**/*.js&matches=false&tests=//%20This%20will%20match%20as%20it%20ends%20with%20%27.js%27&tests=/hello/world.js&tests=//%20This%20won%27t%20match!&tests=/test/some/globs) based schema names filter, e.g. `["public", "auth"]` or `"tenant_*"`

|  |  |
| --- | --- |
| type | `string[]` |
| commands | `push` `pull` |

```ts
export default defineConfig({
  dialect: "cockroach",
  schemaFilter: ["public", "schema1", "schema2"],
});
```

## \---

### entities

This configuration is created to set up management settings for specific `entities` in the database.

For now, it only includes `roles`, but eventually all database entities will migrate here, such as `tables`, `schemas`, `extensions`, `functions`, `triggers`, etc

#### roles

If you are using Drizzle Kit to manage your schema and especially the defined roles, there may be situations where you have some roles that are not defined in the Drizzle schema. In such cases, you may want Drizzle Kit to skip those `roles` without the need to write each role in your Drizzle schema and mark it with `.existing()`.

The `roles` option lets you:

- Enable or disable role management with Drizzle Kit.
- Exclude specific roles from management by Drizzle Kit.
- Include specific roles for management by Drizzle Kit.
- Enable modes for providers like `Neon` and `Supabase`, which do not manage their specific roles.
- Combine all the options above

|  |  |
| --- | --- |
| type | `boolean \| { provider: "neon" \| "supabase", include: string[], exclude: string[] }` |
| default | `false` |
| commands | `push` `pull` |

By default, `drizzle-kit` won’t manage roles for you, so you will need to enable that. in `drizzle.config.ts`

```ts
export default defineConfig({
  dialect: "cockroach",
  entities: {
    roles: true
  }
});
```

**You have a role `admin` and want to exclude it from the list of manageable roles**

```ts
// drizzle.config.ts
import { defineConfig } from "drizzle-kit";

export default defineConfig({
  ...
  entities: {
    roles: {
      exclude: ['admin']
    }
  }
});
```

**You have a role `admin` and want to include to the list of manageable roles**

```ts
// drizzle.config.ts
import { defineConfig } from "drizzle-kit";

export default defineConfig({
  ...
  entities: {
    roles: {
      include: ['admin']
    }
  }
});
```

**If you are using `Neon` and want to exclude roles defined by `Neon`, you can use the provider option**

```ts
// drizzle.config.ts
import { defineConfig } from "drizzle-kit";

export default defineConfig({
  ...
  entities: {
    roles: {
      provider: 'neon'
    }
  }
});
```

**If you are using `Supabase` and want to exclude roles defined by `Supabase`, you can use the provider option**

```ts
// drizzle.config.ts
import { defineConfig } from "drizzle-kit";

export default defineConfig({
  ...
  entities: {
    roles: {
      provider: 'supabase'
    }
  }
});
```

important

You may encounter situations where Drizzle is slightly outdated compared to new roles specified by database providers, so you may need to use both the `provider` option and `exclude` additional roles. You can easily do this with Drizzle:

```ts
// drizzle.config.ts
import { defineConfig } from "drizzle-kit";

export default defineConfig({
  ...
  entities: {
    roles: {
      provider: 'supabase',
      exclude: ['new_supabase_role']
    }
  }
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
  dialect: "cockroach",
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
  dialect: "cockroach",
  breakpoints: false,
});
```
