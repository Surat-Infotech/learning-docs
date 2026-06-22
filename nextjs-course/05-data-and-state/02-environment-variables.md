# Environment Variables

Your app needs to know things that should not live in your code: a database
password, an API key, a "are we in development or production?" flag. These are
called **environment variables** — values that come from the surrounding
**environment** your app runs in, not from the source files themselves.

Think of environment variables like the settings dial on the back of an
appliance. The appliance (your code) stays the same. You just turn the dial
differently depending on where it is plugged in — your laptop, a test server, or
the real production site.

In this lesson we will see how Next.js reads these values, which ones are safe
for the browser, and the one mistake you must never make: leaking a secret.

## What You'll Learn

- What environment variables are and why they exist
- The difference between `.env`, `.env.local`, and other env files
- How to read a variable with `process.env.X`
- Server-only variables versus `NEXT_PUBLIC_` ones (and the danger)
- Why you must never commit `.env.local`
- How to set environment variables when deploying (e.g. Vercel)

## Why Environment Variables?

Two big reasons:

1. **Secrets.** A database password or API key must not sit in your code where
   anyone reading the repository could see it.
2. **Config per environment.** The same code might point at a test database on
   your laptop and the real database in production. The value changes; the code
   does not.

```ts
// lib/db.ts
// The code asks for a value by name. WHERE that value comes from
// depends on the environment, not on this file.
const databaseUrl = process.env.DATABASE_URL;
```

On your laptop `DATABASE_URL` might point to a local test database. In
production it points to the real one. Same line of code, different value.

## The `.env` Files

Next.js automatically loads environment variables from files named `.env*` in
your project root. You write them as `NAME=value`, one per line.

```text
# .env.local
# Real secrets for YOUR machine. NOT committed to git.
DATABASE_URL="postgresql://user:secretpass@localhost:5432/mydb"
STRIPE_SECRET_KEY="sk_test_abc123"
```

Here are the common files and how Next.js treats them:

- **`.env`** — default values shared by everyone. Safe, non-secret defaults can
  go here, and it is usually committed.
- **`.env.local`** — your personal secrets and overrides. **Never committed.**
  Next.js loads it in every environment except testing.
- **`.env.development`** / **`.env.production`** — values used only when running
  in that mode. Often committed if they hold non-secret config.

If the same variable appears in several files, `.env.local` wins. The simple
rule for beginners: **put real secrets in `.env.local`**.

After editing any `.env` file you must **restart the dev server**, because
Next.js only reads these files on startup.

```bash
# Stop the server (Ctrl+C), then start it again so it re-reads .env files.
npm run dev
```

## Reading a Variable: `process.env`

`process.env` is an object holding all the environment variables. You read one
by its name:

```ts
// app/actions.ts
"use server";

export async function chargeCard() {
  // Read the secret key on the server.
  const key = process.env.STRIPE_SECRET_KEY;

  if (!key) {
    // Always handle the "it's missing" case.
    throw new Error("STRIPE_SECRET_KEY is not set");
  }

  // ... use key to talk to Stripe ...
}
```

Note that `process.env.SOMETHING` is a **string** (or `undefined` if it is not
set). If you need a number or boolean, convert it:

```ts
// Convert a string to a number.
const port = Number(process.env.PORT ?? "3000");

// Convert a string to a boolean (env values are always strings).
const debug = process.env.DEBUG === "true";
```

## Server-Only vs `NEXT_PUBLIC_`

This is the most important idea in the lesson. By default, an environment
variable is **server-only**: it exists when your code runs on the server and is
**not** sent to the browser.

```tsx
// app/page.tsx  (Server Component — runs on the server)
export default function Page() {
  // Works: this code runs on the server.
  const secret = process.env.STRIPE_SECRET_KEY;
  // ... use secret safely ...
  return <p>Server is configured.</p>;
}
```

```tsx
// app/widget.tsx
"use client"; // runs in the browser

export default function Widget() {
  // process.env.STRIPE_SECRET_KEY is UNDEFINED here.
  // Next.js does not ship server-only vars to the browser. Good!
  const secret = process.env.STRIPE_SECRET_KEY; // => undefined
  return <p>Hi</p>;
}
```

Sometimes you genuinely need a value in the browser — for example a public API
URL or a non-secret analytics ID. For those, prefix the name with
**`NEXT_PUBLIC_`**. Next.js will then embed that value into the browser bundle.

