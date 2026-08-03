---
url: https://orm.drizzle.team/docs/cockroach/sequences
title: "Sequences"
description: ""
access_date: 2026-08-03T19:38:26.356Z
current_date: 2026-08-03T19:38:26.356Z
---

## Sequences

Sequences in CockroachDB and CockroachDB are special single-row tables created to generate unique identifiers, often used for auto-incrementing primary key values. They provide a thread-safe way to generate unique sequential values across multiple sessions.

```ts
import { cockroachSchema, cockroachSequence } from 'drizzle-orm/cockroach-core';

// No params specified
export const customSequence = cockroachSequence('name');

// Sequence with params
export const customSequence = cockroachSequence('name', {
  startWith: 100,
  maxValue: 10000,
  minValue: 100,
  cache: 10,
  increment: 2,
});

// Sequence in custom schema
export const customSchema = cockroachSchema('custom_schema');
export const customSequence = customSchema.sequence('name');
```
