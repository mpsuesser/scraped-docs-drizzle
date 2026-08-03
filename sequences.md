---
url: https://orm.drizzle.team/docs/sequences
title: "Sequences"
description: ""
access_date: 2026-08-03T19:08:17.353Z
current_date: 2026-08-03T19:08:17.353Z
---

## Sequences

Sequences in PostgreSQL are special single-row tables created to generate unique identifiers, often used for auto-incrementing primary key values. They provide a thread-safe way to generate unique sequential values across multiple sessions.

```ts
import { pgSchema, pgSequence } from "drizzle-orm/pg-core";

// No params specified
export const customSequence = pgSequence("name");

// Sequence with params
export const customSequence = pgSequence("name", {
      startWith: 100,
      maxValue: 10000,
      minValue: 100,
      cycle: true,
      cache: 10,
      increment: 2
});

// Sequence in custom schema
export const customSchema = pgSchema('custom_schema');
export const customSequence = customSchema.sequence("name");
```
