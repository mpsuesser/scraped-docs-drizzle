---
url: https://orm.drizzle.team/docs/mysql/connect-drizzle-proxy
title: "Connect Drizzle Proxy"
description: ""
access_date: 2026-08-03T19:08:17.353Z
current_date: 2026-08-03T19:08:17.353Z
---

## Drizzle HTTP proxy

This guide assumes familiarity with:

- Database [connection basics](connect-overview.md) with Drizzle

How an HTTP Proxy works and why you might need it

Drizzle Proxy is used when you need to implement your own driver communication with the database. It can be used in several cases, such as adding custom logic at the query stage with existing drivers. The most common use is with an HTTP driver, which sends queries to your server with the database, executes the query on your database, and responds with raw data that Drizzle ORM can then map to results

How it works under the hood?

```plaintext
┌───────────────────────────┐                 ┌─────────────────────────────┐              
│       Drizzle ORM         │                 │  HTTP Server with Database  │             
└─┬─────────────────────────┘                 └─────────────────────────┬───┘             
  │                                                ^                    │
  │-- 1. Build query         2. Send built query --│                    │
  │                                                │                    │
  │              ┌───────────────────────────┐     │                    │
  └─────────────>│                           │─────┘                    │ 
                 │      HTTP Proxy Driver    │                          │
  ┌──────────────│                           │<─────────────┬───────────┘
  │              └───────────────────────────┘              │
  │                                                  3. Execute a query + send raw results back
  │-- 4. Map data and return        
  │                   
  v
```

Drizzle ORM also supports simply using asynchronous callback function for executing SQL.

- `sql` is a query string with placeholders.
- `params` is an array of parameters.
- One of the following values will set for `method` depending on the SQL statement - `all`, `execute`.

Drizzle always waits for `{rows: string[][]}` or `{rows: string[]}` for the return value.

- When the `method` is `execute`, you should return a value as `{rows: string[]}`.
- Otherwise, you should return `{rows: string[][]}`.

```typescript
// Example of driver implementation
import { drizzle } from 'drizzle-orm/mysql-proxy';
import axios from "axios";

const db = drizzle(async (sql, params, method) => {
  try {
    const rows = await axios.post('http://localhost:3000/query', { sql, params, method });

    return { rows: rows.data };
  } catch (e: any) {
    console.error('Error from mysql proxy server: ', e.response.data)
    return { rows: [] };
  }
});
```
```ts
// Example of server implementation
import * as mysql from 'mysql2/promise';
import express from 'express';

const app = express();

app.use(express.json());
const port = 3000;

const main = async () => {
    const connection = await mysql.createConnection('mysql://root:mysql@127.0.0.1:5432/drizzle');

    app.post('/query', async (req, res) => {
      const { sql, params, method } = req.body;

      // prevent multiple queries
      const sqlBody = sql.replace(/;/g, '');

      try {
            const result = await connection.query({
                sql: sqlBody,
                values: params,
                rowsAsArray: method === 'all',
                typeCast: function(field: any, next: any) {
                    if (field.type === 'TIMESTAMP' || field.type === 'DATETIME' || field.type === 'DATE') {
                        return field.string();
                    }
                    return next();
                },
            });
      } catch (e: any) {
        res.status(500).json({ error: e });
      }

      if (method === 'all') {
        res.send(result[0]);
      } else if (method === 'execute') {
        res.send(result);
      }
      res.status(500).json({ error: 'Unknown method value' });
    });

    app.listen(port, () => {
      console.log(\`Example app listening on port ${port}\`);
    });
}

main();
```
