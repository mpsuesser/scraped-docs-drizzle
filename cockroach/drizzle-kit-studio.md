---
url: https://orm.drizzle.team/docs/cockroach/drizzle-kit-studio
title: "Drizzle Kit Studio"
description: ""
access_date: 2026-08-21T14:34:37.254Z
current_date: 2026-08-21T14:34:37.254Z
---

## drizzle-kit studio

This guide assumes familiarity with:

- Drizzle Kit [overview](kit-overview.md) and [config file](drizzle-config-file.md)
- Drizzle Studio, our database browser - [read here](https://orm.drizzle.team/drizzle-studio/overview)

`drizzle-kit studio` command spins up a server for [Drizzle Studio](https://orm.drizzle.team/drizzle-studio/overview) hosted on [local.drizzle.studio](https://local.drizzle.studio/). It requires you to specify database connection credentials via [drizzle.config.ts](drizzle-config-file.md) config file.

By default it will start a Drizzle Studio server on `127.0.0.1:4983`

```ts
// drizzle.config.ts
import { defineConfig } from "drizzle-kit";

export default defineConfig({
  dialect: "cockroach",
  dbCredentials: {
    url: "postgresql://user:password@host:port/dbname"
  },
});
```
```shell
npx drizzle-kit studio
```

### Configuring host and port

By default Drizzle Studio server starts on `127.0.0.1:4983`, you can config `host` and `port` via CLI options

```shell
npx drizzle-kit studio --port=3000
npx drizzle-kit studio --host=0.0.0.0
npx drizzle-kit studio --host=0.0.0.0 --port=3000
```

### Logging

You can enable logging of every SQL statement by providing `verbose` flag

```shell
npx drizzle-kit studio --verbose
```

### Safari and Brave support

Safari and Brave block access to localhost by default. You need to install [mkcert](https://github.com/FiloSottile/mkcert) and generate self-signed certificate:

1. Follow the mkcert [installation steps](https://github.com/FiloSottile/mkcert#installation)
2. Run `mkcert -install`
3. Restart your `drizzle-kit studio`

### Other flavours of Drizzle Studio

Apart from `drizzle-kit studio` for local development, Drizzle Studio also comes as a self-hosted Drizzle Gateway, a Chrome extension for serverless database vendors and an embeddable component for your own product — check out the [Drizzle Studio overview](https://orm.drizzle.team/drizzle-studio/overview).

### Limitations

Our hosted version Drizzle Studio is meant to be used for local development and not meant to be used on remote (VPS, etc).

If you want to deploy Drizzle Studio to your VPS — that’s what [Drizzle Gateway](https://orm.drizzle.team/drizzle-studio/overview#drizzle-gateway) is for.

### Is it open source?

No. Drizzle ORM and Drizzle Kit are fully open sourced, while Studio is not.

Drizzle Studio for local development is free to use forever to enrich Drizzle ecosystem, open sourcing one would’ve break our ability to provide B2B offerings and monetise it, unfortunately.
