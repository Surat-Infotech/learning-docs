# Working with a Database

Most real apps need to **remember** things between visits: users, posts,
orders, comments. A web page forgets everything when it closes. A **database**
is the long-term memory that keeps your data safe in one place.

Think of a database like a filing cabinet in a back office. Customers (the
browser) never walk into the back office themselves. They ask a staff member
(your server) to fetch or file a document. In Next.js, that "back office" is
your **Server Components**, **Server Actions**, and **Route Handlers** — never
the browser.

This lesson is a map, not a full setup tutorial. We will explain the pieces and
how they fit together so the later hands-on steps make sense.

## What You'll Learn

- What a database is and where it lives in a Next.js app
- Why the browser must never talk to the database directly
- High-level choices: Postgres, SQLite, MySQL, and hosted services
- What an **ORM** is and why it makes database work easier
- A tiny Prisma-style example: schema then query
- How to keep database credentials server-only
- The connection pooling caveat in serverless environments

## Where the Database Fits

In Next.js, code runs in two very different places:

- **On the server** — Server Components, Server Actions, Route Handlers. This
  code runs on your machine or hosting provider. It is private.
- **In the browser** — Client Components (`"use client"`). This code is sent to
  the user and runs on their device. It is public.

The database must only be touched by **server code**. Here is the flow:

```text
Browser  ->  Server Component / Action / Route Handler  ->  Database
(public)              (private, trusted)                   (private)
```

If the browser could reach the database directly, anyone could open dev tools,
read your connection password, and delete every record. So the rule is simple:

> The database is server-only. Always.

```tsx
// app/page.tsx
// A Server Component (no "use client") runs on the server,
// so it is allowed to query the database directly.
import { db } from "@/lib/db";

export default async function HomePage() {
  // This runs on the server, before HTML is sent to the browser.
  const users = await db.user.findMany(); // safe: server-only

  return (
    <ul>
      {users.map((u) => (
        <li key={u.id}>{u.name}</li>
      ))}
    </ul>
  );
}
```

The browser only ever receives the finished HTML. It never sees the query or
the credentials.

## Choosing a Database

You do not need to pick the "perfect" one to start. Here is a high-level menu.

### Relational databases (tables, rows, columns)

These store data in tables, like a spreadsheet, and let tables relate to each
other (a user has many posts). Most apps use one of these.

- **PostgreSQL (Postgres)** — powerful, popular, a great default. Open source.
- **MySQL** — also very popular and battle-tested.
- **SQLite** — a tiny database stored in a single file. Perfect for learning and
  small projects because there is no server to run.

### Hosted (managed) databases

You do not want to babysit a database server yourself. Hosted services run it
for you and give you a connection string. Common choices:

- **Neon** — serverless Postgres, great fit for Vercel.
- **Supabase** — Postgres plus extras like auth and storage.
- **PlanetScale** — managed MySQL that scales well.

For a first project, **SQLite locally** or **Neon/Supabase Postgres** are both
friendly starting points.

## What Is an ORM?

Writing raw database commands looks like this (the language is called **SQL**):

```sql
-- Raw SQL: powerful but easy to get wrong
SELECT id, name FROM "User" WHERE email = 'sam@example.com';
```

An **ORM** (Object-Relational Mapping) is a tool that lets you talk to the
database using normal JavaScript/TypeScript objects and methods instead of raw
SQL strings. "Object-relational mapping" just means it **maps** database rows
(relational data) to objects in your code.

Why an ORM helps:

- **Type safety** — your editor knows `user.name` is a string and warns you
  about typos before the app runs.
- **Less boilerplate** — no hand-writing every SQL string.
- **Readable code** — `db.user.findMany()` reads like plain English.
- **Migrations** — it can update the database shape as your app grows.

Two popular ORMs in the Next.js world:

- **Prisma** — friendly, lots of guides, great TypeScript support.
- **Drizzle** — lightweight, closer to SQL, very fast.

We will use Prisma-style examples below because they read clearly.

## A Tiny Prisma Example

### Step 1: Describe your data (the schema)

The **schema** is a file that describes the shape of your tables. Prisma reads
this file and creates matching tables and a typed client for you.

```text
// prisma/schema.prisma
// This describes a "User" table with three columns.

datasource db {
  provider = "postgresql"        // which database we use
  url      = env("DATABASE_URL")  // connection string from env (next lesson)
}

generator client {
  provider = "prisma-client-js"   // generates the typed db client
}

model User {
  id    String @id @default(cuid()) // unique id, auto-generated
  email String @unique              // no two users share an email
  name  String                      // the user's display name
}
```

### Step 2: Create a single shared client

