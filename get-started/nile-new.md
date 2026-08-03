---
url: https://orm.drizzle.team/docs/get-started/nile-new
title: "Nile New"
description: ""
access_date: 2026-08-03T19:00:22.305Z
current_date: 2026-08-03T19:00:22.305Z
---

## Get Started with Drizzle and Nile

This guide assumes familiarity with:

- **dotenv** - package for managing environment variables - [read here](https://www.npmjs.com/package/dotenv)
- **tsx** - package for running TypeScript files - [read here](https://tsx.hirok.io/)
- **Nile** - PostgreSQL re-engineered for multi-tenant apps - [read here](https://thenile.dev/)

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

#### Step 1 - Install postgres package

```shell
npm i drizzle-orm@rc pg dotenv
npm i -D drizzle-kit@rc tsx @types/pg
```

#### Step 2 - Setup connection variables

Create a `.env` file in the root of your project and add your database connection variable:

```plaintext
NILEDB_URL=
```

#### Step 3 - Connect Drizzle ORM to the database

Create a `index.ts` file in the `src` directory and initialize the connection:

```typescript
import 'dotenv/config';
import { drizzle } from 'drizzle-orm/node-postgres';

const db = drizzle(process.env.NILEDB_URL!);
```

multi-tenancy

Nile provides **virtual tenant databases**. When you query Nile, you can set the tenant context and Nile will direct your queries to the virtual database for this particular tenant. All queries sent with tenant context will apply to that tenant alone (i.e. `select * from table` will result records only for this tenant). To learn more about how to set tenant context with Drizzle, check the **[official Nile-Drizzle example](https://www.thenile.dev/docs/getting-started/languages/drizzle#72-tenantdb)**.

#### Step 4 - Create a table

Create a `schema.ts` file in the `src/db` directory and declare your tables. Since Nile is Postgres for multi-tenant apps, our schema includes a table for tenants and a todos table with a `tenant_id` column (we refer to those as tenant-aware tables):

```typescript
import { pgTable, uuid, text, timestamp, varchar, vector, boolean } from "drizzle-orm/pg-core"
import { sql } from "drizzle-orm"

export const tenantsTable = pgTable("tenants", {
    id: uuid().default(sql\`public.uuid_generate_v7()\`).primaryKey().notNull(),
    name: text(),
    created: timestamp({ mode: 'string' }).default(sql\`LOCALTIMESTAMP\`).notNull(),
    updated: timestamp({ mode: 'string' }).default(sql\`LOCALTIMESTAMP\`).notNull(),
    deleted: timestamp({ mode: 'string' }),
});

export const todos = pgTable("todos", {
    id: uuid().defaultRandom(),
    tenantId: uuid("tenant_id"),
    title: varchar({ length: 256 }),
    estimate: varchar({ length: 256 }),
    embedding: vector({ dimensions: 3 }),
    complete: boolean(),
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
  dialect: 'postgresql',
  dbCredentials: {
    url: process.env.NILEDB_URL!,
  },
});
```

#### Step 6 - Applying changes to the database

You can directly apply changes to your database using the `drizzle-kit push` command. This is a convenient method for quickly testing new schema designs or modifications in a local development environment, allowing for rapid iterations without the need to manage migration files:

```bash
npx drizzle-kit push
```

Read more about the push command in [documentation](../drizzle-kit-push.md).

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

Read more about migration process in [documentation](../kit-overview.md).

#### Step 7 - Seed and Query the database

Let’s **update** the `src/index.ts` file with queries to create, read, update, and delete tenants and todos.

```typescript
import 'dotenv/config';
import { drizzle } from 'drizzle-orm/node-postgres';
import { eq, sql } from 'drizzle-orm';
import { tenantsTable, todosTable } from './db/schema';
  
const db = drizzle(process.env.NILEDB_URL!);

async function main() {
  const tenant: typeof tenantsTable.$inferInsert = {
    name: 'AwesomeSauce Inc.',
  };

  await db.insert(tenantsTable).values(tenant);
  console.log('New tenant created!')

  const tenants = await db.select().from(tenantsTable);
  console.log('Getting all tenants from the database: ', tenants)

  const todo: typeof todosTable.$inferInsert = {
    tenantId: tenants[0].id,
    title: 'Update pitch deck with AI stuff'
  }

  await db.insert(todosTable).values(todo);
  console.log('New todo created!')

  const todos = await db.select().from(todosTable);
  console.log('Getting all todos from the database: ', todos)

  await db.execute(sql\`SET nile.tenant_id = '${sql.raw(tenants[0].id)}'\`);
  console.log("Set tenant context");

  // note the lack of tenant_id in the query
  const tenant_todos = await db.select().from(todosTable);
  console.log('Getting all todos from the tenant virtual database: ', tenant_todos)

  await db
    .update(todosTable)
    .set({
      complete: true,
    })
    .where(eq(todosTable.id, todo.id));
  console.log('Todo marked as done!')

  await db.delete(todosTable).where(eq(todosTable.id, todo.id));
  console.log('Todo deleted!')
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
