---
url: https://orm.drizzle.team/docs/singlestore/upgrade-v1
title: "Upgrade V1"
description: ""
access_date: 2026-08-03T19:08:17.353Z
current_date: 2026-08-03T19:08:17.353Z
---

## Upgrading to Drizzle v1

```shell
npm i drizzle-orm@rc
npm i -D drizzle-kit@rc
```

#### Step 1 - Run drizzle-kit up

> v3 migration folder discussion: [https://github.com/drizzle-team/drizzle-orm/discussions/2832](https://github.com/drizzle-team/drizzle-orm/discussions/2832)

We’ve updated the migrations folder structure by:

- removing `journal.json`
- grouping SQL files and snapshots into separate migration folders
- removing the `drizzle-kit drop` command

These changes eliminate potential Git conflicts with the journal file and simplify the process of dropping or fixing conflicted migrations

The drizzle-kit has been architecturally redesigned:

- Migrated from database snapshots to DDL snapshots

Commutativity checks were added:

- Detecting non-commutative migrations across branches

> Commutativity discussion: [https://github.com/drizzle-team/drizzle-orm/discussions/5005](https://github.com/drizzle-team/drizzle-orm/discussions/5005)

To migrate from old structure to a new one you would need to run

```shell
npx drizzle-kit up
```

#### Step 2 - Review breaking changes

You can find out the full list of breaking changes [here](../v0-v1-changes.md#breaking-changes) and make sure to update your code accordingly

If you were using Relational Queries, you need to upgrade to v2:

- [How to migrate relations definition from v1 to v2](../relations-v1-v2.md#how-to-migrate-relations-schema-definition-from-v1-to-v2)
- [How to migrate queries from v1 to v2](../relations-v1-v2.md#how-to-migrate-queries-from-v1-to-v2)
