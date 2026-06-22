# Error Handling in Production

Things go wrong. A database is down, a user types a bad URL, a third-party API
times out. In a toy project you might let the app crash. In **production** (the
real, live version that strangers use) a crash looks scary and unprofessional.

Think of error handling like the airbags and seatbelts in a car. You hope you
never need them, but when something goes wrong they protect the people inside.
Next.js gives you a few "airbags" built right into the App Router, plus some
habits that keep your app calm when the unexpected happens.

## What You'll Learn

- How `error.tsx`, `not-found.tsx`, and `global-error.tsx` catch problems
- The difference between **expected** errors and **unexpected** errors
- How to throw and catch errors in Server Components and Server Actions
- Why you show friendly messages to users but log the full details for yourself
- How to use `notFound()` and `redirect()` on purpose
- Why returning an error from a Server Action is often better than throwing one
- How to avoid leaking secret or technical details to users

## A Quick Recap of the Three Error Files

You met these files earlier in the routing module. Here is the short version.

### `error.tsx` — catches errors in one part of the app

If a Server Component throws an error while rendering, Next.js looks for the
nearest `error.tsx` and shows it instead of a blank screen. It must be a Client
Component because it uses state and a button.

```tsx
// app/dashboard/error.tsx
"use client"; // error files are always Client Components

export default function Error({
  error, // the error that was thrown
  reset, // a function that tries to re-render the page
}: {
  error: Error;
  reset: () => void;
}) {
  return (
    <div>
      <h2>Something went wrong.</h2>
      {/* Friendly message only. We do NOT print error.message here. */}
      <button onClick={() => reset()}>Try again</button>
    </div>
  );
}
```

### `not-found.tsx` — shown when something does not exist

This page appears when you call the `notFound()` function (more on that below) or
when a user visits a route that does not exist.

```tsx
// app/not-found.tsx
import Link from "next/link";

export default function NotFound() {
  return (
    <div>
      <h2>Page not found</h2>
      <p>We could not find what you were looking for.</p>
      <Link href="/">Go back home</Link>
    </div>
  );
}
```

### `global-error.tsx` — the last line of defence

A normal `error.tsx` can not catch a crash in your root layout. For that rare
case there is `global-error.tsx`, which replaces the entire page, so it must
include its own `<html>` and `<body>` tags.

```tsx
// app/global-error.tsx
"use client";

export default function GlobalError({ reset }: { reset: () => void }) {
  return (
    <html>
      <body>
        <h2>The app crashed.</h2>
        <button onClick={() => reset()}>Reload</button>
      </body>
    </html>
  );
}
```

## Expected vs Unexpected Errors

This is the most useful idea in the whole lesson. Sort every error into one of
two buckets.

- **Expected errors** are things you can predict: the form is missing a field,
  the email is already taken, the user is not logged in. You should handle these
  calmly and tell the user what to fix.
- **Unexpected errors** are surprises: the database connection dropped, a library
  threw something you did not foresee. These are what `error.tsx` is for.

The rule of thumb: **handle expected errors yourself, and let unexpected errors
bubble up** to the error boundary.

## Throwing Errors in Server Components

A Server Component can throw an error. The nearest `error.tsx` catches it.

```tsx
// app/products/[id]/page.tsx
import { getProduct } from "@/lib/products";

export default async function ProductPage({
  params,
}: {
  params: Promise<{ id: string }>;
}) {
  const { id } = await params;
  const product = await getProduct(id);

  if (!product) {
    // This is an UNEXPECTED render failure if it should always exist...
    // ...but for "missing", prefer notFound() (shown below).
    throw new Error("Product could not be loaded");
  }

  return <h1>{product.name}</h1>;
}
```

## `notFound()` and `redirect()`

Next.js gives you two helper functions for two very common, *expected*
situations. Both work by throwing a special signal that Next.js understands.

```tsx
// app/products/[id]/page.tsx
import { notFound, redirect } from "next/navigation";
import { getProduct } from "@/lib/products";

export default async function ProductPage({
  params,
}: {
  params: Promise<{ id: string }>;
}) {
  const { id } = await params;

  if (id === "old-page") {
    redirect("/products"); // send the user somewhere else
  }

  const product = await getProduct(id);
  if (!product) {
    notFound(); // shows the nearest not-found.tsx
  }

  return <h1>{product.name}</h1>;
}
```

