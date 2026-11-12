---
url: https://orm.drizzle.team/docs/mssql/drizzle-kit-mcp
title: "Drizzle Kit Mcp"
description: ""
access_date: 2026-11-12T09:01:42.866Z
current_date: 2026-11-12T09:01:42.866Z
---

## Drizzle Kit MCP server

The Drizzle Kit MCP server lets you ask your coding agent to generate migrations, check migration history or pull a schema from an existing database. The agent calls the Drizzle Kit installed in your project.

Connecting the server gives your agent access to operations that write project files and change the database selected by your config. Generating a migration writes a file but does not apply it to the database. Your client’s permissions and approval settings govern tool execution.

## Quick start

1. ### Install Drizzle in your project
	These examples use `drizzle-kit@rc` and `drizzle-orm@rc`. Install them in the package that owns your Drizzle schema and config:
	```shell
	npm i drizzle-orm@rc
	npm i -D drizzle-kit@rc
	```
	In a monorepo, use the workspace package that uses Drizzle. **Start the MCP server from that directory** so it can find both packages and your config.
	The server uses your project’s [Drizzle Kit configuration](../drizzle-config-file.md). For `push` or `pull`, that config also needs database credentials.
2. ### Connect your MCP client
	Choose your client below. For CLI setup, run the command from your Drizzle package with the client’s CLI installed.
	Claude Code
	Codex
	OpenCode
	Grok
	Cursor
	Other clients
	Register the server for this project:
	```shell
	claude mcp add --transport stdio --scope project drizzle -- npx --no-install drizzle-kit mcp
	```
	This writes the entry to `.mcp.json`. When Claude Code asks to approve the project server, approve it to connect. Use `/mcp` inside Claude Code to check its status.
	See [Claude Code’s MCP setup](https://code.claude.com/docs/en/mcp) for configuration scopes and client-specific options.
3. ### Verify the connection
	Restart or reconnect the server in your client, then check that its Drizzle tools are available.
	Your client manages the server process. You do not need an HTTP port or a separate terminal running it.
4. ### Try it with your agent
	Once connected, describe the work you want done. For example, inspect migration history without changing it:
	```text
	Use the Drizzle MCP server to check my migration history.
	Explain any conflicts or warnings without changing files or the database.
	```

## Use the right configuration

Drizzle Kit looks for `drizzle.config.*` in the **server’s working directory**. If you use a different config file, tell your agent which path to use. Relative paths resolve from that directory, not the client’s.

## Generate and check migrations

Generate a migration for a schema change you’ve already made:

```text
Use the Drizzle MCP server to generate a migration for my schema changes.
Show me the SQL and explain what it changes. Don't apply it to the database.
```

For PostgreSQL, you can also compare your configured schema with your migrations:

```text
Use the Drizzle MCP server to check whether my PostgreSQL schema
and migrations are in sync. Explain any differences without changing files.
```

Apply reviewed migration files through your existing [migration workflow](../drizzle-kit-migrate.md). rc5 does not expose a `migrate` MCP tool.

## Add the Drizzle skill

Add the [Drizzle skill](../drizzle-kit-skills.md) to help your agent choose a migration workflow, handle rename decisions and interpret results.

[Install the skill from your project’s Drizzle Kit](../drizzle-kit-skills.md#install-the-skill-from-your-projects-drizzle-kit), then ask your agent to use it:

```text
Use the Drizzle skill and MCP server. In src/db/schema.ts, rename
users.full_name to users.display_name and generate a migration.
Show me the SQL before applying anything to the database.
```

For tool arguments and response details, see the [MCP reference](https://github.com/drizzle-team/drizzle-orm/blob/beta/drizzle-kit/MCP.md).
