---
url: https://orm.drizzle.team/docs/cockroach/overview
title: "Overview"
description: ""
access_date: 2026-08-03T19:43:09.006Z
current_date: 2026-08-03T19:43:09.006Z
---

## Drizzle ORM

Drizzle ORM is a headless TypeScript ORM with a head. 🐲

> Drizzle is a good friend who’s there for you when necessary and doesn’t bother when you need some space.

It looks and feels simple, performs on day *1000* of your project,  
lets you do things your way, and is there when you need it.

**It’s the only ORM with both [relational](rqb.md) and [SQL-like](select.md) query APIs**, providing you the best of both worlds when it comes to accessing your relational data. Drizzle is lightweight, performant, typesafe, non-lactose, gluten-free, sober, flexible and **serverless-ready by design**. Drizzle is not just a library, it’s an experience. 🤩

[![Drizzle bestofjs](https://orm.drizzle.team/_astro/bestofjs.Dmfq7AUp_26yiDJ.webp)](https://bestofjs.org/projects/drizzle-orm)

## Headless ORM?

First and foremost, Drizzle is a library and a collection of complementary opt-in tools.

**ORM** stands for *object relational mapping*, and developers tend to call Django-like or Spring-like tools an ORM. We truly believe it’s a misconception based on legacy nomenclature, and we call them **data frameworks**.

WARNING

With data frameworks you have to build projects **around them** and not **with them**.

**Drizzle** lets you build your project the way you want, without interfering with your project or structure.

Using Drizzle you can define and manage database schemas in TypeScript, access your data in a SQL-like or relational way, and take advantage of opt-in tools to push your developer experience *through the roof*. 🤯

## Why SQL-like?

**If you know SQL, you know Drizzle.**

Other ORMs and data frameworks tend to deviate/abstract you away from SQL, which leads to a double learning curve: needing to know both SQL and the framework’s API.

Drizzle is the opposite. We embrace SQL and built Drizzle to be SQL-like at its core, so you can have zero to no learning curve and access to the full power of SQL.

We bring all the familiar **[SQL schema](sql-schema-declaration.md)**, **[queries](select.md)**, **[automatic migrations](migrations.md)** and **[one more thing](rqb.md)**. ✨

```typescript
// Access your data
await db
    .select()
    .from(countries)
    .leftJoin(cities, eq(cities.countryId, countries.id))
    .where(eq(countries.id, 10))
```

## Why not SQL-like?

We’re always striving for a perfectly balanced solution, and while SQL-like does cover 100% of the needs, there are certain common scenarios where you can query data in a better way.

We’ve built the **[Queries API](rqb.md)** for you, so you can fetch relational nested data from the database in the most convenient and performant way, and never think about joins and data mapping.

**Drizzle always outputs exactly 1 SQL query.** Feel free to use it with serverless databases and never worry about performance or roundtrip costs!

```ts
const result = await db._query.users.findMany({
    with: {
        posts: true
    },
});
```

## Serverless?

The best part is no part. **Drizzle has exactly 0 dependencies!**

![Drizzle is slim an Serverless ready](https://orm.drizzle.team/_astro/drizzle31kb.6Mn-oJyX_ZHNm12.webp)

Drizzle ORM is dialect-specific, slim, performant and serverless-ready **by design**.

We’ve spent a lot of time to make sure you have best-in-class SQL dialect support, including Postgres, MySQL, and others.

Drizzle operates natively through industry-standard database drivers. We support all major **[PostgreSQL](../get-started-postgresql.md)**, **[MySQL](../mysql/get-started-mysql.md)**, **[SQLite](../sqlite/get-started-sqlite.md)**, **[SingleStore](../singlestore/get-started-singlestore.md)**, **[MSSQL](../mssql/get-started-mssql.md)** or **[CockroachDB](get-started-cockroach.md)** drivers out there, and we’re adding new ones **[really fast](https://twitter.com/DrizzleORM/status/1653082492742647811?s=20)**.

## Welcome on board!

More and more companies are adopting Drizzle in production, experiencing immense benefits in both DX and performance.

**We’re always there to help, so don’t hesitate to reach out. We’ll gladly assist you in your Drizzle journey!**

We have an outstanding **[Discord community](https://driz.link/discord)** and welcome all builders to our **[Twitter](https://twitter.com/drizzleorm)**.

Now go build something awesome with Drizzle and your **[PostgreSQL](../get-started-postgresql.md)**, **[MySQL](../mysql/get-started-mysql.md)**, **[SQLite](../sqlite/get-started-sqlite.md)**, **[MSSQL](../mssql/get-started-mssql.md)** or **[CockroachDB](get-started-cockroach.md)** database. 🚀