Use `notFound()` for "this thing does not exist" and `redirect()` for "go over
there instead". Do not wrap these in a `try/catch`, because catching the special
signal would stop them from working.

## Server Actions: Return Errors Instead of Throwing

In a Server Action (a function that runs on the server when a form submits),
throwing an error sends the user to the error page. That is fine for true
surprises, but for *expected* problems it is friendlier to **return** an error
object and show it next to the form.

```tsx
// app/actions.ts
"use server";

export async function createUser(formData: FormData) {
  const email = String(formData.get("email") ?? "");

  // Expected error: tell the user, do not crash the page.
  if (!email.includes("@")) {
    return { error: "Please enter a valid email address." };
  }

  try {
    await saveUser(email);
    return { success: true };
  } catch (err) {
    // Unexpected error: log the real details for us...
    console.error("createUser failed:", err);
    // ...but return a calm, generic message to the user.
    return { error: "Something went wrong. Please try again." };
  }
}
```

The form reads that returned value with the `useActionState` hook you saw in the
features module, and shows the message inline.

## Friendly Messages vs Leaking Details

Never show raw error text to users. Messages like
`Error: ECONNREFUSED 10.0.0.5:5432` confuse normal people and hand attackers
clues about your system.

```tsx
// Bad: leaks internal details to the user
return <p>{error.message}</p>;

// Good: a calm, generic message
return <p>Sorry, we could not complete that. Please try again.</p>;
```

## Logging Errors

Friendly messages help the user, but **you** still need the real details to fix
the bug. That is what logging is for. At minimum, write the error to the server
console; in production you would send it to a logging service.

```ts
// lib/log.ts
export function logError(where: string, error: unknown) {
  // In real apps, send this to a service (Sentry, Logtail, etc.).
  console.error(`[${where}]`, error);
}
```

```tsx
// app/dashboard/error.tsx
"use client";
import { useEffect } from "react";

export default function Error({ error }: { error: Error }) {
  useEffect(() => {
    // Report the error so we can investigate it later.
    console.error(error);
  }, [error]);

  return <h2>Something went wrong on the dashboard.</h2>;
}
```

## Common Mistakes

- **Printing `error.message` to the user.** It leaks technical or secret details.
  Show a generic message and log the real one.
- **Wrapping `notFound()` or `redirect()` in `try/catch`.** They work by throwing
  a special value; catching it breaks them.
- **Throwing for every small problem.** Expected problems (bad input, duplicate
  email) are nicer to return from a Server Action and show inline.
- **Forgetting that `error.tsx` must be a Client Component.** It needs `"use
  client"` because it uses the `reset` button and effects.
- **Never logging anything.** If you only show users a friendly message and log
  nothing, you will never know what to fix.

## Exercises

1. Add an `error.tsx` to a route and force an error by throwing inside the page.
   Confirm the friendly UI appears and that the "Try again" button calls `reset`.
2. Add a `not-found.tsx`, then call `notFound()` in a dynamic route when an item
   does not exist. Visit a missing item and watch it appear.
3. Write a Server Action that returns `{ error: "..." }` for an invalid input and
   `{ success: true }` otherwise. Do not throw for the invalid case.
4. In an `error.tsx`, add a `useEffect` that logs the error to the console, and
   confirm the user still only sees a calm message.
5. Sort five errors from a real app into "expected" and "unexpected", then decide
   for each whether to return, call `notFound()`/`redirect()`, or let it throw.

## Key Takeaways

- `error.tsx`, `not-found.tsx`, and `global-error.tsx` are your built-in safety
  nets for unexpected crashes.
- Separate **expected** errors (handle calmly) from **unexpected** ones (let them
  reach the error boundary).
- Use `notFound()` and `redirect()` for "missing" and "go elsewhere"; never wrap
  them in `try/catch`.
- In Server Actions, **return** expected errors and show them inline instead of
  throwing.
- Show users friendly, generic messages, but **log** the full details for
  yourself.

---

[← Previous: Client State Management](../05-data-and-state/03-client-state-management.md) | [Back to Course Index](../README.md) | [Next: Performance and Optimization →](02-performance-optimization.md)
