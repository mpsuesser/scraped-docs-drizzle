---
url: https://orm.drizzle.team/docs/singlestore/tutorials/drizzle-with-vercel-edge-functions
title: "Drizzle With Vercel Edge Functions"
description: ""
access_date: 2026-08-03T18:54:19.022Z
current_date: 2026-08-03T18:54:19.022Z
---

## Drizzle with Vercel Edge Functions

This tutorial demonstrates how to use Drizzle ORM with [Vercel Functions](https://vercel.com/docs/functions) in [Edge runtime](https://vercel.com/docs/functions/runtimes/edge-runtime).

This guide assumes familiarity with:

- You should have the latest version of [Vercel CLI](https://vercel.com/docs/cli#) installed.

```shell
npm i -g vercel
```

- You should have an existing Next.js project or create a new one using the following command:
```bash
npx create-next-app@latest --typescript
```
- You should have installed Drizzle ORM and [Drizzle kit](../../kit-overview.md). You can do this by running the following command:

```shell
npm i drizzle-orm@rc
npm i -D drizzle-kit@rc
```

IMPORTANT

In case you face the issue with resolving dependencies during installation:

If you’re not using React Native, forcing the installation with `--force` or `--legacy-peer-deps` should resolve the issue. If you are using React Native, then you need to use the exact version of React which is compatible with your React Native version.

## Edge-compatible driver

When using Drizzle ORM with Vercel Edge functions you have to use edge-compatible drivers because the functions run in [Edge runtime](https://vercel.com/docs/functions/runtimes/edge-runtime) not in Node.js runtime, so there are some limitations of standard Node.js APIs.

You can choose one of these drivers according to your database dialect:

- [Neon serverless driver](../../get-started-postgresql.md#neon) allows you to query your Neon Postgres databases from serverless and edge environments over HTTP or WebSockets in place of TCP. We recommend using this driver for connecting to `Neon Postgres`.
- [Vercel Postgres driver](../../get-started-postgresql.md#vercel-postgres) is built on top of the `Neon serverless driver`. We recommend using this driver for connecting to `Vercel Postgres`.
- [PlanetScale serverless driver](../../mysql/get-started-mysql.md#planetscale) allows you access any `MySQL` client and execute queries over an HTTP connection, which is generally not blocked by cloud providers.
- [libSQL client](../../sqlite/get-started-sqlite.md#turso) allows you to access [Turso](https://docs.turso.tech/introduction) database.
- Navigate directly to the [Neon Postgres](../../tutorials/drizzle-with-vercel-edge-functions.md#neon-postgres) section.
- Navigate directly to the [Vercel Postgres](../../tutorials/drizzle-with-vercel-edge-functions.md#vercel-postgres) section.
- Navigate directly to the [PlanetScale](../../tutorials/drizzle-with-vercel-edge-functions.md#planetscale) section.
- Navigate directly to the [Turso](../../tutorials/drizzle-with-vercel-edge-functions.md#turso) section.

### Neon Postgres

#### Install the @neondatabase/serverless driver

Install the `@neondatabase/serverless` driver:

```shell
npm i @neondatabase/serverless
```

#### Create a table

Create a `schema.ts` file in the `src/db` directory and declare a table schema:

```typescript
import { pgTable, serial, text } from "drizzle-orm/pg-core";

export const usersTable = pgTable('users_table', {
  id: serial('id').primaryKey(),
  name: text('name').notNull(),
  age: text('age').notNull(),
  email: text('email').notNull().unique(),
})
```

#### Setup Drizzle config file

**Drizzle config** - a configuration file that is used by [Drizzle Kit](../../kit-overview.md) and contains all the information about your database connection, migration folder and schema files.

Create a `drizzle.config.ts` file in the root of your project and add the following content:

```typescript
import { defineConfig } from "drizzle-kit";

export default defineConfig({
  schema: "./src/db/schema.ts",
  dialect: "postgresql",
  dbCredentials: {
    url: process.env.POSTGRES_URL!,
  },
});
```

Configure your database connection string in the `.env` file:

```plaintext
POSTGRES_URL="postgres://[user]:[password]@[host]-[region].aws.neon.tech:5432/[db-name]?sslmode=[ssl-mode]"
```

#### Applying changes to the database

You can generate migrations using `drizzle-kit generate` command and then run them using the `drizzle-kit migrate` command.

Generate migrations:

```bash
npx drizzle-kit generate
```

These migrations are stored in the `drizzle` directory, as specified in your `drizzle.config.ts`. This directory will contain the SQL files necessary to update your database schema and a `meta` folder for storing snapshots of the schema at different migration stages.

Example of a generated migration:

```sql
CREATE TABLE IF NOT EXISTS "users_table" (
    "id" serial PRIMARY KEY NOT NULL,
    "name" text NOT NULL,
    "age" text NOT NULL,
    "email" text NOT NULL,
    CONSTRAINT "users_table_email_unique" UNIQUE("email")
);
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

#### Connect Drizzle ORM to your database

Create a `index.ts` file in the `src/db` directory and set up your database configuration:

```typescript
import { drizzle } from 'drizzle-orm/neon-serverless';

export const db = drizzle(process.env.POSTGRES_URL!)
```

#### Create an API route

Create `route.ts` file in `src/app/api/hello` directory. To learn more about how to write a function, see the [Functions API Reference](https://vercel.com/docs/functions/functions-api-reference) and [Vercel Functions Quickstart](https://vercel.com/docs/functions/quickstart).

```ts
import { db } from "@/db";
import { usersTable } from "@/db/schema";
import { NextResponse } from "next/server";

export const dynamic = 'force-dynamic'; // static by default, unless reading the request
export const runtime = 'edge' // specify the runtime to be edge

export async function GET(request: Request) {
  const users = await db.select().from(usersTable)

  return NextResponse.json({ users, message: 'success' });
}
```

#### Test your code locally

Run the `next dev` command to start your local development server:

```bash
npx next dev
```

Navigate to the route you created `(e.g. /api/hello)` in your browser:

```plaintext
{
  "users": [],
  "message": "success"
}
```

#### Deploy your project

Create a new project in the [dashboard](https://vercel.com/new) or run the `vercel` command to deploy your project:

```bash
vercel
```

Add `POSTGRES_URL` environment variable:

```bash
vercel env add POSTGRES_URL
```

Redeploy your project to update your environment variables:

```bash
vercel
```

Finally, you can use URL of the deployed project and navigate to the route you created `(e.g. /api/hello)` to access your edge function.

### Vercel Postgres

You can check quickstart guide for Drizzle with Vercel Postgres client in the [documentation](../../get-started-postgresql.md#vercel-postgres).

#### Install the @vercel/postgres driver

Install the `@vercel/postgres` driver:

```shell
npm i @vercel/postgres
```

#### Create a table

Create a `schema.ts` file in the `src/db` directory and declare a table schema:

```typescript
import { pgTable, serial, text } from "drizzle-orm/pg-core";

export const usersTable = pgTable('users_table', {
  id: serial('id').primaryKey(),
  name: text('name').notNull(),
  age: text('age').notNull(),
  email: text('email').notNull().unique(),
})
```

#### Setup Drizzle config file

**Drizzle config** - a configuration file that is used by [Drizzle Kit](../../kit-overview.md) and contains all the information about your database connection, migration folder and schema files.

Create a `drizzle.config.ts` file in the root of your project and add the following content:

```typescript
import { defineConfig } from "drizzle-kit";

export default defineConfig({
  schema: "./src/db/schema.ts",
  dialect: "postgresql",
  dbCredentials: {
    url: process.env.POSTGRES_URL!,
  },
});
```

Configure your database connection string in the `.env` file:

```plaintext
POSTGRES_URL="postgres://[user]:[password]@[host]-[region].aws.neon.tech:5432/[db-name]?sslmode=[ssl-mode]"
```

#### Applying changes to the database

You can generate migrations using `drizzle-kit generate` command and then run them using the `drizzle-kit migrate` command.

Generate migrations:

```bash
npx drizzle-kit generate
```

These migrations are stored in the `drizzle` directory, as specified in your `drizzle.config.ts`. This directory will contain the SQL files necessary to update your database schema and a `meta` folder for storing snapshots of the schema at different migration stages.

Example of a generated migration:

```sql
CREATE TABLE IF NOT EXISTS "users_table" (
    "id" serial PRIMARY KEY NOT NULL,
    "name" text NOT NULL,
    "age" text NOT NULL,
    "email" text NOT NULL,
    CONSTRAINT "users_table_email_unique" UNIQUE("email")
);
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

#### Connect Drizzle ORM to your database

Create a `index.ts` file in the `src/db` directory and set up your database configuration:

```typescript
import { drizzle } from 'drizzle-orm/vercel-postgres';

export const db = drizzle()
```

#### Create an API route

Create `route.ts` in `src/app/api/hello` directory. To learn more about how to write a function, see the [Functions API Reference](https://vercel.com/docs/functions/functions-api-reference) and [Vercel Functions Quickstart](https://vercel.com/docs/functions/quickstart).

```ts
import { db } from "@/db";
import { usersTable } from "@/db/schema";
import { NextResponse } from "next/server";

export const dynamic = 'force-dynamic'; // static by default, unless reading the request
export const runtime = 'edge' // specify the runtime to be edge

export async function GET(request: Request) {
  const users = await db.select().from(usersTable)

  return NextResponse.json({ users, message: 'success' });
}
```

#### Test your code locally

Run the `next dev` command to start your local development server:

```bash
npx next dev
```

Navigate to the route you created `(e.g. /api/hello)` in your browser:

```plaintext
{
  "users": [],
  "message": "success"
}
```

#### Deploy your project

Create a new project in the [dashboard](https://vercel.com/new) or run the `vercel` command to deploy your project:

```bash
vercel
```

Add `POSTGRES_URL` environment variable:

```bash
vercel env add POSTGRES_URL
```

Redeploy your project to update your environment variables:

```bash
vercel
```

Finally, you can use URL of the deployed project and navigate to the route you created `(e.g. /api/hello)` to access your edge function.

### PlanetScale

In this tutorial we use [PlanetScale MySQL](https://planetscale.com/).

#### Install the @planetscale/database driver

Install the `@planetscale/database` driver:

```shell
npm i @planetscale/database
```

#### Create a table

Create a `schema.ts` file in the `src/db` directory and declare a table schema:

```typescript
import { mysqlTable, serial, text } from "drizzle-orm/mysql-core";

export const usersTable = mysqlTable('users_table', {
  id: serial('id').primaryKey(),
  name: text('name').notNull(),
  age: text('age').notNull(),
  email: text('email').notNull().unique(),
})
```

#### Setup Drizzle config file

**Drizzle config** - a configuration file that is used by [Drizzle Kit](../../kit-overview.md) and contains all the information about your database connection, migration folder and schema files.

Create a `drizzle.config.ts` file in the root of your project and add the following content:

```typescript
import { defineConfig } from "drizzle-kit";

export default defineConfig({
  schema: "./src/db/schema.ts",
  dialect: "mysql",
  dbCredentials: {
    url: process.env.MYSQL_URL!,
  },
});
```

Configure your database connection string in the `.env` file:

```plaintext
MYSQL_URL="mysql://[user]:[password]@[host].[region].psdb.cloud/[db-name]?ssl={'rejectUnauthorized':[ssl-rejectUnauthorized]}"
```

#### Applying changes to the database

You can generate migrations using `drizzle-kit generate` command and then run them using the `drizzle-kit migrate` command.

Generate migrations:

```bash
npx drizzle-kit generate
```

These migrations are stored in the `drizzle` directory, as specified in your `drizzle.config.ts`. This directory will contain the SQL files necessary to update your database schema and a `meta` folder for storing snapshots of the schema at different migration stages.

Example of a generated migration:

```sql
CREATE TABLE \`users_table\` (
    \`id\` serial AUTO_INCREMENT NOT NULL,
    \`name\` text NOT NULL,
    \`age\` text NOT NULL,
    \`email\` text NOT NULL,
    CONSTRAINT \`users_table_id\` PRIMARY KEY(\`id\`),
    CONSTRAINT \`users_table_email_unique\` UNIQUE(\`email\`)
);
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

#### Connect Drizzle ORM to your database

Create a `index.ts` file in the `src/db` directory and set up your database configuration:

```typescript
import { drizzle } from "drizzle-orm/planetscale-serverless";

export const db = drizzle(process.env.MYSQL_URL!)
```

#### Create an API route

Create `route.ts` in `src/app/api/hello` directory. To learn more about how to write a function, see the [Functions API Reference](https://vercel.com/docs/functions/functions-api-reference) and [Vercel Functions Quickstart](https://vercel.com/docs/functions/quickstart).

```ts
import { db } from "@/app/db/db";
import { usersTable } from "@/app/db/schema";
import { NextResponse } from "next/server";

export const dynamic = 'force-dynamic'; // static by default, unless reading the request
export const runtime = 'edge' // specify the runtime to be edge

export async function GET(request: Request) {
  const users = await db.select().from(usersTable)

  return NextResponse.json({ users, message: 'success' });
}
```

#### Test your code locally

Run the `next dev` command to start your local development server:

```bash
npx next dev
```

Navigate to the route you created `(e.g. /api/hello)` in your browser:

```plaintext
{
  "users": [],
  "message": "success"
}
```

#### Deploy your project

Create a new project in the [dashboard](https://vercel.com/new) or run the `vercel` command to deploy your project:

```bash
vercel
```

Add `MYSQL_URL` environment variable:

```bash
vercel env add MYSQL_URL
```

Redeploy your project to update your environment variables:

```bash
vercel
```

Finally, you can use URL of the deployed project and navigate to the route you created `(e.g. /api/hello)` to access your edge function.

### Turso

You can check [quickstart guide](../../sqlite/get-started-sqlite.md#turso) or [tutorial](../../tutorials/drizzle-with-turso.md) for Drizzle with Turso in the documentation.

#### Install the @libsql/client driver

Install the `@libsql/client` driver:

```shell
npm i @libsql/client
```

#### Create a table

Create a `schema.ts` file in the `src/db` directory and declare a table schema:

```typescript
import { integer, sqliteTable, text } from "drizzle-orm/sqlite-core";

export const usersTable = sqliteTable('users_table', {
  id: integer('id').primaryKey(),
  name: text('name').notNull(),
  age: text('age').notNull(),
  email: text('email').notNull().unique(),
})
```

#### Setup Drizzle config file

**Drizzle config** - a configuration file that is used by [Drizzle Kit](../../kit-overview.md) and contains all the information about your database connection, migration folder and schema files.

Create a `drizzle.config.ts` file in the root of your project and add the following content:

```typescript
import { defineConfig } from "drizzle-kit";

export default defineConfig({
  schema: "./src/db/schema.ts",
  dialect: "turso",
  dbCredentials: {
    url: process.env.TURSO_CONNECTION_URL!,
    authToken: process.env.TURSO_AUTH_TOKEN!,
  },
});
```

Configure your database connection string and auth token in the `.env` file:

```plaintext
TURSO_CONNECTION_URL="libsql://[db-name].turso.io"
TURSO_AUTH_TOKEN="[auth-token]"
```

#### Applying changes to the database

You can generate migrations using `drizzle-kit generate` command and then run them using the `drizzle-kit migrate` command.

Generate migrations:

```bash
npx drizzle-kit generate
```

These migrations are stored in the `drizzle` directory, as specified in your `drizzle.config.ts`. This directory will contain the SQL files necessary to update your database schema and a `meta` folder for storing snapshots of the schema at different migration stages.

Example of a generated migration:

```sql
CREATE TABLE \`users_table\` (
    \`id\` integer PRIMARY KEY NOT NULL,
    \`name\` text NOT NULL,
    \`age\` text NOT NULL,
    \`email\` text NOT NULL
);
--> statement-breakpoint
CREATE UNIQUE INDEX \`users_table_email_unique\` ON \`users_table\` (\`email\`);
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

#### Connect Drizzle ORM to your database

Create a `index.ts` file in the `src/db` directory and set up your database configuration:

```typescript
import { drizzle } from 'drizzle-orm/libsql';

export const db = drizzle({ connection: {
  url: process.env.TURSO_CONNECTION_URL!,
  authToken: process.env.TURSO_AUTH_TOKEN!,
}})
```

#### Create an API route

Create `route.ts` in `src/app/api/hello` directory. To learn more about how to write a function, see the [Functions API Reference](https://vercel.com/docs/functions/functions-api-reference) and [Vercel Functions Quickstart](https://vercel.com/docs/functions/quickstart).

```ts
import { db } from "@/app/db/db";
import { usersTable } from "@/app/db/schema";
import { NextResponse } from "next/server";

export const dynamic = 'force-dynamic'; // static by default, unless reading the request
export const runtime = 'edge' // specify the runtime to be edge

export async function GET(request: Request) {
  const users = await db.select().from(usersTable)

  return NextResponse.json({ users, message: 'success' });
}
```

#### Test your code locally

Run the `next dev` command to start your local development server:

```bash
npx next dev
```

Navigate to the route you created `(e.g. /api/hello)` in your browser:

```plaintext
{
  "users": [],
  "message": "success"
}
```

#### Deploy your project

Create a new project in the [dashboard](https://vercel.com/new) or run the `vercel` command to deploy your project:

```bash
vercel
```

Add `TURSO_CONNECTION_URL` environment variable:

```bash
vercel env add TURSO_CONNECTION_URL
```

Add `TURSO_AUTH_TOKEN` environment variable:

```bash
vercel env add TURSO_AUTH_TOKEN
```

Redeploy your project to update your environment variables:

```bash
vercel
```

Finally, you can use URL of the deployed project and navigate to the route you created `(e.g. /api/hello)` to access your edge function.
