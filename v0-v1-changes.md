---
url: https://orm.drizzle.team/docs/v0-v1-changes
title: "V0 V1 Changes"
description: ""
access_date: 2026-10-31T10:54:20.546Z
current_date: 2026-10-31T10:54:20.546Z
---

## Changes in v1

This page shows everything that changed from `drizzle-orm@0.x`, `drizzle-kit@0.x`, `drizzle-seed@0.x` to the latest `drizzle-orm@1.0`, `drizzle-kit@1.0`, `drizzle-seed@1.0`

Read more on how to upgrade [here](upgrade-v1.md)

## Breaking changes

#### Relational filters reject explicit undefined

In Relational Queries v2, omit an optional filter field or use `EmptyFilter` from `drizzle-orm/relations`. Passing `undefined` explicitly throws. An empty relation filter such as `where: { posts: {} }` is skipped. Use `posts: true` to require related rows and `posts: false` to require their absence.

See the [filter migration examples](relations-v1-v2.md#handle-optional-filters-explicitly), especially if you are upgrading from an earlier v1 release candidate.

#### Read replicas require explicit routing

With `withReplicas(primary, replicas)`, queries use the primary database by default. In 0.x, reads such as `select()` and `db.query` were routed to a replica automatically. Use `db.$replica` for reads you want to send to a replica:

```ts
// After: drizzle-orm@rc, with db created by withReplicas()
await db.select().from(users); // primary
await db.$replica.select().from(users); // replica
```

#### Query errors omit parameters by default

`DrizzleQueryError` no longer includes query parameters by default. Set `paramsInErrors: true` in your `drizzle()` configuration if you need them for debugging. This also includes parameter values in the error message, so consider what your error logs retain.

#### Relational Queries v1 removed

RQBv1 has been removed. Use [Relational Queries v2](relations.md) with the new `defineRelations()` API instead and new features. You can find more details about migrating from rqb v1 to v2 [here](relations-v1-v2.md)

A new relations system built around `defineRelations()`:

```ts
import { defineRelations } from 'drizzle-orm';
import * as schema from './schema';

const relations = defineRelations(schema, (r) => ({
  users: {
    posts: r.many.posts(),
  },
  posts: {
      from: r.posts.authorId,
      to: r.users.id,
    }),
  },
}));
```

#### Remove postgres-js JSON serialization workarounds

If you manually added `JSON.stringify()` around JSON values in raw SQL or direct `postgres-js` calls to work around Drizzle’s driver overrides, remove that workaround. The driver serializes JSON parameters itself, so keeping it can double-stringify the value. In a `sql` template, wrap a JSON array in `sql.param(jsonArrayValue)` so it is treated as one parameter rather than a SQL value list.

#### Relational queries use an optimized SQL shape

Raw SQL that refers directly to generated relation aliases may need updating. Prefer relation filters and ordering, or set `strategy: 'basic'` on the affected query. See [the migration example](relations-v1-v2.md#check-raw-sql-references-to-relations).

#### New Casing API

The legacy `drizzle({ casing: 'camelCase' })` has been replaced with a table/view/schema-level API:

```ts
import { snakeCase, camelCase } from 'drizzle-orm/pg-core';

export const users = snakeCase.table('users', {
  id: serial().primaryKey(),
  fullName: text(),        // → full_name in DB
  createdAt: timestamp(),  // → created_at in DB
});

// Available on all entity types:
snakeCase.table / snakeCase.view / snakeCase.materializedView / snakeCase.schema
camelCase.table / camelCase.view / camelCase.materializedView / camelCase.schema
```

#### Validator packages consolidated into drizzle-orm

IMPORTANT

You can still use drizzle-zod/valibot/typebox/arktype validation packages but all new update will be added to Drizzle ORM directly

- drizzle-zod -> [drizzle-orm/zod](zod.md)
- drizzle-valibot -> [drizzle-orm/valibot](valibot.md)
- drizzle-typebox -> [drizzle-orm/typebox-legacy](typebox-legacy.md) (using @sinclair/typebox)
- drizzle-typebox -> [drizzle-orm/typebox](typebox.md) (using typebox)
- drizzle-arktype -> [drizzle-orm/arktype](arktype.md)
- new `drizzle-orm/effect-schema` package added

#### .array() is no longer chainable

Use string syntax for multidimensional arrays:

```ts
// ❌ Before
column.array().array()

// ✅ After
column.array('[][]')
column.array('[][][]')
```

#### .enableRLS() deprecated

The `.enableRLS()` method on tables has been deprecated. Use `pgTable.withRLS()` instead:

```ts
// ❌ Before
const users = pgTable('users', { ... }).enableRLS();

// ✅ After
const users = pgTable.withRLS('users', { ... });
```

#### .generatedAlwaysAs() only accepts SQL

```ts
// ❌ Before — accepted raw values
.generatedAlwaysAs('some expression')

// ✅ After — only sql\`\` or () => sql\`\`
.generatedAlwaysAs(sql\`some expression\`)
.generatedAlwaysAs(() => sql\`some expression\`)
```

#### getTableColumns deprecation

Use `getColumns` instead

```ts
// ❌ Before
import { getTableColumns } from 'drizzle-orm';

const columns = getTableColumns(users);

// ✅ After
import { getColumns } from 'drizzle-orm';

const columns = getColumns(users);
```

#### New migration folder structure (v3)

**Linked discussion**: [#2832](https://github.com/drizzle-team/drizzle-orm/discussions/2832)

- No more `journal.json`
- SQL files and snapshots grouped into separate migration folders
- `drizzle-kit drop` command removed

These changes eliminate potential Git conflicts with the journal file and simplify the process of dropping or fixing conflicted migrations

Read more on how to upgrade [here](upgrade-v1.md)

#### schemaFilter default behavior changed

`drizzle-kit push` and `drizzle-kit pull` now manage **all schemas** by default, not just `public`. It also now supports glob patterns.

How it worked in 0.x versions

In 0.x, `drizzle-kit push` and `drizzle-kit pull` managed only tables in the `public` schema by default. You had to explicitly configure `schemaFilter` to manage other schemas.

```ts
// ❌ Before (0.x) — only \`public\` by default, explicit list only
export default defineConfig({
  schemaFilter: ['public', 'analytics'],
});

// ✅ After (1.0) — all schemas by default, glob patterns supported
export default defineConfig({
  // no schemaFilter needed if you want all schemas managed
});

// To limit to specific schemas, you can still use schemaFilter:
export default defineConfig({
  schemaFilter: ['public', 'app_*'], // glob patterns supported
});
```

#### drizzle-kit push --strict deprecation

The `--strict` flag has been removed — its behavior is now the default. `drizzle-kit push` always prompts for confirmation for data-loss statements unless the `--force` flag is passed. Use [drizzle-kit push —explain](drizzle-kit-push.md) to preview the SQL that would be executed before running it.

## Other features

#### Drizzle Kit for scripts and agents

If you use a coding agent for the upgrade, Drizzle Kit provides two ways to help it work with your project:

- The [Drizzle skill](https://orm.drizzle.team/docs/drizzle-kit-skills) gives your agent guidance on migration workflows, rename decisions and interpreting command results.
- The [MCP server](https://orm.drizzle.team/docs/drizzle-kit-mcp) lets your agent call your project’s Drizzle Kit to check migration history, generate migrations and read an existing database’s schema.

You can use them together, or use the skill with the CLI. After setup, try:

```text
Use the Drizzle skill to check this project's migration history.
Use the Drizzle MCP server if connected. Explain any conflicts or warnings
before changing files or applying anything to the database.
```

Use `--output json` with `generate`, `push`, `pull`, `up`, `export`, or `check` to receive a structured response. JSON mode is non-interactive. When a command needs a rename or data-loss decision, provide it through `--hints` or `--hints-file` and run the command again.

The [programmatic SDK](https://github.com/drizzle-team/drizzle-orm/blob/beta/drizzle-kit/SDK.md) exposes these operations from `drizzle-kit`, with `exportSql()` for `export`.

#### Relational query ordering and caching

Relational Queries v2 can order by related rows, including aggregates such as `orderBy: { posts: 'desc(count)' }`. Both object and callback forms of `orderBy` are supported. See [ordering by relations](relations-v1-v2.md#order-by-related-rows).

When a cache is configured with the `all` strategy, relational queries are cached by default. Set `withCache: false` on a query to opt out, or `withCache: true` to opt in with an explicit cache strategy.

#### JIT Mappers (opt-in)

Just-in-time compiled row mappers that make mapping as fast as the raw driver. You can find out more [here](jit-mappers.md)

```ts
const db = drizzle({ ..., jit: true });
```

#### Codecs

This is a driver-aware transform layer for normalizing requests/responses to/from DB. It resolves data-mapping inconsistencies between regular select, JSON object, and array contexts. You can find out more [here](codecs.md)

#### New MSSQL dialect

New MSSQL dialect support added. You can read more [here](mssql/get-started-mssql.md)

#### New CockroachDB dialect

New CockroachDB dialect support added. You can read more [here](cockroach/get-started-cockroach.md)

#### New NetlifyDB driver

New NetlifyDB driver support added. You can read more [here](connect-netlify-db.md)

Note: The Netlify Database driver is developed and maintained by the Netlify team.

#### Effect integration

- New **`@effect/sql-pg` driver support**. You can read more [here](connect-effect-postgres.md)
- **Effect Schema validation** via `drizzle-orm/effect-schema`. You can read more [here](effect-schema.md)

#### SQLcommenter support

Add custom tags to queries. Tags are appended as SQL comments at the end of each query. You can read more [here](sql-comments.md)

```ts
db.select().from(users).comment("my_first_tag");
// select "id", "name" from "users" /*my_first_tag*/
```

#### Column aliases via.as()

You can find out more aliasing info [here](aliases.md)

```ts
const query = db
  .select({ age: users.age.as('ageOfUser'), id: users.id.as('userId') })
  .from(users)
  .orderBy(asc(users.id.as('userId')));
```

#### Prepared queries

`.prepare(name)` name is now optional

#### Full drizzle-kit rewrite

The kit has been architecturally redesigned:

- Migrated from database snapshots to DDL snapshots
- Reworked diff detection and application
- Schema introspection reduced from ~10s to < 1s
- Added query hints and explain support for push

#### drizzle-kit pull --init

Creates the drizzle migration table and marks the first pulled migration as applied. You can read more [here](drizzle-kit-pull.md)

#### drizzle-kit push --explain

Shows the SQL that would be executed without actually running it. You can read more [here](drizzle-kit-push.md)

#### drizzle-kit check — commutativity check

Detects non-commutative migrations across branches — e.g., two branches altering the same column, or one renaming a table that another is altering.

How it works:

- Builds a DAG from snapshot `prevIds` to understand branch structure
- Finds fork points where branches diverged
- Computes the DDL diff from parent snapshot to each branch leaf
- Checks conflicts using a footprint map of which DDL statement types can interfere

If conflicts are found, the report tells you exactly which migrations on which branches are incompatible

#### PostgreSQL snapshots and migration conflicts

The initial PostgreSQL snapshot contains the full schema. Subsequent snapshots store structured diffs. Keep migration folders and their snapshots together in version control.

Remove `--ignore-conflicts` from your PostgreSQL scripts. If `generate` reports conflicting migration branches, preview the proposed repair:

```shell
drizzle-kit generate --fix-conflicts --explain
```

This shows the SQL and migration folders the repair would delete, without changing files. Review that output before running without `--explain`: the repair deletes the conflicting migrations and their children, then generates one replacement migration. Any database that ran a deleted migration must be reset.

#### Updates to the migration table

The migration table is automatically upgraded when you run `drizzle-kit up`. Two new columns are added:

- **`name`** — the full migration folder name (e.g. `20250220153045_brave_wolverine`)
- **`applied_at`** — timestamp of when the migration was executed (pre-existing rows are backfilled with `NULL`)

| Column | Type | v0 | v1 |
| --- | --- | --- | --- |
| `id` | serial | yes | yes |
| `hash` | text | yes | yes |
| `created_at` | bigint | yes | yes (legacy) |
| `name` | text | — | new |
| `applied_at` | timestamp with time zone | — | new |

Migrations are now matched by their full folder name instead of timestamps. Even if two migrations are generated within the same second, the name suffix guarantees uniqueness.

During the upgrade, existing rows are backfilled using a multi-step strategy:

1. **Millis match** — truncate stored millis to seconds, match against local folder timestamps
2. **Hash tiebreaker** — if multiple migrations share the same second, disambiguate via the SQL hash
3. **Hash-only fallback** — if millis matching fails entirely, match by hash alone

This means the upgrade handles all edge cases: normal single-developer projects, teams with closely-timed migrations, and even cases where the old journal data doesn’t perfectly align with the new folder names.

#### Migrator applies all missing migrations

Previously the migrator only looked for local migrations with a creation date later than the last applied one. Now it detects and applies **every missing migration**, regardless of timestamp ordering. All migrations are matched against the full folder name (14-digit UTC timestamp + name suffix).

#### Top-level await in drizzle config files

Top-level `await` is now supported in `drizzle.config.ts` and schema files on Node.js.

#### Drizzle-kit: Build system improvements

- Migrated from `esbuild-register` to `tsx` loader — seamless ESM and CJS support
- Native Bun and Deno launch support

#### TS Schema file processing

Only files with the following extensions are processed: `.js`, `.mjs`, `.cjs`, `.jsx`, `.ts`, `.mts`, `.cts`, `.tsx`. All other file types are ignored.
