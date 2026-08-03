---
url: https://orm.drizzle.team/docs/quick
title: "Quick"
description: ""
access_date: 2026-08-03T19:43:09.006Z
current_date: 2026-08-03T19:43:09.006Z
---

## Quick start

Lets build a quick start app with `PostgreSQL` + `postgresjs` and run our first migration.

The first thing we need to do is to install `drizzle-orm` and `drizzle-kit`:

```shell
npm i drizzle-orm@rc postgres
npm i -D drizzle-kit@rc
```

Lets declare our `schema.ts`:

```plaintext
📦 <project root>
 ├ ...
 ├ 📂 src
 │ └ 📜 schema.ts
 └ 📜 package.json
```
```ts
import { serial, text, timestamp, pgTable } from "drizzle-orm/pg-core";

export const user = pgTable("user", {
  id: serial("id"),
  name: text("name"),
  email: text("email"),
  password: text("password"),
  role: text("role").$type<"admin" | "customer">(),
  createdAt: timestamp("created_at"),
  updatedAt: timestamp("updated_at"),
});
```

Now lets add drizzle configuration file:

```plaintext
📦 <project root>
 ├ ...
 ├ 📂 src
 ├ 📜 drizzle.config.ts
 └ 📜 package.json
```
```ts
import { defineConfig } from "drizzle-kit";

export default defineConfig({
  dialect: "postgresql",
  schema: "./src/schema.ts",
  out: "./drizzle",
});
```

Add `generate` and `migrate` commands to `package.json` and run our first migrations generation:

```json
{
  "name": "first time?",
  "version": "0.0.1",
  "scripts": {
    "generate": "drizzle-kit generate",
    "migrate": "drizzle-kit migrate"
  }, 
}
```
```shell
$ npm run generate
...

[✓] Your SQL migration file ➜ drizzle/20242409125510_pale_mister_fear/migration.sql 🚀
```

Done! We now have our first SQL migration file 🥳

```plaintext
📦 <project root>
 ├ 📂 drizzle
 │ ├ 📂 20242409125510_pale_mister_fear
 ├ 📂 src
 ├ 📜 drizzle.config.ts
 └ 📜 package.json
```

Now lets run our first migration to the database:

```shell
$ npm run migrate
```

That’s it, folks!

**My personal congratulations 🎉**
