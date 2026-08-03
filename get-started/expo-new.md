---
url: https://orm.drizzle.team/docs/get-started/expo-new
title: "Expo New"
description: ""
access_date: 2026-08-03T19:38:26.356Z
current_date: 2026-08-03T19:38:26.356Z
---

## Get Started with Drizzle and Expo

This guide assumes familiarity with:

- **Expo SQLite** - A library that provides access to a database that can be queried through a SQLite API - [read here](https://docs.expo.dev/versions/latest/sdk/sqlite/)

#### Step 1 - Setup a project from Expo Template

```shell
npx create expo-app --template blank-typescript
```

You can read more about this template [here](https://docs.expo.dev/more/create-expo/#create-a-new-project).

#### Basic file structure

After installing the template and adding the `db` folder, you’ll find the following content: In the `db/schema.ts` file with drizzle table definitions. The `drizzle` folder contains SQL migration files and snapshots

```plaintext
📦 <project root>
 ├ 📂 assets
 ├ 📂 drizzle
 ├ 📂 db
 │  └ 📜 schema.ts
 ├ 📜 .gitignore
 ├ 📜 .npmrc
 ├ 📜 app.json
 ├ 📜 App.tsx
 ├ 📜 babel.config.ts
 ├ 📜 drizzle.config.ts
 ├ 📜 package.json
 └ 📜 tsconfig.json
```

#### Step 2 - Install expo-sqlite package

```shell
npx expo install expo-sqlite
```

#### Step 3 - Install required packages

```shell
npm i drizzle-orm@rc
npm i -D drizzle-kit@rc
```

#### Step 4 - Connect Drizzle ORM to the database

Create a `App.tsx` file in the root directory and initialize the connection:

```ts
import * as SQLite from 'expo-sqlite';
import { drizzle } from 'drizzle-orm/expo-sqlite';

const expo = SQLite.openDatabaseSync('db.db');

const db = drizzle(expo);
```

#### Step 4 - Create a table

Create a `schema.ts` file in the `db` directory and declare your table:

```typescript
import { int, sqliteTable, text } from "drizzle-orm/sqlite-core";

export const usersTable = sqliteTable("users_table", {
  id: int().primaryKey({ autoIncrement: true }),
  name: text().notNull(),
  age: int().notNull(),
  email: text().notNull().unique(),
});
```

#### Step 5 - Setup Drizzle config file

**Drizzle config** - a configuration file that is used by [Drizzle Kit](../kit-overview.md) and contains all the information about your database connection, migration folder and schema files.

Create a `drizzle.config.ts` file in the root of your project and add the following content:

```ts
import { defineConfig } from 'drizzle-kit';

export default defineConfig({
  dialect: 'sqlite',
  driver: 'expo',
  schema: './db/schema.ts',
  out: './drizzle',
});
```

#### Step 6 - Setup metro config

Create a file `metro.config.js` in root folder and add this code inside:

```js
const { getDefaultConfig } = require('expo/metro-config');
/** @type {import('expo/metro-config').MetroConfig} */
const config = getDefaultConfig(__dirname);
config.resolver.sourceExts.push('sql');
module.exports = config;
```

#### Step 7 - Update babel config

```js
module.exports = function(api) {
  api.cache(true);
  return {
    presets: ['babel-preset-expo'],
    plugins: [["inline-import", { "extensions": [".sql"] }]] // <-- add this
  };
};
```

#### Step 8 - Applying changes to the database

With Expo, you would need to generate migrations using the `drizzle-kit generate` command and then apply them at runtime using the `drizzle-orm` `migrate()` function

Generate migrations:

```bash
npx drizzle-kit generate
```

#### Step 9 - Apply migrations and query your db:

Let’s **App.tsx** file with migrations and queries to create, read, update, and delete users

```ts
import { Text, View } from 'react-native';
import * as SQLite from 'expo-sqlite';
import { useEffect, useState } from 'react';
import { drizzle } from 'drizzle-orm/expo-sqlite';
import { usersTable } from './db/schema';
import { useMigrations } from 'drizzle-orm/expo-sqlite/migrator';
import migrations from './drizzle/migrations';

const expo = SQLite.openDatabaseSync('db.db');

const db = drizzle(expo);

export default function App() {
  const { success, error } = useMigrations(db, migrations);
  const [items, setItems] = useState<typeof usersTable.$inferSelect[] | null>(null);

  useEffect(() => {
    if (!success) return;

    (async () => {
      await db.delete(usersTable);

      await db.insert(usersTable).values([
        {
            name: 'John',
            age: 30,
            email: 'john@example.com',
        },
      ]);

      const users = await db.select().from(usersTable);
      setItems(users);
    })();
  }, [success]);

  if (error) {
    return (
      <View>
        <Text>Migration error: {error.message}</Text>
      </View>
    );
  }

  if (!success) {
    return (
      <View>
        <Text>Migration is in progress...</Text>
      </View>
    );
  }

  if (items === null || items.length === 0) {
    return (
      <View>
        <Text>Empty</Text>
      </View>
    );
  }

  return (
    <View
      style={{
        display: 'flex',
        flexDirection: 'column',
        alignItems: 'center',
        width: '100%',
        height: '100%',
        justifyContent: 'center',
      }}
    >
      {items.map((item) => (
        <Text key={item.id}>{item.email}</Text>
      ))}
    </View>
  );
}
```

#### Step 10 - Prebuild and run expo app

```bash
npx expo run:ios
```
