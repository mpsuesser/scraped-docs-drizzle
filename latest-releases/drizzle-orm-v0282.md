---
url: https://orm.drizzle.team/docs/latest-releases/drizzle-orm-v0282
title: "Drizzle Orm V0282"
description: ""
access_date: 2026-08-03T19:38:26.356Z
current_date: 2026-08-03T19:38:26.356Z
---

## DrizzleORM v0.28.2 release

## The community contributions release 🎉

### Internal Features and Changes

- Added a set of tests for d1
- Fixed issues in internal documentation

### Fixes

- Resolved the issue of truncating timestamp milliseconds for MySQL
- Corrected the type of the `.get()` method for sqlite-based dialects ([#565](https://github.com/drizzle-team/drizzle-orm/issues/565))
- Rectified the sqlite-proxy bug that caused the query to execute twice

### New packages 🎉

Added a support for [Typebox](https://github.com/sinclairzx81/typebox) in [drizzle-typebox](../typebox.md) package.

Please check documentation page for more usage examples: /docs/typebox
