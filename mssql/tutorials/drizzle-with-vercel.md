---
url: https://orm.drizzle.team/docs/mssql/tutorials/drizzle-with-vercel
title: "Drizzle With Vercel"
description: ""
access_date: 2026-08-03T19:00:22.305Z
current_date: 2026-08-03T19:00:22.305Z
---

## Drizzle with Vercel Postgres

This tutorial demonstrates how to use Drizzle ORM with [Vercel Postgres](https://vercel.com/docs/storage/vercel-postgres). Vercel Postgres is a serverless SQL database designed to integrate with Vercel Functions and your frontend framework.

This guide assumes familiarity with:

- You should have installed Drizzle ORM and [Drizzle kit](../../kit-overview.md). You can do this by running the following command:

```shell
npm i drizzle-orm@rc
npm i -D drizzle-kit@rc
```

- You should have installed `dotenv` package for managing environment variables. Read more about this package [here](https://www.npmjs.com/package/dotenv)

```shell
npm i dotenv
```

- You should have installed `@vercel/postgres` package. Read more about this package [here](https://www.npmjs.com/package/@vercel/postgres)

```shell
npm i @vercel/postgres
```

Check [Vercel documentation](https://vercel.com/docs/storage/vercel-postgres/using-an-orm#drizzle) to learn how to connect to the database with Drizzle ORM.

## Setup Vercel Postgres and Drizzle ORM

#### Create a new Vercel Postgres database

You can create new Vercel Postgres database in the [dashboard](https://vercel.com/dashboard).

Read Vercel Postgres [documentation](https://vercel.com/docs/storage/vercel-postgres/quickstart) to learn how to create a new database.

#### Setup connection string variable

Navigate to your Vercel Postgres database and copy `POSTGRES_URL` from `.env.local` section.

Add `POSTGRES_URL` to your `.env.local` or `.env` file.

```plaintext
POSTGRES_URL=<YOUR_DATABASE_URL>
```

#### Connect Drizzle ORM to your database

Create a `index.ts` file in the `src/db` directory and set up your database configuration:

```typescript
import { drizzle } from 'drizzle-orm/vercel-postgres';
import { config } from 'dotenv';

config({ path: '.env.local' }); // or .env

export const db = drizzle();
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

**Drizzle config** - a configuration file that is used by [Drizzle Kit](../../kit-overview.md) and contains all the information about your database connection, migration folder and schema files.

Create a `drizzle.config.ts` file in the root of your project and add the following content:

```typescript
import { config } from 'dotenv';
import { defineConfig } from 'drizzle-kit';

config({ path: '.env.local' });

export default defineConfig({
  schema: './src/db/schema.ts',
  out: './migrations',
  dialect: 'postgresql',
  dbCredentials: {
    url: process.env.POSTGRES_URL!,
  },
});
```

#### Applying changes to the database

You can generate migrations using `drizzle-kit generate` command and then run them using the `drizzle-kit migrate` command.

Generate migrations:

```bash
npx drizzle-kit generate
```

These migrations are stored in the `drizzle/migrations` directory, as specified in your `drizzle.config.ts`. This directory will contain the SQL files necessary to update your database schema and a `meta` folder for storing snapshots of the schema at different migration stages.

Example of a generated migration:

```sql
CREATE TABLE IF NOT EXISTS "posts_table" (
    "id" serial PRIMARY KEY NOT NULL,
    "title" text NOT NULL,
    "content" text NOT NULL,
    "user_id" integer NOT NULL,
    "created_at" timestamp DEFAULT now() NOT NULL,
    "updated_at" timestamp NOT NULL
);
--> statement-breakpoint
CREATE TABLE IF NOT EXISTS "users_table" (
    "id" serial PRIMARY KEY NOT NULL,
    "name" text NOT NULL,
    "age" integer NOT NULL,
    "email" text NOT NULL,
    CONSTRAINT "users_table_email_unique" UNIQUE("email")
);
--> statement-breakpoint
DO $$ BEGIN
 ALTER TABLE "posts_table" ADD CONSTRAINT "posts_table_user_id_users_table_id_fk" FOREIGN KEY ("user_id") REFERENCES "users_table"("id") ON DELETE cascade ON UPDATE no action;
EXCEPTION
 WHEN duplicate_object THEN null;
END $$;
```

Run migrations:

```bash
npx drizzle-kit migrate
```

Alternatively, you can push changes directly to the database using [Drizzle kit push command](../../kit-overview.md#prototyping-with-db-push):

```bash
npx drizzle-kit push
```

IMPORTANT

Push command is good for situations where you need to quickly test new schema designs or changes in a local development environment, allowing for fast iterations without the overhead of managing migration files.

## Basic file structure

This is the basic file structure of the project. In the `src/db` directory, we have database-related files including connection in `index.ts` and schema definitions in `schema.ts`.

```plaintext
📦 <project root>
 ├ 📂 src
 │   ├ 📂 db
 │   │  ├ 📜 index.ts
 │   │  └ 📜 schema.ts
 ├ 📂 migrations
 │   ├ 📂 meta
 │   │  ├ 📜 _journal.json
 │   │  └ 📜 0000_snapshot.json
 │   └ 📜 0000_watery_spencer_smythe.sql
 ├ 📜 .env.local
 ├ 📜 drizzle.config.ts
 ├ 📜 package.json
 └ 📜 tsconfig.json
```

## Query examples

For instance, we create `src/db/queries` folder and separate files for each operation: insert, select, update, delete.

#### Insert data

Read more about insert query in the [documentation](../../insert.md).

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

Read more about select query in the [documentation](../../select.md).

IMPORTANT

`getColumns` available starting from `drizzle-orm@1.0.0-beta.2` (read more [here](../../upgrade-v1.md))

If you are on pre-1 version(like `0.45.1`) then use `getTableColumns`

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

Alternatively, you can use [relational query syntax](../../rqb.md).

#### Update data

Read more about update query in the [documentation](../../update.md).

```typescript
import { eq } from 'drizzle-orm';
import { db } from '../index';
import { SelectPost, postsTable } from '../schema';

export async function updatePost(id: SelectPost['id'], data: Partial<Omit<SelectPost, 'id'>>) {
  await db.update(postsTable).set(data).where(eq(postsTable.id, id));
}
```

#### Delete data

Read more about delete query in the [documentation](../../delete.md).

```typescript
import { eq } from 'drizzle-orm';
import { db } from '../index';
import { SelectUser, usersTable } from '../schema';

export async function deleteUser(id: SelectUser['id']) {
  await db.delete(usersTable).where(eq(usersTable.id, id));
}
```
