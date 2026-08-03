---
url: https://orm.drizzle.team/docs/latest-releases/drizzle-orm-v0313
title: "Drizzle Orm V0313"
description: ""
access_date: 2026-08-03T19:08:17.353Z
current_date: 2026-08-03T19:08:17.353Z
---

## DrizzleORM v0.31.3 release

### Bug fixed

- 🛠️ Fixed RQB behavior for tables with same names in different schemas
- 🛠️ Fixed \[BUG\]: Mismatched type hints when using RDS Data API - #2097

### New Prisma-Drizzle extension

```ts
import { PrismaClient } from '@prisma/client';
import { drizzle } from 'drizzle-orm/prisma/pg';
import { User } from './drizzle';

const prisma = new PrismaClient().$extends(drizzle());
const users = await prisma.$drizzle.select().from(User);
```

For more info, check docs: /docs/prisma
