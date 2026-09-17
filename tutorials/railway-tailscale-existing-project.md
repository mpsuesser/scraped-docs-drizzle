---
url: https://orm.drizzle.team/docs/tutorials/railway-tailscale-existing-project
title: "Railway Tailscale Existing Project"
description: ""
access_date: 2026-09-17T14:33:08.314Z
current_date: 2026-09-17T14:33:08.314Z
---

## Add Tailscale to Drizzle Studio or Gateway on Railway

Add a Tailscale service to a Railway project that runs Drizzle Studio or Drizzle Gateway, so that the database UI is reachable on your tailnet.

If you didn’t deploy Drizzle Studio or Gateway yet, follow [Drizzle Studio and Gateway on Railway with Tailscale](railway-studio-tailscale.md).

This guide assumes the following:

- Your project runs a service named `Drizzle Studio` or `Drizzle Gateway`.
- You have a [Tailscale](https://tailscale.com/) account.
- You have the Tailscale client installed on the machine you work from, and that machine is signed in to your tailnet. You can find the downloads on the [Tailscale website](https://tailscale.com/download).
- MagicDNS is on in your tailnet, because the address and the machine name depend on it. Tailnets created on or after 2022-10-20 have it on by default. If your tailnet is older, check the [DNS page](https://login.tailscale.com/admin/dns).

## How it works

This template adds a Tailscale service to your project. It joins your tailnet as its own machine and forwards a port on that machine to the database UI over Railway’s private network. Traffic goes from your device over the tailnet to that machine, and from there to the database UI.

## Add the Tailscale service

Pick the tab for your database UI.

Drizzle Studio

Drizzle Gateway

#### Open the template

Press the button below to open the Tailscale for Drizzle template.

**Don’t deploy it right away.** Set the variables first.

[![Deploy on Railway](https://railway.com/button.svg)](https://railway.com/new/template/tailscale-for-drizzle?referralCode=eobqmj&utm_medium=integration&utm_source=button&utm_campaign=tailscale-for-drizzle)

Open the `Deploy to` dropdown and select the project that holds your Drizzle Studio service.

![Tailscale for Drizzle deploy form with TS_FORWARD_BROWSER set to a reference to the Drizzle Studio service](https://orm.drizzle.team/_astro/railway-tailscale-sidecar-studio-form.BgUz0F6Y_KbMW2.webp)

#### Set the variables

| Variable | What to set |
| --- | --- |
| `TS_FORWARD_BROWSER` | The field is empty. Pick `RAILWAY_PRIVATE_DOMAIN` of your Studio service from the list. The field then reads `${{"Drizzle Studio".RAILWAY_PRIVATE_DOMAIN}}`. |
| `TS_FORWARD_DATABASE` | Optional. If you also want to reach the database itself from your tailnet, for example with `drizzle-kit`, pick `RAILWAY_PRIVATE_DOMAIN` of your database service from the list. Port 5432 on the machine then forwards to your database over Railway’s private network. You can also set it later on the Tailscale service’s `Variables` tab. |
| `TS_HOSTNAME` | Change it to `drizzle-studio`, or pick another name for the machine |

Some variables, such as `TS_HOSTNAME`, are in the collapsed `Pre-Configured Environment Variables` section of the card. Expand it to see or change them.

#### Deploy the template

On the service card, click `Save Config`. Then click `Review and deploy`. Railway opens your project with the template in a side panel. Click `Deploy Template` there.

#### Remove the public domain

This step is optional. Without it, the database UI stays reachable on its public domain as well as on your tailnet.

Go to the `Settings` tab of the Drizzle Studio service and find the `Networking` section. If Railway generated a public domain, remove it. Keep the private domain.

![Drizzle Studio Settings tab with the delete button of the public domain marked](https://orm.drizzle.team/_astro/railway-drizzle-studio-remove-public-domain.hqzvWid5_1TKMLA.webp)

#### Sign in from the deploy log

Open the `Tailscale` service, then its `Deployments` tab, and read the deploy log. The last lines of the log hold a sign-in link:

```plaintext
This service is not on your tailnet yet. Open this link and sign in
with the account that owns your tailnet:
  https://login.tailscale.com/a/xxxxxxxxxxxx
```

Open the link and sign in. Anyone who opens that link first gets this machine on their tailnet, so do not forward it. The log is visible only to members of your Railway project.

![Tailscale deploy log with the sign-in message and a hidden link](https://orm.drizzle.team/_astro/railway-tailscale-signin-log.ClF8PJDM_Z27w11o.webp)

If your tailnet requires device approval, the log says so and the machine waits until you approve it on the [Machines page](https://login.tailscale.com/admin/machines).

#### Read the address from the deploy log

A few seconds after you sign in, the same log prints where to go:

```plaintext
joined tailnet: example.com
4983: ready
open https://drizzle-studio.example-com.ts.net from any device on your tailnet
```

![Tailscale deploy log with the joined tailnet, open and 4983 ready lines](https://orm.drizzle.team/_astro/railway-tailscale-open-log.DHl8dX05_Z1fMJao.webp)

Check that the tailnet name is yours. If it is not, someone else opened the sign-in link first, and their tailnet can now reach this service. To take the service back:

1. On the project canvas, click the volume under the Tailscale service, open its `Settings` tab, and click `Wipe volume`.
2. Type the confirmation text and click `Wipe`. Railway deletes the machine’s key and redeploys the service.
3. Open the sign-in link from the new deploy log yourself.
4. Change `PASSCODE` on your Drizzle Studio service. If you set `TS_FORWARD_DATABASE`, also change the database user’s password.

If your tailnet already has a machine with that name, the machine name ends in `-1`, `-2` and so on. Use the address the log prints.

If HTTPS certificates are off in your tailnet, the address starts with `http://` and ends in `:4983` instead.

If you set `TS_FORWARD_DATABASE`, the log also prints `5432: ready`.

#### Open Drizzle Studio

Open that address in your web browser. On the first `https://` visit, the Tailscale service still fetches its certificate, so the page can fail or load slowly for about two minutes. If that happens, reload the page.

Drizzle Studio asks for a passcode on first use.

![Drizzle Studio passcode prompt with a hint to check PASSCODE in Railway container variables](https://orm.drizzle.team/_astro/railway-drizzle-studio-passcode-request.BsRdtiv0_2nsikq.webp)

Open your Drizzle Studio service in Railway, go to the `Variables` tab, and copy the value of `PASSCODE`. Paste it into the prompt.

#### Check the connection

Drizzle Studio opens on the database from `DATABASE_URL`, as it did on the public domain.

![Drizzle Studio on the tailnet with the posts table open](https://orm.drizzle.team/_astro/drizzle-studio-on-tailnet.CMPOI83I_2p3xB3.webp)

## Use other ports

By default, `TS_FORWARD_BROWSER` uses port 4983 and `TS_FORWARD_DATABASE` uses port 5432, on the machine and on the service. To use other ports, write the value in the form `[port:]host[:port]`:

| Value | Port on the machine | Port on the service |
| --- | --- | --- |
| `host` | The default port | The default port |
| `host:port` | `port` | `port` |
| `source_port:host:target_port` | `source_port` | `target_port` |

For example, `5433:${{Postgres.RAILWAY_PRIVATE_DOMAIN}}:5432` forwards port 5433 on the machine to port 5432 of the database service.

If the port on the machine for `TS_FORWARD_BROWSER` is not 4983, an `http://` address in the log ends in that port.

## Disable key expiry

A machine that joined by signing in has its key expire on your tailnet’s schedule, 180 days by default. When that happens, the Tailscale service prints a new sign-in link and waits. The address is unreachable until you click it.

To avoid that, open the [Machines page](https://login.tailscale.com/admin/machines) once, find the machine, open the menu at the far right of its row, and choose `Disable key expiry`.

![Tailscale Machines page menu with Disable key expiry](https://orm.drizzle.team/_astro/tailscale-machines-disable-key-expiry.CpBnGkG0_1Jf6GJ.webp)

If you automate deployments and want no sign-in at all, set `TS_AUTHKEY` on the Tailscale service:

- For an auth key, set `TS_AUTHKEY` to the key.
- For an OAuth client secret, set `TS_AUTHKEY` to the secret with `?ephemeral=false&preauthorized=true` appended. Without that suffix, Tailscale removes the machine when it disconnects, and the machine waits for approval before it joins. Also set `TS_TAGS` to a tag you own, for example `tag:drizzle`. A tagged machine never has its key expire.

The Tailscale service reads `TS_AUTHKEY` only before the machine joins. If the machine has already joined, remove the machine and wipe the volume first.

## Troubleshoot the Tailscale setup

Open the entry that matches what you see.

The log prints no `open` line

`TS_FORWARD_BROWSER` is empty, or the Tailscale service skipped it. A reference resolves to an empty value when it names a service that was not in the project when the template deployed. A skipped variable has its own line in the log, and the next entry explains that line. Open the Tailscale service’s `Variables` tab, set `TS_FORWARD_BROWSER` to `${{"Drizzle Gateway".RAILWAY_PRIVATE_DOMAIN}}` (with `"Drizzle Studio"` for Studio), and click `Deploy` in the top bar.

The log prints a line that ends in `skipped`

The line starts with the variable name, then gives the reason:

- `does not match [port:]host[:port]`: the value has more than two colons.
- `has an empty target host`: the value has a port but no host. A reference that names a service outside your project resolves to an empty host.
- `still contains a Railway reference that did not resolve`: Railway did not replace the reference. Check the service name inside it.
- `has a port outside 1-65535`: a port is not a number from 1 to 65535.
- `reuses port`: an earlier forward already uses that port on the machine.
- `uses port 443, which the https listener owns`: pick another port on the machine. The address of the database UI already uses 443 when HTTPS certificates are on.

Fix the value on the Tailscale service’s `Variables` tab and click `Deploy` in the top bar.

A new sign-in link appeared in the log

The service restarted, for example after the key expired or the machine was removed. Open the newest link. The link works once.

The log says the machine is waiting for approval

Your tailnet has device approval turned on. Open the [Machines page](https://login.tailscale.com/admin/machines) and approve it.

The address does not open

Check these causes in order:

- Your device is signed in to another tailnet. Run `tailscale status` on your device and compare its tailnet with the one the log named.
- The log printed an `http://` address. Use that exact address with the port, because `https://` works only when HTTPS certificates are on in your tailnet.
- The Tailscale service is still fetching its certificate on the first `https://` visit. Wait about two minutes and reload the page.
- MagicDNS is off, so the address does not resolve. Enable MagicDNS on the [DNS page](https://login.tailscale.com/admin/dns).

It worked for months and then stopped

The machine’s key expired. Open the Tailscale service’s deploy log; it is printing a fresh sign-in link. Click it, then disable key expiry on the machine so it does not happen again.

The machine name gained a `-1` suffix

Your tailnet already had a machine with this name, for example from an earlier deploy. Tailscale keeps the suffix. If you want the short name, rename the machine on the Machines page. Otherwise, use the address the log prints.

## Next steps

- If you set `TS_FORWARD_DATABASE`, [connect Drizzle Kit to the database](railway-postgres-tailscale.md) to run migrations over your tailnet.
- [Drizzle migrations fundamentals](../migrations.md) to choose how to apply schema changes.