```ts
// lib/db.ts
// Create ONE PrismaClient and reuse it everywhere.
import { PrismaClient } from "@prisma/client";

// In development, Next.js reloads often. Without this guard we would
// create a new client on every reload and run out of connections.
const globalForPrisma = globalThis as unknown as {
  prisma?: PrismaClient;
};

export const db = globalForPrisma.prisma ?? new PrismaClient();

if (process.env.NODE_ENV !== "production") {
  globalForPrisma.prisma = db; // remember it across hot reloads
}
```

### Step 3: Query it from a Server Component

```tsx
// app/users/page.tsx
import { db } from "@/lib/db";

export default async function UsersPage() {
  // findMany() reads all rows from the User table.
  const users = await db.user.findMany({
    orderBy: { name: "asc" }, // sort A -> Z
  });

  return (
    <main>
      <h1>Users</h1>
      <ul>
        {users.map((user) => (
          // user.name is fully typed thanks to the ORM
          <li key={user.id}>{user.name} ({user.email})</li>
        ))}
      </ul>
    </main>
  );
}
```

You can also write to the database from a **Server Action**:

```tsx
// app/users/actions.ts
"use server";
import { db } from "@/lib/db";

export async function createUser(name: string, email: string) {
  // create() inserts one new row and returns it.
  return db.user.create({
    data: { name, email },
  });
}
```

## Keeping Credentials Server-Only

The database connection string contains a password. It must stay on the server.

```text
// Looks like this (never share it):
DATABASE_URL="postgresql://user:secretpass@host:5432/mydb"
```

You store it in an environment variable, not in your code. Because the variable
is **not** prefixed with `NEXT_PUBLIC_`, Next.js keeps it on the server and
never ships it to the browser. The next lesson covers this in detail.

```tsx
// app/page.tsx  (Server Component)
// Reading the secret here is fine — this code never reaches the browser.
const url = process.env.DATABASE_URL;
```

```tsx
// app/widget.tsx
"use client";
// DO NOT do this. A Client Component runs in the browser.
// process.env.DATABASE_URL is undefined here, and trying to use the
// database from the browser would be a serious security mistake.
```

## The Serverless Connection Pooling Caveat

A short but important warning. When you deploy to a **serverless** platform
(like Vercel), your code may spin up many short-lived instances. Each one can
open its own database connection. Traditional databases only allow a limited
number of connections, so you can run out fast.

The fix, in plain terms: use a **connection pooler** — a middle layer that
shares a small set of connections among many instances. Hosted Postgres
services help here:

- Neon offers a **pooled** connection string (use it for serverless).
- Supabase offers a pooler on a separate port.
- Prisma offers **Accelerate**; Drizzle has serverless drivers too.

You do not need to solve this on day one. Just remember: in serverless, prefer
the **pooled** connection URL your host provides.

## Common Mistakes

- **Querying the database from a Client Component.** It will not work and is a
  security risk. Move the query into a Server Component or Server Action.
- **Creating a new database client on every request.** Use the single shared
  client pattern shown above to avoid running out of connections.
- **Hardcoding the connection string in code.** Always read it from an
  environment variable so it stays out of your repository.
- **Using `NEXT_PUBLIC_` for the database URL.** That prefix exposes it to the
  browser. Never do this for secrets.
- **Ignoring pooling in serverless.** A working local app can hit connection
  limits in production. Use the pooled URL when deploying serverless.

## Exercises

1. In one sentence, explain why the browser must never connect directly to the
   database. Name the three server-only places that may query it.
2. Add a `Post` model to the Prisma schema sketch above with `id`, `title`, and
   a `published` boolean that defaults to `false`. (Just write the model block.)
3. Write a Server Component that lists all posts where `published` is `true`,
   sorted by `title`. Assume `db.post.findMany` works like `db.user.findMany`.
4. Explain the difference between `DATABASE_URL` and a `NEXT_PUBLIC_` variable
   in terms of who can read it.
5. In your own words, what problem does a connection pooler solve in a
   serverless deployment?

## Key Takeaways

- A database is your app's long-term memory; only **server code** may touch it.
- Query from Server Components, Server Actions, or Route Handlers — never the
  client.
- Choices range from file-based **SQLite** to hosted **Postgres** (Neon,
  Supabase) and **MySQL** (PlanetScale).
- An **ORM** (Prisma, Drizzle) lets you use typed JavaScript instead of raw SQL.
- Keep the connection string in a non-public environment variable.
- In serverless, use a **pooled** connection URL to avoid connection limits.

---

[← Previous: Authentication Basics](../04-building-features/05-authentication-basics.md) | [Back to Course Index](../README.md) | [Next: Environment Variables →](02-environment-variables.md)
