---
url: https://orm.drizzle.team/docs/tutorials/railway-postgres-tailscale
title: "Railway Postgres Tailscale"
description: ""
access_date: 2026-08-27T16:48:15.483Z
current_date: 2026-08-27T16:48:15.483Z
---

## Connect Drizzle Kit to a private Railway database with Tailscale

This tutorial runs a PostgreSQL database on [Railway](https://driz.link/railway) with public access turned off, and connects `drizzle-kit` to it from your machine.

A [Tailscale](https://tailscale.com/) Forwarder joins your tailnet and forwards the database port into Railway’s private network. Nothing is exposed to the public internet, and every device on your tailnet can run migrations against the database.

This guide assumes the following:

- You have a [Railway](https://driz.link/railway) account.
- You have a [Tailscale](https://tailscale.com/) account.
- You have the Tailscale client installed on the machine you work from, and that machine is signed in to your tailnet. You can find the downloads on the [Tailscale website](https://tailscale.com/download).
- You have installed Drizzle ORM and [Drizzle Kit](../kit-overview.md).

```shell
npm i drizzle-orm@rc
npm i -D drizzle-kit@rc
```

## How it works

Railway gives every service a private domain that ends in `.railway.internal`. Those domains resolve only inside the project, so `drizzle-kit` on your machine cannot use them.

The Tailscale Forwarder solves that from both sides. It runs as a service in the same Railway project, so it resolves the private domains. It also joins your tailnet as a machine, so your own devices reach it by its tailnet name. Traffic goes from `drizzle-kit` over the tailnet to the forwarder, and from the forwarder to the database over Railway’s private network.

![](https://orm.drizzle.team/_astro/railway-postgres-tailscale-diagram-light.5GKar08z_Z2p4nFL.webp) ![](https://orm.drizzle.team/_astro/railway-postgres-tailscale-diagram-dark.BgsFEVKQ_200Dfn.webp)

## Set up the Railway project

#### Create a Railway project

Log in to your [Railway dashboard](https://railway.app/dashboard) and click the `New Project` button.

![](https://orm.drizzle.team/_astro/railway-create-service-menu.CGwVzlVj_ZHpEVP.webp)

#### Provision a PostgreSQL database

On the project canvas, click the `New` button in the top right corner and select `Database` → `PostgreSQL`. You can also use the command palette (`Cmd/Ctrl + K`) and search for PostgreSQL.

![](https://orm.drizzle.team/_astro/railway-select-postgres.CyJphHAC_ZIczs6.webp)

#### Keep the database on the private network

Click on the PostgreSQL service, go to the `Settings` tab, and find the `Networking` section. Verify that public access is disabled. Under `Private Networking` you will see the private domain of the service, for example `postgres.railway.internal`. The forwarder will use that domain.

![](https://orm.drizzle.team/_astro/railway-postgres-private-networking.2NwPiIFJ_Z1WT18O.webp)

## Add the database to your tailnet

#### Check MagicDNS and HTTPS certificates

Open the [DNS page](https://login.tailscale.com/admin/dns) in the Tailscale admin console. Both **MagicDNS** and **HTTPS Certificates** must be enabled. Each section offers the opposite action, so a button that reads `Disable MagicDNS...` means MagicDNS is already on.

MagicDNS gives the forwarder a stable machine name, which the database connection needs. HTTPS certificates are not needed for the database connection, but they are needed to open a browser UI over the forwarder, and both settings live on this page.

![](https://orm.drizzle.team/_astro/tailscale-dns-magicdns-https.B7RbkVw2_pKPF8.webp)

#### Create a reusable auth key

Go to the [Keys page](https://login.tailscale.com/admin/settings/keys) in the settings menu and click `Generate auth key`. Add a description and turn on `Reusable`. Leave the other settings as they are. A reusable key lets the forwarder register itself again when it starts without existing state.

![](https://orm.drizzle.team/_astro/tailscale-generate-auth-key.CXe6lSd__ZqEHR4.webp)

Click `Generate key`. Copy the key now, because the admin console shows it only once.

![](https://orm.drizzle.team/_astro/tailscale-auth-key-created.3CktNE2S_Z1aH9cK.webp)

#### Deploy the Tailscale Forwarder

The Tailscale Forwarder is a small TCP proxy. It joins your tailnet as a machine and forwards ports on that machine to services on the Railway private network. Railway publishes the template, and it runs the `brody192/tailscale-forwarder` image.

[![Deploy on Railway](https://railway.com/button.svg)](https://railway.com/deploy/tailscale-forwarder)

Open the `Deploy to` dropdown and select the project that holds your database. Both services must be in the same project, otherwise the forwarder won’t be able to reach the database over the private network, since it will be placed on a different private network.

Paste your auth key into `TS_AUTHKEY`. Then set `CONNECTION_MAPPING_01` to the database mapping. The format is `[https:]<source port>:<target host>:<target port>`.

```plaintext
5432:${{Postgres.RAILWAY_PRIVATE_DOMAIN}}:${{Postgres.PGPORT}}
```

This forwards port `5432` on the forwarder to your database.

![](https://orm.drizzle.team/_astro/railway-tailscale-forwarder-template-config.Dtw36ieS_Z2w6GtM.webp)

The template setup screen shows only the variables the template declares, and you cannot add new ones there. `CONNECTION_MAPPING_01` is the only mapping you can set now. You add further mappings on the service itself, after it exists.

#### Deploy and check the logs

Click `Deploy`. The template attaches a volume and sets `TS_EPHEMERAL=false`, so the machine keeps the same identity across redeploys instead of registering a duplicate.

Open the `Deployments` tab and read the deploy logs. Three lines tell you the forwarder started correctly:

- `tsnet starting with hostname "..."` gives the machine name on your tailnet. You need it in a later step.
- `Starting tailscale_fwdr` lists what it parsed, including `ts-ephemeral: false` and the source port, target address and target port of each mapping.
- `listening for connections` confirms the forwarder accepts traffic on that source port.

![](https://orm.drizzle.team/_astro/railway-tailscale-forwarder-logs.BzPtusak_1Q6z2n.webp)

#### Copy the forwarder domain

`TS_HOSTNAME` defaults to your project name, environment name and service name joined by dashes, for example `my-project-production-tailscale-forwarder`. The deploy log prints it on the `tsnet starting with hostname` line.

Open the [Machines page](https://login.tailscale.com/admin/machines) in the Tailscale admin console. The forwarder appears there on its own once it starts with a valid auth key.

![](https://orm.drizzle.team/_astro/tailscale-machines-forwarder.7qHmxVbZ_Z29ufOO.webp)

Click the machine to open its page. Under `Machine Details`, copy the `Full domain` value. That is the host you connect to from your machine.

![](https://orm.drizzle.team/_astro/tailscale-machine-full-domain.Dbyeapim_Z1USs6p.webp)

## Connect Drizzle Kit from your machine

IMPORTANT

The machine you run `drizzle-kit` on must be signed in to the same tailnet as the forwarder, because the database connection is routed through it.

#### Set the connection string

Open the PostgreSQL service in Railway, go to the `Variables` tab, and copy `DATABASE_URL`.

![](https://orm.drizzle.team/_astro/railway-postgres-database-url.BJ_Jex_d_29LfvM.webp)

Replace the `***.railway.internal` host with the [Tailscale forwarder’s domain](#copy-the-forwarder-domain) and keep the rest of the string. `railway.internal` cannot be resolved on a machine outside Railway, so you need to connect through the Tailscale forwarder. Put the result in a `.env` file.

```plaintext
DATABASE_URL=postgresql://postgres:password@<full-domain>:5432/railway
```

Tailscale encrypts the connection between your machine and the forwarder, and Railway encrypts the hop between the forwarder and the database. Railway’s PostgreSQL also accepts TLS through the forwarder, so most clients connect with their default settings.

#### Set up the Drizzle config file

Create a `drizzle.config.ts` file in the root of your project:

```typescript
import { defineConfig } from "drizzle-kit";

export default defineConfig({
  schema: "./src/schema.ts",
  out: "./migrations",
  dialect: "postgresql",
  dbCredentials: {
    url: process.env.DATABASE_URL!,
  },
});
```

#### Apply your schema

Generate migrations and apply them:

```bash
npx drizzle-kit generate
```
```bash
npx drizzle-kit migrate
```

For rapid prototyping you can push the schema directly with [drizzle-kit push](../kit-overview.md#prototyping-with-db-push). For production deployments, prefer the `generate` and `migrate` workflow, so you keep a versioned history of schema changes. See the [Drizzle migrations fundamentals](../migrations.md) page for details.

#### Check the result

Open the database in [Drizzle Studio](../drizzle-kit-studio.md):

```bash
npx drizzle-kit studio
```

It runs on your machine and reads the same `DATABASE_URL`, so it reaches the database through the forwarder. Your new tables are there.

![](https://orm.drizzle.team/_astro/drizzle-studio-posts-table.Bzl9irRq_Z13Lyt6.webp)

For occasional access you do not need Tailscale at all. The Railway CLI opens a temporary tunnel to a private database with `railway connect Postgres --tunnel-only`. A forwarder is the right tool when you need a permanent route, or access from every device on your tailnet.

## Next steps

The forwarder can carry more than one port. [“Drizzle Studio and Gateway on Railway behind Tailscale”](railway-studio-tailscale.md) tutorial deploys a database browser into the same Railway project and adds a second mapping for it, so you browse your tables from your tailnet without a public domain. That tutorial starts from an empty project, so with the forwarder already running you can go straight to the template deployment and the second mapping.

## Troubleshooting

The forwarder is not in the Machines list

**The forwarder is not in the Machines list.** The auth key expired, or a key that was not reusable had already been used. Generate a new reusable key, update `TS_AUTHKEY` on the forwarder service, and redeploy.

drizzle-kit cannot connect

**`drizzle-kit` cannot connect.** Run `tailscale status` and check that your device is signed in to the same tailnet. Check that the port in your connection string matches the source port in `CONNECTION_MAPPING_01`. Railway’s PostgreSQL accepts TLS through the forwarder, so you do not need to disable SSL.

The mapping variable is ignored

**The mapping variable is ignored.** Confirm the exact variable name on the forwarder service. The template declares `CONNECTION_MAPPING_01`, with a leading zero. Some older guides write `CONNECTION_MAPPING_1`. Use the form the config panel shows you, and keep the numbers unique.
