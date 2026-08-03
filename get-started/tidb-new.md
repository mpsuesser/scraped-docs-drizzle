---
url: https://orm.drizzle.team/docs/get-started/tidb-new
title: "Tidb New"
description: ""
access_date: 2026-08-03T18:54:19.022Z
current_date: 2026-08-03T18:54:19.022Z
---

## Get Started with Drizzle and TiDB

This guide assumes familiarity with:

- **dotenv** - package for managing environment variables - [read here](https://www.npmjs.com/package/dotenv)
- **tsx** - package for running TypeScript files - [read here](https://tsx.hirok.io/)
- **TiDB** - The Distributed SQL Database by PingCAP - [read here](https://www.pingcap.com/)
- **serverless-js** - package for serverless and edge compute platforms that require HTTP external connections - [read here](https://github.com/tidbcloud/serverless-js)

important

For this tutorial, we will use the `@tidbcloud/serverless` driver to make **HTTP** calls. If you need to connect to TiDB through TCP, you can refer to our [MySQL Get Started](mysql-new.md) page

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

#### Step 1 - Install @tidbcloud/serverless package

```shell
npm i drizzle-orm@rc @tidbcloud/serverless dotenv
npm i -D drizzle-kit@rc tsx
```

#### Step 2 - Setup connection variables

Create a `.env` file in the root of your project and add your database connection variable:

```plaintext
DATABASE_URL=
```

#### Step 3 - Connect Drizzle ORM to the database

Create a `index.ts` file in the `src/db` directory and initialize the connection:

```typescript
import { drizzle } from 'drizzle-orm/tidb-serverless';

const db = drizzle(process.env.DATABASE_URL!);
```

If you need to provide your existing driver:

```typescript
import { connect } from '@tidbcloud/serverless';
import { drizzle } from 'drizzle-orm/tidb-serverless';

const client = connect({ url: process.env.DATABASE_URL! });
const db = drizzle({ client: client });
```

#### Step 4 - Create a table

Create a `schema.ts` file in the `src/db` directory and declare your table:

```typescript
import { int, mysqlTable, varchar } from 'drizzle-orm/mysql-core';

export const usersTable = mysqlTable('users_table', {
  id: int().primaryKey().autoincrement(),
  name: varchar({ length: 255 }).notNull(),
  age: int().notNull(),
  email: varchar({ length: 255 }).notNull().unique(),
});
```

#### Step 5 - Setup Drizzle config file

**Drizzle config** - a configuration file that is used by [Drizzle Kit](../mysql/kit-overview.md) and contains all the information about your database connection, migration folder and schema files.

Create a `drizzle.config.ts` file in the root of your project and add the following content:

```typescript
import 'dotenv/config';
import { defineConfig } from 'drizzle-kit';

export default defineConfig({
  out: './drizzle',
  schema: './src/db/schema.ts',
  dialect: 'mysql',
  dbCredentials: {
    url: process.env.DATABASE_URL!,
  },
});
```

#### Step 6 - Applying changes to the database

You can directly apply changes to your database using the `drizzle-kit push` command. This is a convenient method for quickly testing new schema designs or modifications in a local development environment, allowing for rapid iterations without the need to manage migration files:

```bash
npx drizzle-kit push
```

Read more about the push command in [documentation](../mysql/drizzle-kit-push.md).

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

Read more about migration process in [documentation](../mysql/kit-overview.md).

#### Step 7 - Seed and Query the database

Let’s **update** the `src/index.ts` file with queries to create, read, update, and delete users

```typescript
import 'dotenv/config';
import { drizzle } from 'drizzle-orm/tidb-serverless';
import { eq } from 'drizzle-orm';
import { usersTable } from './db/schema';
  
const db = drizzle(process.env.DATABASE_URL!);

async function main() {
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
