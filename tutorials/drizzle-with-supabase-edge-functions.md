---
url: https://orm.drizzle.team/docs/tutorials/drizzle-with-supabase-edge-functions
title: "Drizzle With Supabase Edge Functions"
description: ""
access_date: 2026-11-17T09:48:18.024Z
current_date: 2026-11-17T09:48:18.024Z
---

## Drizzle with Supabase Edge Functions

This tutorial demonstrates how to use Drizzle ORM with [Supabase Edge Functions](https://supabase.com/docs/guides/functions).

Upgrading an existing Drizzle project? Follow the [upgrade guide](../upgrade-v1.md) before continuing with this tutorial.

This guide assumes familiarity with:

- You should have the latest version of [Supabase CLI](https://supabase.com/docs/guides/cli/getting-started#installing-the-supabase-cli) installed.
- You should have installed Drizzle ORM and [Drizzle kit](../kit-overview.md). You can do this by running the following command:

```shell
npm i drizzle-orm@rc
npm i -D drizzle-kit@rc
```

- Install `dotenv` for the Drizzle config and `postgres` for the database connection:

```shell
npm i dotenv
npm i postgres
```

- You should have installed Docker Desktop. It is a prerequisite for local development. Follow the official [docs](https://docs.docker.com/desktop) to install.

To learn how to create a basic Edge Function on your local machine and then deploy it, see the [Edge Functions Quickstart](https://supabase.com/docs/guides/functions/quickstart).

#### Create a table

Create a `schema.ts` file in your `src` directory and declare a table schema:

```typescript
import { pgTable, serial, text, integer } from "drizzle-orm/pg-core";

export const usersTable = pgTable('users_table', {
  id: serial('id').primaryKey(),
  name: text('name').notNull(),
  age: integer('age').notNull()
})
```

This file will be used to generate migrations for your database.

#### Setup Drizzle config file

**Drizzle config** - a configuration file that is used by [Drizzle Kit](../kit-overview.md) and contains all the information about your database connection, migration folder and schema files.

Create a `drizzle.config.ts` file in the root of your project and add the following content:

```typescript
import "dotenv/config";
import { defineConfig } from "drizzle-kit";

export default defineConfig({
  schema: "./src/schema.ts",
  out: "./drizzle",
  dialect: "postgresql",
  dbCredentials: {
    url: process.env.DATABASE_URL!,
  },
});
```

In this tutorial we will use Drizzle kit to generate migrations for our schema.

#### Initialize a new Supabase project

Create a new Supabase project in a folder on your local machine:

```bash
supabase init
```

It will create `supabase` folder with `config.toml` file:

```text
└── supabase
    └── config.toml
```

If you are using Visual Studio Code, follow the [Supabase documentation](https://supabase.com/docs/guides/functions/local-development#deno-with-visual-studio-code) to setup settings for Deno.

#### Generate migrations

Run the `drizzle-kit generate` command to generate migrations:

```bash
npx drizzle-kit generate
```

These migrations are stored in the `drizzle` directory, as specified in your `drizzle.config.ts`. Each migration has its own timestamped folder containing a `migration.sql` file and a `snapshot.json` file.

#### Apply migrations

To start the Supabase local development stack, run the following command:

```bash
supabase start
```

Copy the local database connection string from `supabase status` into a `.env` file in your project root:

```text
DATABASE_URL=postgresql://postgres:postgres@127.0.0.1:54322/postgres
```

This example uses the default local port. Use the URL printed by your CLI if your configuration differs.

Apply the generated migrations with Drizzle Kit:

```bash
npx drizzle-kit migrate
```

Drizzle Kit manages this tutorial’s application tables. Supabase CLI manages the local services and Edge Functions. Keep Drizzle’s migration folders in `drizzle`. If you prefer Supabase-managed migrations, follow the [optional Supabase CLI workflow](drizzle-with-supabase.md#optional-manage-migrations-with-supabase-cli) instead of the Kit migration commands in this tutorial, including the hosted migration step below.

IMPORTANT

Don’t forget to run Docker

Learn more about the [Drizzle migration process](../migrations.md).

#### Create a new Edge Function

Run the `supabase functions new [FUNCTION_NAME]` command to create a new Edge Function:

```bash
supabase functions new drizzle-tutorial
```

It will create a new folder with the function name in the `supabase/functions` directory:

```text
└── supabase
    └── functions
    │   └── drizzle-tutorial
    │   │   ├── .npmrc ## Function-specific npm configuration (if needed)
    │   │   ├── deno.json ## Function-specific Deno configuration
    │   │   └── index.ts ## Your function code
```

When you create a new Edge Function, it will use TypeScript by default. However, it is possible write Edge Function in JavaScript. Learn more about it in the [documentation](https://supabase.com/docs/guides/functions/quickstart#not-using-typescript).

#### Setup imports

Add the following imports to the `deno.json` file in the `supabase/functions/drizzle-tutorial` directory:

```json
{
  "imports": {
    "drizzle-orm": "npm:drizzle-orm@1.0.0-rc.5",
    "postgres": "npm:postgres@3.4.7"
  }
}
```

Keep the ORM version in `deno.json` aligned with the version installed in your project. This example uses `drizzle-orm@1.0.0-rc.5`.

You can read more about managing dependencies [here](https://supabase.com/docs/guides/functions/dependencies#managing-dependencies).

#### Copy your schema to the functions directory

Copy the code that you will use in your edge function from `src/schema.ts` file to the `supabase/functions/drizzle-tutorial/index.ts` file:

```typescript
// Setup type definitions for built-in Supabase Runtime APIs
import "jsr:@supabase/functions-js/edge-runtime.d.ts"
import { pgTable, serial, text, integer } from "drizzle-orm/pg-core";

const usersTable = pgTable('users_table', {
  id: serial('id').primaryKey(),
  name: text('name').notNull(),
  age: integer('age').notNull()
})

Deno.serve(async (req) => {
  const { name } = await req.json()
  const data = {
    message: \`Hello ${name}!\`,
  }

  return new Response(
    JSON.stringify(data),
    { headers: { "Content-Type": "application/json" } },
  )
})
```

IMPORTANT

In the Deno ecosystem, each function should be treated as an independent project with its own set of dependencies and configurations. For these reasons, Supabase recommend maintaining separate configuration files (`deno.json`, `.npmrc`, or `import_map.json`) within each function’s directory, even if it means duplicating some configurations. Read more [here](https://supabase.com/docs/guides/functions/dependencies#managing-dependencies).

#### Connect Drizzle ORM to your database

Update your edge function code with your database configuration:

```typescript
// Setup type definitions for built-in Supabase Runtime APIs
import { integer, pgTable, serial, text } from "drizzle-orm/pg-core";
import { drizzle } from "drizzle-orm/postgres-js";
import "jsr:@supabase/functions-js/edge-runtime.d.ts";
import postgres from "postgres";

const usersTable = pgTable('users_table', {
  id: serial('id').primaryKey(),
  name: text('name').notNull(),
  age: integer('age').notNull()
})

Deno.serve(async () => {
  const connectionString = Deno.env.get("SUPABASE_DB_URL")!;

  // Disable prepared statements for transaction pooling
  const client = postgres(connectionString, { prepare: false });
  const db = drizzle({ client });

  await db.insert(usersTable).values({
    name: "Alice",
    age: 25
  })
  const data = await db.select().from(usersTable);

  return new Response(
    JSON.stringify(data)
  )
})
```

`SUPABASE_DB_URL` is default environment variable for the direct database connection. Learn more about managing environment variables in Supabase Edge Functions in the [documentation](https://supabase.com/docs/guides/functions/secrets).

#### Test your code locally

Run the following command to test your function locally:

```bash
supabase functions serve --no-verify-jwt
```

Navigate to the route `(e.g. /drizzle-tutorial)` in your browser:

```plaintext
[
  {
    "id": 1,
    "name": "Alice",
    "age": 25
  }
]
```

#### Link your local project to a hosted Supabase project

You can create new Supabase project in the [dashboard](https://supabase.com/dashboard) or by following this [link](https://database.new/).

Copy the `Reference ID` from project settings and use it to link your local development project to a hosted Supabase project by running the following command:

```bash
supabase link --project-ref=<REFERENCE_ID>
```

Click **Connect** in the hosted project’s dashboard and copy the **Direct connection** URI, or the **Session pooler** URI for an IPv4-only network. Update `DATABASE_URL` in your local `.env` file with that hosted database URI, replacing the password placeholder.

Apply the same Drizzle migrations to the hosted database:

```bash
npx drizzle-kit migrate
```

The `.env` file is used by Drizzle Kit on your machine. Configure the deployed function’s connection separately in the next step.

#### Setup environment variables

You can find `Project connect details` by clicking **Connect** in the top bar of the dashboard and copy the URI from the `Transaction pooler` section. Remember to replace the password placeholder with your actual database password.

Read more about Connection Pooler in the [documentation](https://supabase.com/docs/guides/database/connecting-to-postgres#connection-pooler).

Update your edge function code to use the `DATABASE_URL` environment variable instead of `SUPABASE_DB_URL`:

```typescript
// imports

// const connectionString = Deno.env.get("SUPABASE_DB_URL")!;
const connectionString = Deno.env.get("DATABASE_URL")!;

// code
```

Run the following command to set the environment variable:

```bash
supabase secrets set DATABASE_URL=<CONNECTION_STRING>
```

Learn more about managing environment variables in Supabase Edge Functions in the [documentation](https://supabase.com/docs/guides/functions/secrets).

#### Deploy your function

Deploy your function by running the following command:

```bash
supabase functions deploy drizzle-tutorial --no-verify-jwt
```

Finally, you can use URL of the deployed project and navigate to the route you created `(e.g. /drizzle-tutorial)` to access your edge function.
