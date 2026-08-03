---
url: https://orm.drizzle.team/docs/cockroach/kit-custom-migrations
title: "Kit Custom Migrations"
description: ""
access_date: 2026-08-03T19:43:09.006Z
current_date: 2026-08-03T19:43:09.006Z
---

## Migrations with Drizzle Kit

This guide assumes familiarity with:

- Get started with Drizzle and `drizzle-kit` - [read here](https://orm.drizzle.team/docs/cockroach/get-started)
- Drizzle schema fundamentals - [read here](sql-schema-declaration.md)
- Database connection basics - [read here](connect-overview.md)
- Drizzle migrations fundamentals - [read here](migrations.md)
- Drizzle Kit [overview](kit-overview.md) and [config file](drizzle-config-file.md)
- `drizzle-kit generate` command - [read here](drizzle-kit-generate.md)
- `drizzle-kit migrate` command - [read here](drizzle-kit-migrate.md)

Drizzle lets you generate empty migration files to write your own custom SQL migrations for DDL alternations currently not supported by Drizzle Kit or data seeding, which you can then run with [`drizzle-kit migrate`](drizzle-kit-migrate.md) command.

```shell
drizzle-kit generate --custom --name=seed-users
```

```plaintext
📦 <project root>
 ├ 📂 drizzle
 │ ├ 📂 20242409125510_init_sql
 │ └ 📂 20242409135510_seed-users
 ├ 📂 src
 └ …
```
```sql
-- ./drizzle/20242409135510_seed-users.sql

INSERT INTO "users" ("name") VALUES('Dan');
INSERT INTO "users" ("name") VALUES('Andrew');
INSERT INTO "users" ("name") VALUES('Dandrew');
```

### Running JavaScript and TypeScript migrations

We will add ability to run custom JavaScript and TypeScript migration/seeding scripts in the upcoming release, you can follow [github discussion](https://github.com/drizzle-team/drizzle-orm/discussions/2832).