```text
# .env.local
# This WILL be visible to the browser. Only use it for non-secrets.
NEXT_PUBLIC_API_URL="https://api.example.com"
```

```tsx
// app/widget.tsx
"use client";

export default function Widget() {
  // Works in the browser because of the NEXT_PUBLIC_ prefix.
  const apiUrl = process.env.NEXT_PUBLIC_API_URL;
  return <p>API: {apiUrl}</p>;
}
```

> Warning: `NEXT_PUBLIC_` variables are **baked into the JavaScript sent to
> every visitor**. Anyone can open dev tools and read them. Never put a
> password, secret key, or database URL behind `NEXT_PUBLIC_`.

A quick mental check before adding `NEXT_PUBLIC_`:

```text
Is this value safe to print on a public billboard?
  Yes -> NEXT_PUBLIC_ is fine.
  No  -> keep it server-only (no prefix).
```

## Never Commit `.env.local`

Because `.env.local` holds your real secrets, it must never go into git. If you
push it to a public repository, your keys are leaked to the world.

Next.js's default `.gitignore` already ignores it. Confirm yours has these
lines:

```text
# .gitignore
# Ignore all local env files (they contain secrets)
.env*.local
```

To share which variables are needed **without** sharing the values, commit a
sample file with blank or fake values:

```text
# .env.example  (safe to commit — no real secrets)
DATABASE_URL=
STRIPE_SECRET_KEY=
NEXT_PUBLIC_API_URL=https://api.example.com
```

A new teammate copies it and fills in real values:

```bash
# Copy the example, then edit .env.local with real secrets.
cp .env.example .env.local
```

## Environment Variables in Deployment

On your laptop the values come from `.env.local`. But that file is **not**
uploaded when you deploy. So where do production values come from? You set them
in your hosting provider's settings.

On **Vercel**, for example, you add them in the dashboard:

```text
Project -> Settings -> Environment Variables
  Name:  DATABASE_URL
  Value: postgresql://...the production database...
  Environments: Production, Preview, Development (choose which apply)
```

Vercel injects these into `process.env` when your app runs in the cloud. The
same `process.env.DATABASE_URL` line works everywhere — only the value differs.

A few deployment tips:

- After adding or changing a variable in the dashboard, **redeploy** so the new
  value takes effect.
- Use separate values for **Preview** (test) and **Production** when it matters,
  like pointing previews at a non-production database.
- `NEXT_PUBLIC_` values are read at **build time**, so changing one requires a
  rebuild, not just a restart.

## Common Mistakes

- **Putting a secret behind `NEXT_PUBLIC_`.** It becomes visible to every
  visitor. Only public, non-secret values get that prefix.
- **Forgetting to restart the dev server** after editing a `.env` file. Next.js
  reads them on startup only.
- **Committing `.env.local`.** This leaks secrets. Keep it in `.gitignore` and
  commit a `.env.example` instead.
- **Expecting server-only vars in the browser.** They are `undefined` in Client
  Components by design.
- **Treating env values as non-strings.** `process.env.PORT` is a string;
  convert it with `Number(...)` or compare with `=== "true"`.
- **Forgetting to set the variable in the deployment dashboard.** It works
  locally but crashes in production because the value is missing there.

## Exercises

1. List which of these belong behind `NEXT_PUBLIC_` and which must stay
   server-only: a Stripe secret key, a public Google Maps embed URL, a database
   password, a public analytics site ID.
2. Write a `.env.local` containing a `DATABASE_URL` and a
   `NEXT_PUBLIC_SITE_NAME`. Then write a one-line `.env.example` for each.
3. Write a server function that reads `process.env.MAX_UPLOAD_MB`, converts it
   to a number with a default of `5`, and returns it.
4. Explain in one sentence what happens if you read a server-only variable
   inside a `"use client"` component.
5. Describe the steps to add a new secret to a Vercel-deployed app and make it
   take effect.

## Key Takeaways

- Environment variables hold secrets and per-environment config, kept out of
  your code.
- Next.js loads `.env*` files; put real secrets in **`.env.local`**.
- Read values with `process.env.NAME` (always a string or `undefined`).
- Server-only by default; add **`NEXT_PUBLIC_`** only for values safe to expose.
- Never commit `.env.local`; share a `.env.example` with blank values instead.
- In production, set variables in your host's dashboard (e.g. Vercel) and
  redeploy.

---

[← Previous: Working with a Database](01-databases-overview.md) | [Back to Course Index](../README.md) | [Next: Client State Management →](03-client-state-management.md)
