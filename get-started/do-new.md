---
url: https://orm.drizzle.team/docs/get-started/do-new
title: "Do New"
description: ""
access_date: 2026-08-03T19:43:09.006Z
current_date: 2026-08-03T19:43:09.006Z
---

## Get Started with Drizzle and SQLite Durable Objects

This guide assumes familiarity with:

- **dotenv** - package for managing environment variables - [read here](https://www.npmjs.com/package/dotenv)
- **tsx** - package for running TypeScript files - [read here](https://tsx.hirok.io/)
- **Cloudflare SQLite Durable Objects** - SQLite database embedded within a Durable Object - [read here](https://developers.cloudflare.com/durable-objects/api/sql-storage/)
- **wrangler** - Cloudflare Developer Platform command-line interface - [read here](https://developers.cloudflare.com/workers/wrangler)

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
npm i drizzle-orm@rc wrangler dotenv
npm i -D drizzle-kit@rc tsx
```

#### Step 2 - Setup wrangler.toml

You would need to have a `wrangler.toml` file for D1 database and will look something like this:

```toml
#:schema node_modules/wrangler/config-schema.json
name = "sqlite-durable-objects"
main = "src/index.ts"
compatibility_date = "2024-11-12"
compatibility_flags = [ "nodejs_compat" ]

# Bind a Durable Object. Durable objects are a scale-to-zero compute primitive based on the actor model.
# Durable Objects can live for as long as needed. Use these when you need a long-running "server", such as in realtime apps.
# Docs: https://developers.cloudflare.com/workers/wrangler/configuration/#durable-objects
[[durable_objects.bindings]]
name = "MY_DURABLE_OBJECT"
class_name = "MyDurableObject"

# Durable Object migrations.
# Docs: https://developers.cloudflare.com/workers/wrangler/configuration/#migrations
[[migrations]]
tag = "v1"
new_sqlite_classes = ["MyDurableObject"]

# We need rules so we can import migrations in the next steps
[[rules]] 
type = "Text"
globs = ["**/*.sql"]
fallthrough = true
```

#### Step 3 - Connect Drizzle ORM to the database

```ts
import { drizzle, type DrizzleSqliteDODatabase } from 'drizzle-orm/durable-sqlite';
import { DurableObject } from 'cloudflare:workers'

export class MyDurableObject extends DurableObject {
    storage: DurableObjectStorage;
    db: DrizzleSqliteDODatabase;

    constructor(ctx: DurableObjectState, env: Env) {
        super(ctx, env);
        this.storage = ctx.storage;
        this.db = drizzle(this.storage, { logger: false });
    }
}
```

#### Step 4 - Generate wrangler types

```shell
npx wrangler types
```

The output of this command will be a `worker-configuration.d.ts` file.

#### Step 5 - Create a table

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

#### Step 6 - Setup Drizzle config file

**Drizzle config** - a configuration file that is used by [Drizzle Kit](../kit-overview.md) and contains all the information about your database connection, migration folder and schema files.

Create a `drizzle.config.ts` file in the root of your project and add the following content:

```typescript
import 'dotenv/config';
import { defineConfig } from 'drizzle-kit';

export default defineConfig({
  out: './drizzle',
  schema: './src/db/schema.ts',
  dialect: 'sqlite',
  driver: 'durable-sqlite',
});
```

#### Step 7 - Applying changes to the database

Generate migrations:

```bash
npx drizzle-kit generate
```

You can apply migrations only from Cloudflare Workers. To achieve this, let’s define the migrate functionality in MyDurableObject:

```ts
import { drizzle, type DrizzleSqliteDODatabase } from 'drizzle-orm/durable-sqlite';
import { DurableObject } from 'cloudflare:workers'
import { migrate } from 'drizzle-orm/durable-sqlite/migrator';
import migrations from '../drizzle/migrations';

export class MyDurableObject extends DurableObject {
    storage: DurableObjectStorage;
    db: DrizzleSqliteDODatabase;

    constructor(ctx: DurableObjectState, env: Env) {
        super(ctx, env);
        this.storage = ctx.storage;
        this.db = drizzle(this.storage, { logger: false });
    }

    async migrate() {
        migrate(this.db, migrations);
    }
}
```

#### Step 8 - Migrate and Query the database

```typescript
import { drizzle, DrizzleSqliteDODatabase } from 'drizzle-orm/durable-sqlite';
import { DurableObject } from 'cloudflare:workers'
import { migrate } from 'drizzle-orm/durable-sqlite/migrator';
import migrations from '../drizzle/migrations';
import { usersTable } from './db/schema';

export class MyDurableObject extends DurableObject {
    storage: DurableObjectStorage;
    db: DrizzleSqliteDODatabase<any>;

    constructor(ctx: DurableObjectState, env: Env) {
        super(ctx, env);
        this.storage = ctx.storage;
        this.db = drizzle(this.storage, { logger: false });

        // Make sure all migrations complete before accepting queries.
        // Otherwise you will need to run \`this.migrate()\` in any function
        // that accesses the Drizzle database \`this.db\`.
        ctx.blockConcurrencyWhile(async () => {
            await this._migrate();
        });
    }

    async insertAndList(user: typeof usersTable.$inferInsert) {
        await this.insert(user);
        return this.select();
    }

    async insert(user: typeof usersTable.$inferInsert) {
        await this.db.insert(usersTable).values(user);
    }

    async select() {
        return this.db.select().from(usersTable);
    }

    async _migrate() {
        migrate(this.db, migrations);
    }
}

export default {
    /**
     * This is the standard fetch handler for a Cloudflare Worker
     *
     * @param request - The request submitted to the Worker from the client
     * @param env - The interface to reference bindings declared in wrangler.toml
     * @param ctx - The execution context of the Worker
     * @returns The response to be sent back to the client
     */
    async fetch(request: Request, env: Env): Promise<Response> {
        const id: DurableObjectId = env.MY_DURABLE_OBJECT.idFromName('durable-object');
        const stub = env.MY_DURABLE_OBJECT.get(id);

        // Option A - Maximum performance.
        // Prefer to bundle all the database interaction within a single Durable Object call
        // for maximum performance, since database access is fast within a DO.
        const usersAll = await stub.insertAndList({
            name: 'John',
            age: 30,
            email: 'john@example.com',
        });
        console.log('New user created. Getting all users from the database: ', users);

        // Option B - Slow but maybe useful sometimes for debugging.
        // You can also directly call individual Drizzle queries if they are exposed
        // but keep in mind every query is a round-trip to the Durable Object instance.
        await stub.insert({
            name: 'John',
            age: 30,
            email: 'john@example.com',
        });
        console.log('New user created!');
    
        const users = await stub.select();
        console.log('Getting all users from the database: ', users);

        return Response.json(users);
    }
}
```
