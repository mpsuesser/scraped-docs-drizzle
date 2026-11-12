---
url: https://orm.drizzle.team/docs/mysql/agents
title: "Agents"
description: ""
access_date: 2026-11-12T09:15:29.353Z
current_date: 2026-11-12T09:15:29.353Z
---

## Set up your AI agent

Give your coding agent Drizzle skills and access to your project’s Drizzle Kit.

The setup instructions currently target the `@rc` packages. Existing legacy or beta projects need a separate upgrade.

## Copy into your agent

Open your coding agent in your project, then paste this prompt.

View prompt
```
Set up my coding agent for Drizzle in this project. Fetch and follow the instructions at https://orm.drizzle.team/agent-setup/prompt.md
```
[Read the full setup instructions](https://orm.drizzle.team/agent-setup/prompt.md)

## What setup does

1. **Checks your Drizzle packages.** Uses your project’s package manager and existing compatible rc versions. Missing packages are installed from `drizzle-orm@rc` and `drizzle-kit@rc`.
2. **Installs the Drizzle skill.** Adds the instructions bundled with your installed Kit so your agent can work with schema, queries, and migrations.
3. **Connects Drizzle Kit.** Configures a local MCP server for your agent and verifies the connection. Some clients need a new session or approval before the tools become available.

Setup keeps your existing agent configuration. It does not change your application’s schema, generate migrations, or apply changes to a database.

## After setup

Your agent reports what was installed and whether you need to approve access or restart your session.

Once your agent confirms the skill and tools are available, give it a first task:

```text
Use the drizzle skill to inspect my Drizzle schema and config. Explain the
migration workflow this project uses. Do not change files or the database.
```

## Set up manually

- [Install the Drizzle skill](../drizzle-kit-skills.md) for project and migration guidance.
- [Connect the MCP server](../drizzle-kit-mcp.md) so your agent can use the Kit installed in your project.
