---
url: https://orm.drizzle.team/docs/tutorials/drizzle-with-supabase
title: "Drizzle With Supabase"
description: ""
access_date: 2026-11-17T09:48:18.024Z
current_date: 2026-11-17T09:48:18.024Z
---

## Drizzle with Supabase Database

This tutorial demonstrates how to use Drizzle ORM with [Supabase Database](https://supabase.com/docs/guides/database/overview). Every Supabase project comes with a full [Postgres](https://www.postgresql.org/docs) database.

Upgrading an existing Drizzle project? Follow the [upgrade guide](../upgrade-v1.md) before continuing with this tutorial.

This guide assumes familiarity with:

- You should have installed Drizzle ORM and [Drizzle kit](../kit-overview.md). You can do this by running the following command:

```shell
npm i drizzle-orm@rc
npm i -D drizzle-kit@rc
```

- You should have installed `dotenv` package for managing environment variables. Read more about this package [here](https://www.npmjs.com/package/dotenv)

```shell
npm i dotenv
```

- You should have installed `postgres` package for connecting to the Postgres database. Read more about this package [here](https://www.npmjs.com/package/postgres)

```shell
npm i postgres
```

Check [Supabase documentation](https://supabase.com/docs/guides/database/connecting-to-postgres#connecting-with-drizzle) to learn how to connect to the database with Drizzle ORM.

## Setup Supabase and Drizzle ORM

#### Create a new Supabase project

You can create new Supabase project in the [dashboard](https://supabase.com/dashboard) or by following this [link](https://database.new/).

#### Setup connection string variable

Click **Connect** in your Supabase project dashboard and copy the **Direct connection** URI. If your network only supports IPv4, use the **Session pooler** URI instead. This connection is used by both Drizzle ORM and Drizzle Kit migrations. Replace the password placeholder with your database password.

Add `DATABASE_URL` variable to your `.env` or `.env.local` file.

```plaintext
DATABASE_URL=<YOUR_DATABASE_URL>
```

Read more about Connection Pooler and pooling modes in the [documentation](https://supabase.com/docs/guides/database/connecting-to-postgres#connection-pooler).

#### Connect Drizzle ORM to your database

Create a `index.ts` file in the `src/db` directory and set up your database configuration:

```typescript
import { config } from 'dotenv';
import { drizzle } from 'drizzle-orm/postgres-js';
import postgres from 'postgres';

config({ path: '.env' }); // or .env.local

const client = postgres(process.env.DATABASE_URL!);
export const db = drizzle({ client });
```

#### Create tables

Create a `schema.ts` file in the `src/db` directory and declare your tables:

```typescript
import { integer, pgTable, serial, text, timestamp } from 'drizzle-orm/pg-core';

export const usersTable = pgTable('users_table', {
  id: serial('id').primaryKey(),
  name: text('name').notNull(),
  age: integer('age').notNull(),
  email: text('email').notNull().unique(),
});

export const postsTable = pgTable('posts_table', {
  id: serial('id').primaryKey(),
  title: text('title').notNull(),
  content: text('content').notNull(),
  userId: integer('user_id')
    .notNull()
    .references(() => usersTable.id, { onDelete: 'cascade' }),
  createdAt: timestamp('created_at').notNull().defaultNow(),
  updatedAt: timestamp('updated_at')
    .notNull()
    .$onUpdate(() => new Date()),
});

export type InsertUser = typeof usersTable.$inferInsert;
export type SelectUser = typeof usersTable.$inferSelect;

export type InsertPost = typeof postsTable.$inferInsert;
export type SelectPost = typeof postsTable.$inferSelect;
```

#### Setup Drizzle config file

**Drizzle config** - a configuration file that is used by [Drizzle Kit](../kit-overview.md) and contains all the information about your database connection, migration folder and schema files.

Create a `drizzle.config.ts` file in the root of your project and add the following content:

```typescript
import { config } from 'dotenv';
import { defineConfig } from 'drizzle-kit';

config({ path: '.env' });

export default defineConfig({
  schema: './src/db/schema.ts',
  out: './drizzle',
  dialect: 'postgresql',
  dbCredentials: {
    url: process.env.DATABASE_URL!,
  },
});
```

#### Applying changes to the database

You can generate migrations using `drizzle-kit generate` command and then run them using the `drizzle-kit migrate` command.

Generate migrations:

```bash
npx drizzle-kit generate
```

These migrations are stored in the `drizzle` directory, as specified in your `drizzle.config.ts`. Each migration has its own timestamped folder containing a `migration.sql` file and a `snapshot.json` file.

Example of a generated migration:

```sql
CREATE TABLE "posts_table" (
    "id" serial PRIMARY KEY,
    "title" text NOT NULL,
    "content" text NOT NULL,
    "user_id" integer NOT NULL,
    "created_at" timestamp DEFAULT now() NOT NULL,
    "updated_at" timestamp NOT NULL
);
--> statement-breakpoint
CREATE TABLE "users_table" (
    "id" serial PRIMARY KEY,
    "name" text NOT NULL,
    "age" integer NOT NULL,
    "email" text NOT NULL UNIQUE
);
--> statement-breakpoint
ALTER TABLE "posts_table" ADD CONSTRAINT "posts_table_user_id_users_table_id_fkey" FOREIGN KEY ("user_id") REFERENCES "users_table"("id") ON DELETE CASCADE;
```

Run migrations:

```bash
npx drizzle-kit migrate
```

Learn more about the [migration process](../migrations.md). Drizzle Kit manages the migrations in this tutorial. To use Supabase as your migration runner instead, follow the [optional Supabase CLI workflow](#optional-manage-migrations-with-supabase-cli).

Alternatively, you can push changes directly to the database using [Drizzle kit push command](../kit-overview.md#prototyping-with-db-push):

```bash
npx drizzle-kit push
```

IMPORTANT

Push command is good for situations where you need to quickly test new schema designs or changes in a local development environment, allowing for fast iterations without the overhead of managing migration files.

## Optional: manage migrations with Supabase CLI

If you want Supabase to apply migrations, keep Drizzle Kit for generating SQL and use Supabase CLI as the migration runner. Choose this workflow before applying your first migration. The commands below use [Supabase CLI 3.x](https://supabase.com/docs/guides/cli/getting-started).

Initialize a local Supabase project if you have not already done so:

```bash
supabase init
```

Generate a migration from your Drizzle schema:

```bash
npx drizzle-kit generate
```

Drizzle Kit creates a folder containing `migration.sql` and `snapshot.json`. Supabase expects flat `<timestamp>_<name>.sql` files in `supabase/migrations` and skips subdirectories. Create a Supabase migration using the generated SQL:

```bash
supabase migration new create_tables < drizzle/20261116131610_moaning_turbo/migration.sql
```

Replace the example folder with the one produced by `drizzle-kit generate`. Repeat this export once for each new migration, in generation order. Keep both directories in version control: Drizzle uses its snapshots to generate subsequent changes; Supabase uses its SQL files and migration history to apply them.

Start your local Supabase stack with Docker running, then apply pending migrations:

```bash
supabase start
supabase migration up
```

To deploy to a hosted project with Supabase CLI 3.x, link the project and explicitly select the remote target:

```bash
supabase link --project-ref=<REFERENCE_ID>
supabase db push --remote
```

In this workflow, Supabase applies the SQL and owns the migration history. Use `drizzle-kit generate` for schema changes and export each new SQL file; use Supabase CLI to apply it. Do not also run `drizzle-kit migrate` or `drizzle-kit push` against the same database. If you have already applied migrations with Drizzle Kit, reconcile the existing schema and migration history before switching runners rather than replaying those migrations.

## Basic file structure

This is the basic file structure of the project. In the `src/db` directory, we have database-related files including connection in `index.ts` and schema definitions in `schema.ts`.

```plaintext
📦 <project root>
 ├ 📂 src
 │   ├ 📂 db
 │   │  ├ 📜 index.ts
 │   │  └ 📜 schema.ts
 ├ 📂 drizzle
 │   └ 📂 20261116131610_moaning_turbo
 │      ├ 📜 migration.sql
 │      └ 📜 snapshot.json
 ├ 📜 .env
 ├ 📜 drizzle.config.ts
 ├ 📜 package.json
 └ 📜 tsconfig.json
```

## Query examples

For instance, we create `src/db/queries` folder and separate files for each operation: insert, select, update, delete.

#### Insert data

Read more about insert query in the [documentation](../insert.md).

```typescript
import { db } from '../index';
import { InsertPost, InsertUser, postsTable, usersTable } from '../schema';

export async function createUser(data: InsertUser) {
  await db.insert(usersTable).values(data);
}

export async function createPost(data: InsertPost) {
  await db.insert(postsTable).values(data);
}
```

#### Select data

Read more about select query in the [documentation](../select.md).

```typescript
import { asc, between, count, eq, getColumns, sql } from 'drizzle-orm';
import { db } from '../index';
import { SelectUser, postsTable, usersTable } from '../schema';

export async function getUserById(id: SelectUser['id']): Promise<
  Array<{
    id: number;
    name: string;
    age: number;
    email: string;
  }>
> {
  return db.select().from(usersTable).where(eq(usersTable.id, id));
}

export async function getUsersWithPostsCount(
  page = 1,
  pageSize = 5,
): Promise<
  Array<{
    postsCount: number;
    id: number;
    name: string;
    age: number;
    email: string;
  }>
> {
  return db
    .select({
      ...getColumns(usersTable),
      postsCount: count(postsTable.id),
    })
    .from(usersTable)
    .leftJoin(postsTable, eq(usersTable.id, postsTable.userId))
    .groupBy(usersTable.id)
    .orderBy(asc(usersTable.id))
    .limit(pageSize)
    .offset((page - 1) * pageSize);
}

export async function getPostsForLast24Hours(
  page = 1,
  pageSize = 5,
): Promise<
  Array<{
    id: number;
    title: string;
  }>
> {
  return db
    .select({
      id: postsTable.id,
      title: postsTable.title,
    })
    .from(postsTable)
    .where(between(postsTable.createdAt, sql\`now() - interval '1 day'\`, sql\`now()\`))
    .orderBy(asc(postsTable.title), asc(postsTable.id))
    .limit(pageSize)
    .offset((page - 1) * pageSize);
}
```

Alternatively, you can use [relational query syntax](../rqb.md).

#### Update data

Read more about update query in the [documentation](../update.md).

```typescript
import { eq } from 'drizzle-orm';
import { db } from '../index';
import { SelectPost, postsTable } from '../schema';

export async function updatePost(id: SelectPost['id'], data: Partial<Omit<SelectPost, 'id'>>) {
  await db.update(postsTable).set(data).where(eq(postsTable.id, id));
}
```

#### Delete data

Read more about delete query in the [documentation](../delete.md).

```typescript
import { eq } from 'drizzle-orm';
import { db } from '../index';
import { SelectUser, usersTable } from '../schema';

export async function deleteUser(id: SelectUser['id']) {
  await db.delete(usersTable).where(eq(usersTable.id, id));
}
```
