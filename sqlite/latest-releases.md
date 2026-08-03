---
url: https://orm.drizzle.team/docs/sqlite/latest-releases
title: "Latest Releases"
description: ""
access_date: 2026-08-03T19:08:17.353Z
current_date: 2026-08-03T19:08:17.353Z
---

Latest Releases[Drizzle ORM and Drizzle Kit v1.0.0-beta.2 release](../latest-releases/drizzle-orm-v1beta2.md)

[

Feb 12, 2025

v1 pre-release

Read more

](../latest-releases/drizzle-orm-v1beta2.md)[

Drizzle Kit v0.23.2 release

Aug 5, 2024

Bug fixes

Read more

](../latest-releases/drizzle-kit-v0232.md)[

DrizzleORM v0.32.2 release

Aug 5, 2024

Bug fixes

Read more

](../latest-releases/drizzle-orm-v0322.md)[

DrizzleORM v0.32.1 release

Jul 23, 2024

Bug fixes

Read more

](../latest-releases/drizzle-orm-v0321.md)[

DrizzleORM v0.32.0 release

Jul 10, 2024

Generated columns, identity columns, sequences and more

Read more

](../latest-releases/drizzle-orm-v0320.md)[

DrizzleORM v0.31.3 release

Jul 8, 2024

Prisma-Drizzle extension

Read more

](../latest-releases/drizzle-orm-v0313.md)[

DrizzleORM v0.31.4 release

Jul 8, 2024

Mark prisma clients package as optional

Read more

](../latest-releases/drizzle-orm-v0314.md)[

DrizzleORM v0.31.2 release

Jun 7, 2024

Added support for TiDB Cloud Serverless driver

Read more

](../latest-releases/drizzle-orm-v0312.md)[

DrizzleORM v0.31.1 release

Jun 4, 2024

React Live Queries 🎉

Read more

](../latest-releases/drizzle-orm-v0311.md)[

DrizzleORM v0.31.0 release

May 31, 2024

PostgreSQL indexes API changes

Read more

](../latest-releases/drizzle-orm-v0310.md)[

DrizzleORM v0.30.10 release

May 1, 2024

Added '.if()' function to all WHERE expressions and fixed internal mappings for sessions '.all', '.values', '.execute' functions in AWS DataAPI.

Read more

](../latest-releases/drizzle-orm-v03010.md)[

DrizzleORM v0.30.9 release

Apr 22, 2024

Added 'setWhere' and 'targetWhere' fields to '.onConflictDoUpdate()' config in SQLite, added schema information to Drizzle instances via 'db.\_.fullSchema' and fixed migrator in AWS Data API.

Read more

](../latest-releases/drizzle-orm-v0309.md)[

DrizzleORM v0.30.8 release

Apr 11, 2024

Added custom schema support to enums in Postgres, changed D1 'migrate()' function to use batch API, updated '.onConflictDoUpdate' method, fixed query generation for 'where' clause in Postgres '.onConflictDoNothing' method and fixed various bugs related to AWS Data API.

Read more

](../latest-releases/drizzle-orm-v0308.md)[

DrizzleORM v0.30.7 release

Apr 3, 2024

Added mappings for '@vercel/postgres' package and fixed interval mapping for neon drivers.

Read more

](../latest-releases/drizzle-orm-v0307.md)[

DrizzleORM v0.30.6 release

Mar 28, 2024

Added support for PGlite driver.

Read more

](../latest-releases/drizzle-orm-v0306.md)[

DrizzleORM v0.30.5 release

Mar 27, 2024

Added '$onUpdate' functionality for PostgreSQL, MySQL and SQLite and fixed insertions on columns with the smallserial datatype.

Read more

](../latest-releases/drizzle-orm-v0305.md)[

DrizzleORM v0.30.4 release

Mar 19, 2024

Added support for Xata driver.

Read more

](../latest-releases/drizzle-orm-v0304.md)[

DrizzleORM v0.30.3 release

Mar 19, 2024

Added raw query support to batch API in Neon HTTP driver, fixed '@neondatabase/serverless' HTTP driver types issue, and fixed sqlite-proxy driver '.run()' result.

Read more

](../latest-releases/drizzle-orm-v0303.md)[

DrizzleORM v0.30.2 release

Mar 14, 2024

Updated LibSQL migrations to utilize batch execution instead of transactions and fixed findFirst query for bun:sqlite.

Read more

](../latest-releases/drizzle-orm-v0302.md)[

DrizzleORM v0.30.1 release

Mar 8, 2024

Added support for op-sqlite driver and fixed migration hook for Expo driver.

Read more

](../latest-releases/drizzle-orm-v0301.md)[

DrizzleORM v0.30.0 release

Mar 7, 2024

Modified the 'postgres.js' driver instance to always return strings for dates, and then Drizzle will provide you with either strings of mapped dates, depending on the selected 'mode'. Fixed many bugs related to timestamps and dates.

Read more

](../latest-releases/drizzle-orm-v0300.md)[

DrizzleORM v0.29.5 release

Mar 6, 2024

Added with update, with delete, with insert, possibility to specify custom schema and custom name for migrations table, sqlite proxy batch and relational queries support.

Read more

](../latest-releases/drizzle-orm-v0295.md)[

DrizzleORM v0.29.4 release

Feb 22, 2024

Added Neon HTTP Batch support and updated the default behavior and instances of database-js.

Read more

](../latest-releases/drizzle-orm-v0294.md)[

DrizzleORM v0.29.3 release

Jan 2, 2024

Fixed expo peer dependencies.

Read more

](../latest-releases/drizzle-orm-v0293.md)[

DrizzleORM v0.29.2 release

Dec 25, 2023

Implemented enhancements in the planescale relational tests. Corrected string escaping for empty PgArrays. Rectified syntax error for the exists function in SQLite. Ensured proper handling of dates in AWS Data API. Resolved the Hermes mixins constructor issue.

Read more

](../latest-releases/drizzle-orm-v0292.md)[

DrizzleORM v0.29.1 release

Nov 29, 2023

Fixed issues include forwarding arguments correctly when using the withReplica feature, resolving the selectDistinctOn not working with multiple columns problem, and providing detailed JSDoc for all query builders in all dialects. Additionally, introduced new helpers for aggregate functions in SQL and a new ESLint Drizzle Plugin package.

Read more

](../latest-releases/drizzle-orm-v0291.md)[

DrizzleORM v0.29.0 release

Nov 9, 2023

Added new features like unsigned option for bigint in MySQL, improved query builder types, added possibility to specify name for primary keys and foreign keys, read replicas support, set operators support, new MySQL and PostgreSQL proxy drivers, D1 Batch API support.

Read more

](../latest-releases/drizzle-orm-v0290.md)[

DrizzleORM v0.28.6 release

Sep 6, 2023

Changed datetime mapping for MySQL, added LibSQL batch API support, added JSON mode for text in SQLite, added '.toSQL()' to Relational Query API calls, added new PostgreSQL operators for Arrays, added more SQL operators for the 'where' function in Relational Queries, and fixed bugs.

Read more

](../latest-releases/drizzle-orm-v0286.md)[

DrizzleORM v0.28.5 release

Aug 24, 2023

Fixed incorrect OpenTelemetry type import that caused a runtime error.

Read more

](../latest-releases/drizzle-orm-v0285.md)[

DrizzleORM v0.28.4 release

Aug 24, 2023

Fixed imports in ESM-based projects and type error on Postgres table definitions.

Read more

](../latest-releases/drizzle-orm-v0284.md)[

DrizzleORM v0.28.3 release

Aug 22, 2023

Added SQLite simplified query API, added methods to column builders and to table model type inference. Fixed sqlite-proxy and SQL.js response from '.get()' when the result is empty.

Read more

](../latest-releases/drizzle-orm-v0283.md)[

DrizzleORM v0.28.2 release

Aug 10, 2023

Added a set of tests for d1, fixed issues in internal documentation, resolved the issue of truncating timestamp milliseconds for MySQL, corrected the type of the '.get()' method for sqlite-based dialects, rectified the sqlite-proxy bug that caused the query to execute twice. Added a support for Typebox package.

Read more

](../latest-releases/drizzle-orm-v0282.md)[

DrizzleORM v0.28.1 release

Aug 7, 2023

Read more

](../latest-releases/drizzle-orm-v0281.md)[

DrizzleORM v0.28.0 release

Aug 6, 2023

Removed support for filtering by nested relations, Added Relational Queries mode config for mysql2 driver, improved IntelliSense performance for large schemas, improved Relational Queries Permormance and Read Usage. Added possibility to insert rows with default values for all columns.

Read more

](../latest-releases/drizzle-orm-v0280.md)[

DrizzleORM v0.27.2 release

Jul 12, 2023

Added support for unique constraints in PostgreSQL, MySQL, SQLite.

Read more

](../latest-releases/drizzle-orm-v0272.md)[

DrizzleORM v0.16.2 release

Jan 21, 2023

Drizzle ORM - is an idiomatic TypeScript ORM which can be used as query builder and as an ORM being the source of truth for SQL schema and CLI for automatic migrations generation.

Read more

](../latest-releases/drizzle-orm-v0162.md)[

DrizzleORM v0.11.0 release

Jul 20, 2022

DrizzleORM - is an open source TypeScript ORM, supports PostgreSQL and about to have MySQL and SQLite support in couple of weeks. We've decided it's time to share it with public.

Read more

](../latest-releases/drizzle-orm-v0110.md)
