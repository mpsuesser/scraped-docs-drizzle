---
url: https://orm.drizzle.team/docs/sqlite/tutorials/drizzle-nextjs-neon
title: "Drizzle Nextjs Neon"
description: ""
access_date: 2026-08-03T19:38:26.356Z
current_date: 2026-08-03T19:38:26.356Z
---

## Todo App with Neon Postgres

This tutorial demonstrates how to build `Todo app` using **Drizzle ORM** with **Neon database** and **Next.js**.

This guide assumes familiarity with:

- You should have an existing Next.js project or create a new one using the following command:
```bash
npx create-next-app@latest --typescript
```
- You should have installed Drizzle ORM and [Drizzle kit](../../kit-overview.md). You can do this by running the following command:

```shell
npm i drizzle-orm@rc
npm i -D drizzle-kit@rc
```

- You should have installed the [Neon serverless driver](https://neon.tech/docs/serverless/serverless-driver).

```shell
npm i @neondatabase/serverless
```

- You should have installed the `dotenv` package for managing environment variables.

```shell
npm i dotenv
```

IMPORTANT

In case you face the issue with resolving dependencies during installation:

If you’re not using React Native, forcing the installation with `--force` or `--legacy-peer-deps` should resolve the issue. If you are using React Native, then you need to use the exact version of React which is compatible with your React Native version.

## Setup Neon and Drizzle ORM

#### Create a new Neon project

Log in to the [Neon Console](https://console.neon.tech/app/projects) and navigate to the Projects section. Select a project or click the `New Project` button to create a new one.

Your Neon projects come with a ready-to-use Postgres database named `neondb`. We’ll use it in this tutorial.

#### Setup connection string variable

Navigate to the **Connection Details** section in the project console to find your database connection string. It should look similar to this:

```bash
postgres://username:password@ep-cool-darkness-123456.us-east-2.aws.neon.tech/neondb
```

Add the `DATABASE_URL` environment variable to your `.env` or `.env.local` file, which you’ll use to connect to the Neon database.

```bash
DATABASE_URL=NEON_DATABASE_CONNECTION_STRING
```

#### Connect Drizzle ORM to your database

Create a `drizzle.ts` file in your `src/db` folder and set up your database configuration:

```tsx
import { config } from "dotenv";
import { drizzle } from 'drizzle-orm/neon-http';

config({ path: ".env" }); // or .env.local

export const db = drizzle(process.env.DATABASE_URL!);
```

#### Declare todo schema

```tsx
import { integer, text, boolean, pgTable } from "drizzle-orm/pg-core";

export const todo = pgTable("todo", {
  id: integer("id").primaryKey(),
  text: text("text").notNull(),
  done: boolean("done").default(false).notNull(),
});
```

Here we define the **`todo`** table with fields **`id`**, **`text`**, and **`done`**, using data types from Drizzle ORM.

#### Setup Drizzle config file

**Drizzle config** - a configuration file that is used by [Drizzle Kit](../../kit-overview.md) and contains all the information about your database connection, migration folder and schema files.

Create a `drizzle.config.ts` file in the root of your project and add the following content:

```typescript
import { config } from 'dotenv';
import { defineConfig } from "drizzle-kit";

config({ path: '.env' });

export default defineConfig({
  schema: "./src/db/schema.ts",
  out: "./migrations",
  dialect: "postgresql",
  dbCredentials: {
    url: process.env.DATABASE_URL!,
  },
});
```

#### Applying changes to the database

You can generate migrations using `drizzle-kit generate` command and then run them using the `drizzle-kit migrate` command.

Generate migrations:

```bash
npx drizzle-kit generate
```

These migrations are stored in the `drizzle/migrations` directory, as specified in your `drizzle.config.ts`. This directory will contain the SQL files necessary to update your database schema and a `meta` folder for storing snapshots of the schema at different migration stages.

Example of a generated migration:

```sql
CREATE TABLE IF NOT EXISTS "todo" (
    "id" integer PRIMARY KEY NOT NULL,
    "text" text NOT NULL,
    "done" boolean DEFAULT false NOT NULL
);
```

Run migrations:

```bash
npx drizzle-kit migrate
```

Alternatively, you can push changes directly to the database using [Drizzle kit push command](../../kit-overview.md#prototyping-with-db-push):

```bash
npx drizzle-kit push
```

IMPORTANT

Push command is good for situations where you need to quickly test new schema designs or changes in a local development environment, allowing for fast iterations without the overhead of managing migration files.

#### Establish server-side functions

In this step, we establish server-side functions in the **src/actions/todoAction.ts** file to handle crucial operations on todo items:

1. **`getData`:**
	- Fetches all existing todo items from the database.
2. **`addTodo`:**
	- Adds a new todo item to the database with the provided text.
		- Initiates revalidation of the home page using **`revalidatePath("/")`**.
3. **`deleteTodo`:**
	- Removes a todo item from the database based on its unique ID.
		- Triggers a revalidation of the home page.
4. **`toggleTodo`:**
	- Toggles the completion status of a todo item, updating the database accordingly.
		- Revalidates the home page after the operation.
5. **`editTodo`:**
	- Modifies the text of a todo item identified by its ID in the database.
		- Initiates a revalidation of the home page.
```tsx
"use server";
import { eq, not } from "drizzle-orm";
import { revalidatePath } from "next/cache";
import { db } from "@/db/drizzle";
import { todo } from "@/db/schema";

export const getData = async () => {
  const data = await db.select().from(todo);
  return data;
};

export const addTodo = async (id: number, text: string) => {
  await db.insert(todo).values({
    id: id,
    text: text,
  });
};

export const deleteTodo = async (id: number) => {
  await db.delete(todo).where(eq(todo.id, id));

  revalidatePath("/");
};

export const toggleTodo = async (id: number) => {
  await db
    .update(todo)
    .set({
      done: not(todo.done),
    })
    .where(eq(todo.id, id));

  revalidatePath("/");
};

export const editTodo = async (id: number, text: string) => {
  await db
    .update(todo)
    .set({
      text: text,
    })
    .where(eq(todo.id, id));

  revalidatePath("/");
};
```

## Setup home page with Next.js

#### Define a TypeScript type

Define a TypeScript type for a todo item in `src/types/todoType.ts` with three properties: **`id`** of type **`number`**, **`text`** of type **`string`**, and **`done`** of type **`boolean`**. This type, named **`todoType`**, represents the structure of a typical todo item within your application.

```ts
export type todoType = {
  id: number;
  text: string;
  done: boolean;
};
```

#### Create a home page for a to-do application

1. **`src/components/todo.tsx`:** Create a `Todo` component that represents a single todo item. It includes features for displaying and editing the todo text, marking it as done with a checkbox, and providing actions for editing, saving, canceling, and deleting the todo.
2. **`src/components/addTodo.tsx`:** The `AddTodo` component provides a simple form for adding new todo items to the Todo app. It includes an input field for entering the todo text and a button for triggering the addition of the new todo.
3. **`src/components/todos.tsx`:** Create Todos components that represents the main interface of a Todo app. It manages the state of todo items, provides functions for creating, editing, toggling, and deleting todos, and renders the individual todo items using the `Todo` component.

```tsx
"use client";
import { ChangeEvent, FC, useState } from "react";
import { todoType } from "@/types/todoType";

interface Props {
  todo: todoType;
  changeTodoText: (id: number, text: string) => void;
  toggleIsTodoDone: (id: number, done: boolean) => void;
  deleteTodoItem: (id: number) => void;
}

const Todo: FC<Props> = ({
  todo,
  changeTodoText,
  toggleIsTodoDone,
  deleteTodoItem,
}) => {
  // State for handling editing mode
  const [editing, setEditing] = useState(false);

  // State for handling text input
  const [text, setText] = useState(todo.text);

  // State for handling "done" status
  const [isDone, setIsDone] = useState(todo.done);

  // Event handler for text input change
  const handleTextChange = (e: ChangeEvent<HTMLInputElement>) => {
    setText(e.target.value);
  };

  // Event handler for toggling "done" status
  const handleIsDone = async () => {
    toggleIsTodoDone(todo.id, !isDone);
    setIsDone((prev) => !prev);
  };

  // Event handler for initiating the edit mode
  const handleEdit = () => {
    setEditing(true);
  };

  // Event handler for saving the edited text
  const handleSave = async () => {
    changeTodoText(todo.id, text);
    setEditing(false);
  };

  // Event handler for canceling the edit mode
  const handleCancel = () => {
    setEditing(false);
    setText(todo.text);
  };

  // Event handler for deleting a todo item
  const handleDelete = () => {
    if (confirm("Are you sure you want to delete this todo?")) {
      deleteTodoItem(todo.id);
    }
  };

  // Rendering the Todo component
  return (
    <div className="flex items-center gap-2 p-4 border-gray-200 border-solid border rounded-lg">
      {/* Checkbox for marking the todo as done */}
      <input
        type="checkbox"
        className="text-blue-200 rounded-sm h-4 w-4"
        checked={isDone}
        onChange={handleIsDone}
      />
      {/* Input field for todo text */}
      <input
        type="text"
        value={text}
        onChange={handleTextChange}
        readOnly={!editing}
        className={\`${
          todo.done ? "line-through" : ""
        } outline-none read-only:border-transparent focus:border border-gray-200 rounded px-2 py-1 w-full\`}
      />
      {/* Action buttons for editing, saving, canceling, and deleting */}
      <div className="flex gap-1 ml-auto">
        {editing ? (
          <button
            onClick={handleSave}
            className="bg-green-600 text-green-50 rounded px-2 w-14 py-1"
          >
            Save
          </button>
        ) : (
          <button
            onClick={handleEdit}
            className="bg-blue-400 text-blue-50 rounded w-14 px-2 py-1"
          >
            Edit
          </button>
        )}
        {editing ? (
          <button
            onClick={handleCancel}
            className="bg-red-400 w-16 text-red-50 rounded px-2 py-1"
          >
            Close
          </button>
        ) : (
          <button
            onClick={handleDelete}
            className="bg-red-400 w-16 text-red-50 rounded px-2 py-1"
          >
            Delete
          </button>
        )}
      </div>
    </div>
  );
};

export default Todo;
```

Update the `page.tsx` file in the `src/app` folder to fetch the todo items from the database and render the `Todos` component:

```tsx
import { getData } from "@/actions/todoAction";
import Todos from "@/components/todos";

export default async function Home() {
  const data = await getData();
  return <Todos todos={data} />;
}
```

## Basic file structure

This guide uses the following file structure:

```text
📦 <project root>
 ├ 📂 migrations
 │  ├ 📂 meta
 │  └ 📜 0000_heavy_doctor_doom.sql
 ├ 📂 public
 ├ 📂 src
 │  ├ 📂 actions
 │  │  └ 📜 todoActions.ts
 │  ├ 📂 app
 │  │  ├ 📜 favicon.ico
 │  │  ├ 📜 globals.css
 │  │  ├ 📜 layout.tsx
 │  │  └ 📜 page.tsx
 │  ├ 📂 components
 │  │  ├ 📜 addTodo.tsx
 │  │  ├ 📜 todo.tsx
 │  │  └ 📜 todos.tsx
 │  └ 📂 db
 │  │  ├ 📜 drizzle.ts
 │  │  └ 📜 schema.ts
 │  └ 📂 types
 │     └ 📜 todoType.ts
 ├ 📜 .env
 ├ 📜 .eslintrc.json
 ├ 📜 .gitignore
 ├ 📜 drizzle.config.ts
 ├ 📜 next-env.d.ts
 ├ 📜 next.config.mjs
 ├ 📜 package-lock.json
 ├ 📜 package.json
 ├ 📜 postcss.config.mjs
 ├ 📜 README.md
 ├ 📜 tailwind.config.ts
 └ 📜 tsconfig.json
```
