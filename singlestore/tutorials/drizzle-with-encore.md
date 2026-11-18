---
url: https://orm.drizzle.team/docs/singlestore/tutorials/drizzle-with-encore
title: "Drizzle With Encore"
description: ""
access_date: 2026-11-18T08:29:58.297Z
current_date: 2026-11-18T08:29:58.297Z
---

## Drizzle with Encore

Build a PostgreSQL API with the Drizzle ORM and Drizzle Kit v1 release candidate. Encore provisions the database and applies your migrations when the application starts.

- You need [Docker](https://docs.docker.com/get-started/get-docker/) running for the local PostgreSQL database.
- You should have the Encore CLI installed. This tutorial uses Encore 1.58.9, which supports Drizzle v1 migration folders with `source: "drizzle/v1"`. You can install it with:
```bash
# macOS
brew install encoredev/tap/encore

# Linux
curl -L https://encore.dev/install.sh | bash

# Windows
iwr https://encore.dev/install.ps1 | iex
```

## Setup Encore and Drizzle ORM

#### Create a new Encore project

Create an empty Encore TypeScript project:

```bash
encore app create my-app --example=ts/empty
cd my-app
```

Install Drizzle ORM, Drizzle Kit and the PostgreSQL driver:

```shell
npm i drizzle-orm@rc
npm i pg
npm i -D drizzle-kit@rc
npm i -D @types/pg
```

#### Create the database

Define your database in a `database.ts` file. Encore automatically provisions a PostgreSQL database locally using Docker and in the cloud when you deploy:

```typescript
import { SQLDatabase } from "encore.dev/storage/sqldb";
import { drizzle } from "drizzle-orm/node-postgres";

const db = new SQLDatabase("mydb", {
  migrations: {
    path: "migrations",
    source: "drizzle/v1",
  },
});

export const orm = drizzle(db.connectionString);
```

Setting `source: "drizzle/v1"` tells Encore to read each timestamped migration folder and its `migration.sql`. This example uses SQL queries, so the Drizzle client does not need a schema option.

#### Define your schema

Create a `schema.ts` file to define your tables:

```typescript
import * as p from "drizzle-orm/pg-core";

export const users = p.pgTable("users", {
  id: p.serial().primaryKey(),
  name: p.text().notNull(),
  email: p.text().unique().notNull(),
  createdAt: p.timestamp().defaultNow().notNull(),
});
```

#### Setup Drizzle config

Create a `drizzle.config.ts` file:

```typescript
import { defineConfig } from "drizzle-kit";

export default defineConfig({
  out: "migrations",
  schema: "schema.ts",
  dialect: "postgresql",
});
```

#### Generate migrations

Run Drizzle Kit to generate migrations from your schema:

```bash
npx drizzle-kit generate
```

This creates a timestamped folder under `migrations`, containing `migration.sql` and `snapshot.json`:

```text
migrations/
└── 20261117000000_initial/
    ├── migration.sql
    └── snapshot.json
```

The timestamp and generated name will differ. Keep both files. Encore applies the SQL, and Drizzle Kit uses the snapshot to generate subsequent changes.

#### Create an API endpoint

Use Drizzle in your Encore endpoints:

```typescript
import { api } from "encore.dev/api";
import { orm } from "./database";
import { users } from "./schema";

interface User {
  id: number;
  name: string;
  email: string;
}

export const list = api(
  { expose: true, method: "GET", path: "/users" },
  async (): Promise<{ users: User[] }> => {
    const result = await orm.select().from(users);
    return { users: result };
  }
);

export const create = api(
  { expose: true, method: "POST", path: "/users" },
  async (req: { name: string; email: string }): Promise<User> => {
    const [user] = await orm
      .insert(users)
      .values({ name: req.name, email: req.email })
      .returning();
    return user;
  }
);
```

#### Run your application

Start your Encore app:

```bash
encore run
```

Encore applies migrations when starting, so you do not need to run `drizzle-kit migrate` separately. After generating another migration while the app is running, restart `encore run` to apply it. Open [localhost:9400](http://localhost:9400/) to see the local dashboard with API docs, database explorer, and tracing.

## Learn more

- [Encore Drizzle Guide](https://encore.dev/docs/ts/develop/orms/drizzle)
- [Drizzle ORM Documentation](../../overview.md)
