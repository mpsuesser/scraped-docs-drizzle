---
url: https://orm.drizzle.team/docs/singlestore/tutorials/drizzle-with-nile
title: "Drizzle With Nile"
description: ""
access_date: 2026-08-03T19:43:09.006Z
current_date: 2026-08-03T19:43:09.006Z
---

## Drizzle with Nile Database

This tutorial demonstrates how to use Drizzle ORM with [Nile Database](https://thenile.dev/). Nile is Postgres, re-engineered for multi-tenant applications.

This tutorial will demonstrate how to use Drizzle with Nile’s virtual tenant databases to developer a secure, scalable, multi-tenant application.

We’ll walk through building this example application step-by-step. If you want to peek at the complete example, you can take a look at its [Github repository](https://github.com/niledatabase/niledatabase/tree/main/examples/quickstart/drizzle).

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

- You should have installed `node-postgres` package for connecting to the Postgres database. Read more about this package [here](https://www.npmjs.com/package/node-postgres)

```shell
npm i node-postgres
```

- You should have installed `express` package for the web framework. Read more about express [here](https://expressjs.com/)

```shell
npm i express
```

- This guide uses [AsyncLocalStorage](https://nodejs.org/api/async_context.html) to manage the tenant context. If your framework or runtime does not support `AsyncLocalStorage`, you can refer to [Drizzle<>Nile](../../connect-nile.md) doc for alternative options.

## Setup Nile and Drizzle ORM

#### Signup to Nile and create a database

If you haven’t already, sign up to [Nile](https://console.thenile.dev/) and follow the app instructions to create a new database.

#### Get database connection string

On the left side-bar menu, select the “Settings” option, click on the Postgres logo, and click “generate credentials”. Copy the connection string and add it to the `.env` file in your project:

```plaintext
NILEDB_URL=postgres://youruser:yourpassword@us-west-2.db.thenile.dev:5432:5432/your_db_name
```

#### Connect Drizzle ORM to your database

Create a `db.ts` file in the `src/db` directory and set up your database configuration:

```typescript
import { drizzle } from 'drizzle-orm/node-postgres';
import dotenv from "dotenv/config";
import { sql } from "drizzle-orm";
import { AsyncLocalStorage } from "async_hooks";

export const db = drizzle(process.env.NILEDB_URL);
export const tenantContext = new AsyncLocalStorage<string | undefined>();

export function tenantDB<T>(cb: (tx: any) => T | Promise<T>): Promise<T> {
  return db.transaction(async (tx) => {
    const tenantId = tenantContext.getStore();
    console.log("executing query with tenant: " + tenantId);
    // if there's a tenant ID, set it in the transaction context
    if (tenantId) {
      await tx.execute(sql\`set local nile.tenant_id = '${sql.raw(tenantId)}'\`);
    }

    return cb(tx);
  }) as Promise<T>;
}
```

#### Setup Drizzle config file

**Drizzle config** - a configuration file that is used by [Drizzle Kit](../../kit-overview.md) and contains all the information about your database connection, migration folder and schema files.

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

#### Introspect Nile database

Nile databases have built-in tables. The most important of these is the `tenants` table, which is used to create and manage tenants. In order to use this table from our application, we’ll use Drizzle Kit CLI to generate a schema file that includes this schema.

```bash
npx drizzle-kit pull
```

The result of introspection will be a `schema.ts` file, `meta` folder with snapshots of your database schema, sql file with the migration and `relations.ts` file for [relational queries](../../rqb.md).

We recommend transferring the generated code from `drizzle/schema.ts` and `drizzle/relations.ts` to the actual schema file. In this guide we transferred code to `src/db/schema.ts`. Generated files for schema and relations can be deleted. This way you can manage your schema in a more structured way.

```plaintext
├ 📂 drizzle
│ ├ 📂 20242409125510_premium_mister_fear
│ │ ├ 📜 snapshot.json
│ │ └ 📜 migration.sql
│ ├ 📜 relations.ts ────────┐
│ └ 📜 schema.ts ───────────┤
├ 📂 src                    │ 
│ ├ 📂 db                   │
│ │ ├ 📜 relations.ts <─────┤
│ │ └ 📜 schema.ts <────────┘
│ └ 📜 index.ts         
└ …
```

Here is an example of the generated `schema.ts` file:

```typescript
// table schema generated by introspection
import { pgTable, uuid, text, timestamp, varchar, vector, boolean } from "drizzle-orm/pg-core"
import { sql } from "drizzle-orm"

export const tenants = pgTable("tenants", {
    id: uuid().default(sql\`public.uuid_generate_v7()\`).primaryKey().notNull(),
    name: text(),
    created: timestamp({ mode: 'string' }).default(sql\`LOCALTIMESTAMP\`).notNull(),
    updated: timestamp({ mode: 'string' }).default(sql\`LOCALTIMESTAMP\`).notNull(),
    deleted: timestamp({ mode: 'string' }),
});
```

#### Create additional tables

In addition to the built-in tables, our application will need some tables to store its data. We will add them to `src/db/schema.ts` that we previously generated, so this file will look like this:

```typescript
// table schema generated by introspection
import { pgTable, uuid, text, timestamp, varchar, vector, boolean } from "drizzle-orm/pg-core"
import { sql } from "drizzle-orm"

export const tenants = pgTable("tenants", {
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

#### Apply changes to the database

You can directly apply changes to your database using the `drizzle-kit push` command. This is a convenient method for quickly testing new schema designs or modifications in a local development environment, allowing for rapid iterations without the need to manage migration files:

```bash
npx drizzle-kit push
```

Read more about the push command in [documentation](../../drizzle-kit-push.md).

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

Read more about migration process in [documentation](../../kit-overview.md).

#### Initialize the webapp

Now that we have set up Drizzle to connect to Nile and we have our schema in place, we can use them in a multi-tenant web application. We are using Express as the web framework in this example, although Nile and Drizzle can be used from any web framework.

To keep the example simple, we’ll implement the webapp in a single file - `src/app.ts`. We’ll start by initializing the webapp:

```typescript
import express from "express";
import { tenantDB, tenantContext, db } from "./db/db";
import {
  tenants as tenantSchema,
  todos as todoSchema,
} from "./db/schema";
import { eq } from "drizzle-orm";

const PORT = process.env.PORT || 3001;

const app = express();
app.listen(PORT, () => console.log(\`Server is running on port ${PORT}\`));
app.use(express.json());
```

#### Initialize the tenant-aware middleware

Next, we’ll add middleware to the example. This middleware grabs the tenant ID from the path parameters and stores it in the `AsyncLocalStorage`. The `tenantDB` wrapper that we created in `src/db/index.ts` uses this tenant ID to set `nile.tenant_id` when executing queries, which then guarantees that the queries will execute against this tenant’s virtual database.

```typescript
// set the tenant ID in the context based on the URL parameter
app.use('/api/tenants/:tenantId/*', (req, res, next) => {
  const tenantId = req.params.tenantId;
  console.log("setting context to tenant: " + tenantId);
  tenantContext.run(tenantId, next);
});
```

The example gets the tenant ID from path parameter, but it is also common to set the tenant ID in a header such as `x-tenant-id` or in a cookie.

#### Add routes

Lastly, we need to add some routes for creating and listing tenants and todos. Note how we are using `tenantDB` wrapper to connect to the tenant’s virtual database. Also note how in `app.get("/api/tenants/:tenantId/todos"` we did not need to specify `where tenant_id=...` in the query. This is exactly because we are routed to that tenant’s database and the query cannot return data for any other tenant.

```typescript
// create new tenant
app.post("/api/tenants", async (req, res) => {
  try {
    const name = req.body.name;
    var tenants: any = null;
    tenants = await tenantDB(async (tx) => {
        return await tx.insert(tenantSchema).values({ name }).returning();
    });
    res.json(tenants);
  } catch (error: any) {
    console.log("error creating tenant: " + error.message);
    res.status(500).json({message: "Internal Server Error",});
  }
});

// return list of tenants
app.get("/api/tenants", async (req, res) => {
  let tenants: any = [];
  try {
      tenants = await tenantDB(async (tx) => {
        return await tx.select().from(tenantSchema);
      });
    res.json(tenants);
  } catch (error: any) {
    console.log("error listing tenants: " + error.message);
    res.status(500).json({message: "Internal Server Error",});
  }
});

// add new task for tenant
app.post("/api/tenants/:tenantId/todos", async (req, res) => {
  try {
    const { title, complete } = req.body;
    if (!title) {
      res.status(400).json({message: "No task title provided",});
    }
    const tenantId = req.params.tenantId;

    const newTodo = await tenantDB(async (tx) => {
      return await tx
        .insert(todoSchema)
        .values({ tenantId, title, complete })
        .returning();
    });
    // return without the embedding vector, since it is huge and useless
    res.json(newTodo);
  } catch (error: any) {
    console.log("error adding task: " + error.message);
    res.status(500).json({message: "Internal Server Error",});
  }
});

// update tasks for tenant
// No need for where clause because we have the tenant in the context
app.put("/api/tenants/:tenantId/todos", async (req, res) => {
  try {
    const { id, complete } = req.body;
    await tenantDB(async (tx) => {
      return await tx
        .update(todoSchema)
        .set({ complete })
        .where(eq(todoSchema.id, id));
    });
    res.sendStatus(200);
  } catch (error: any) {
    console.log("error updating tasks: " + error.message);
    res.status(500).json({message: "Internal Server Error",});
  }
});

// get all tasks for tenant
app.get("/api/tenants/:tenantId/todos", async (req, res) => {
  try {
    // No need for a "where" clause here because we are setting the tenant ID in the context
    const todos = await tenantDB(async (tx) => {
      return await tx
        .select({
          id: todoSchema.id,
          tenant_id: todoSchema.tenantId,
          title: todoSchema.title,
          estimate: todoSchema.estimate,
        })
        .from(todoSchema);
    });
    res.json(todos);
  } catch (error: any) {
    console.log("error listing tasks: " + error.message);
    res.status(500).json({message: error.message,});
  }
});
```

#### Try it out!

You can now run your new web application:

```bash
npx tsx src/app.ts
```

and use `curl` to try the routes you just created:

```bash
# create a tenant
curl --location --request POST 'localhost:3001/api/tenants' \
--header 'Content-Type: application/json' \
--data-raw '{"name":"my first customer"}'

# get tenants
curl  -X GET 'http://localhost:3001/api/tenants'

# create a todo (don't forget to use a real tenant-id in the URL)
curl  -X POST \
  'http://localhost:3001/api/tenants/108124a5-2e34-418a-9735-b93082e9fbf2/todos' \
  --header 'Content-Type: application/json' \
  --data-raw '{"title": "feed the cat", "complete": false}'

# list todos for tenant (don't forget to use a real tenant-id in the URL)
curl  -X GET \
  'http://localhost:3001/api/tenants/108124a5-2e34-418a-9735-b93082e9fbf2/todos'
```

## Project file structure

This is the file structure of the project. In the `src/db` directory, we have database-related files including connection in `db.ts` and schema definitions in `schema.ts`. The files generated by the migrations and introspections are in `./drizzle`

```plaintext
📦 <project root>
 ├ 📂 src
 │   ├ 📂 db
 │   │  ├ 📜 db.ts
 │   │  └ 📜 schema.ts
 │   └ 📜 app.ts
 ├ 📂 drizzle
 │   ├ 📂 meta
 │   │  ├ 📜 _journal.json
 │   │  └ 📜 0000_snapshot.json
 │   ├ 📜 relations.ts
 │   ├ 📜 schema.ts
 │   └ 📜 0000_watery_spencer_smythe.sql
 ├ 📜 .env
 ├ 📜 drizzle.config.ts
 └ 📜 package.json
```
