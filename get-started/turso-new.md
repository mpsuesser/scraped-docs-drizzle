---
url: https://orm.drizzle.team/docs/get-started/turso-new
title: "Turso New"
description: ""
access_date: 2026-08-03T19:00:22.305Z
current_date: 2026-08-03T19:00:22.305Z
---

## Get Started with Drizzle and Turso Cloud

This guide assumes familiarity with:

- **dotenv** - package for managing environment variables - [read here](https://www.npmjs.com/package/dotenv)
- **tsx** - package for running TypeScript files - [read here](https://tsx.hirok.io/)
- **turso** - SQLite for Production - [read here](https://turso.tech/)
- **libsql** - a fork of SQLite optimized for low query latency, making it suitable for global applications - [read here](https://docs.turso.tech/libsql)

#### Basic file structure

This is the basic file structure of the project. In the `src/db` directory, we have table definition in `schema.ts`. In `drizzle` folder there are sql migration file and snapshots.

```plaintext
📦 <project root>
 ├ 📂 drizzle
 ├ 📂 src
 │   ├ 📂 db
 │   │  └ 📜 schema.ts
 │   └ 📜 index.ts
 ├ 📜 .env
 ├ 📜 drizzle.config.ts
 ├ 📜 package.json
 └ 📜 tsconfig.json
```

#### Step 1 - Install required packages

```shell
npm i drizzle-orm@rc @libsql/client dotenv
npm i -D drizzle-kit@rc tsx
```

#### Step 2 - Setup connection variables

Create a `.env` file in the root of your project and add you Turso database url and auth token:

```plaintext
TURSO_DATABASE_URL=
TURSO_AUTH_TOKEN=
```

important

If you don’t know your `TURSO_DATABASE_URL` and `TURSO_AUTH_TOKEN` values, you can refer to the LibSQL Driver SDK tutorial. Check it out [here](https://docs.turso.tech/sdk/ts/quickstart), then return with all the values generated and added to the `.env` file

#### Step 3 - Connect Drizzle ORM to the database

Drizzle has native support for all @libsql/client driver variations:

|  |  |
| --- | --- |
| `@libsql/client` | defaults to `node` import, automatically changes to `web` if `target` or `platform` is set for bundler, e.g. `esbuild --platform=browser` |
| `@libsql/client/node` | `node` compatible module, supports `:memory:`, `file`, `wss`, `http` and `turso` connection protocols |
| `@libsql/client/web` | module for fullstack web frameworks like `next`, `nuxt`, `astro`, etc. |
| `@libsql/client/http` | module for `http` and `https` connection protocols |
| `@libsql/client/ws` | module for `ws` and `wss` connection protocols |
| `@libsql/client/sqlite3` | module for `:memory:` and `file` connection protocols |
| `@libsql/client-wasm` | Separate experimental package for WASM |

```typescript
import { drizzle } from 'drizzle-orm/libsql';

const db = drizzle({ connection: {
  url: process.env.DATABASE_URL, 
  authToken: process.env.DATABASE_AUTH_TOKEN 
}});
```

Create a `index.ts` file in the `src` directory and initialize the connection:

```typescript
import 'dotenv/config';
import { drizzle } from 'drizzle-orm/libsql';

// You can specify any property from the libsql connection options
const db = drizzle({ 
  connection: { 
    url: process.env.TURSO_DATABASE_URL!, 
    authToken: process.env.TURSO_AUTH_TOKEN!
  }
});
```

If you need to provide your existing driver:

```typescript
import 'dotenv/config';
import { drizzle } from 'drizzle-orm/libsql';
import { createClient } from '@libsql/client';

const client = createClient({ 
  url: process.env.TURSO_DATABASE_URL!, 
  authToken: process.env.TURSO_AUTH_TOKEN!
});
const db = drizzle({ client });
```

#### Step 4 - Create a table

Create a `schema.ts` file in the `src/db` directory and declare your table:

```typescript
import { int, sqliteTable, text } from "drizzle-orm/sqlite-core";

export const usersTable = sqliteTable("users_table", {
  id: int().primaryKey({ autoIncrement: true }),
  name: text().notNull(),
  age: int().notNull(),
  email: text().notNull().unique(),
});
```

#### Step 5 - Setup Drizzle config file

**Drizzle config** - a configuration file that is used by [Drizzle Kit](../kit-overview.md) and contains all the information about your database connection, migration folder and schema files.

Create a `drizzle.config.ts` file in the root of your project and add the following content:

```typescript
import 'dotenv/config';
import { defineConfig } from 'drizzle-kit';

export default defineConfig({
  out: './drizzle',
  schema: './src/db/schema.ts',
  dialect: 'turso',
  dbCredentials: {
    url: process.env.TURSO_DATABASE_URL,
    authToken: process.env.TURSO_AUTH_TOKEN,
  },
});
```

#### Step 6 - Applying changes to the database

You can directly apply changes to your database using the `drizzle-kit push` command. This is a convenient method for quickly testing new schema designs or modifications in a local development environment, allowing for rapid iterations without the need to manage migration files:

```bash
npx drizzle-kit push
```

Read more about the push command in [documentation](../sqlite/drizzle-kit-push.md).

Tips

Alternatively, you can generate migrations using the `drizzle-kit generate` command and then apply them using the `drizzle-kit migrate` command:

Generate migrations:

```bash
npx drizzle-kit generate
```

Apply migrations:

```bash
npx drizzle-kit migrate
```

Read more about migration process in [documentation](../sqlite/kit-overview.md).

#### Step 7 - Seed and Query the database

Let’s **update** the `src/index.ts` file with queries to create, read, update, and delete users

```typescript
import 'dotenv/config';
import { eq } from 'drizzle-orm';
import { drizzle } from 'drizzle-orm/libsql';
import { usersTable } from './db/schema';

async function main() {
  const db = drizzle({ 
    connection: { 
        url: process.env.TURSO_DATABASE_URL!, 
        authToken: process.env.TURSO_AUTH_TOKEN!
    }
  });

  const user: typeof usersTable.$inferInsert = {
    name: 'John',
    age: 30,
    email: 'john@example.com',
  };

  await db.insert(usersTable).values(user);
  console.log('New user created!')

  const users = await db.select().from(usersTable);
  console.log('Getting all users from the database: ', users)
  /*
  const users: {
    id: number;
    name: string;
    age: number;
    email: string;
  }[]
  */

  await db
    .update(usersTable)
    .set({
      age: 31,
    })
    .where(eq(usersTable.email, user.email));
  console.log('User info updated!')

  await db.delete(usersTable).where(eq(usersTable.email, user.email));
  console.log('User deleted!')
}

main();
```

#### Step 8 - Run index.ts file

To run any TypeScript files, you have several options, but let’s stick with one: using `tsx`

You’ve already installed `tsx`, so we can run our queries now

**Run `index.ts` script**

```shell
npx tsx src/index.ts
```

tips

We suggest using `bun` to run TypeScript files. With `bun`, such scripts can be executed without issues or additional settings, regardless of whether your project is configured with CommonJS (CJS), ECMAScript Modules (ESM), or any other module format. To run a script with `bun`, use the following command:

```bash
bun src/index.ts
```

If you don’t have bun installed, check the [Bun installation docs](https://bun.sh/docs/installation#installing)
