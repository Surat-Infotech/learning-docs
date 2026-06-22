# Data Fetching in Server Components

In older React apps, getting data usually meant writing a `useEffect`, juggling
loading flags, and watching the page flicker. In the App Router it is much
simpler: because Server Components run on the server, you can just `await` your
data right where you use it.

Think of it like ordering at a counter where the food is already cooked. You ask,
you wait a moment, and you get the finished plate. No callbacks, no flicker, no
"is it ready yet?" checks scattered around.

## What You'll Learn

- How to fetch data directly inside an `async` Server Component
- Why you no longer need `useEffect` for the initial data
- How to fetch from an external API and from your own database
- How to pass fetched data down into Client Components as props
- The difference between **sequential** and **parallel** fetching (and why it matters)
- How to handle errors with `try/catch` and the `notFound()` helper
- A complete worked example: a product list page and a product detail page

## Fetching Right in the Component

A Server Component can be `async`, so you can `await fetch(...)` and use the
result immediately.

```tsx
// app/products/page.tsx
export default async function ProductsPage() {
  // Wait for the data on the server, before sending HTML.
  const res = await fetch("https://api.example.com/products");
  const products = await res.json();

  return (
    <ul>
      {products.map((p: { id: number; name: string }) => (
        <li key={p.id}>{p.name}</li>
      ))}
    </ul>
  );
}
```

No `useState`, no `useEffect`, no loading flag. The page that reaches the browser
already contains the products.

### Why no `useEffect`?

`useEffect` runs **in the browser, after** the page is shown. That is why old
apps flickered: first an empty page, then the data popped in. A Server Component
fetches **before** the HTML is sent, so the user sees the real content right
away. Save `useEffect` for things that genuinely happen after load, like
reacting to a click.

## Fetching From Your Own Database

You are not limited to `fetch`. Because the code runs on the server, you can call
your database directly.

```tsx
// app/users/page.tsx
import { db } from "@/lib/db"; // your database client

export default async function UsersPage() {
  // A direct database query. Safe here because it runs on the server.
  const users = await db.user.findMany();

  return (
    <ul>
      {users.map((u) => (
        <li key={u.id}>{u.name}</li>
      ))}
    </ul>
  );
}
```

Whether you use `fetch` to an external API or a database client, the shape is the
same: `await` it, then render.

## Passing Data to Client Components

Sometimes the data is needed by an interactive piece. The Server Component
fetches it, then passes it down as a prop. (Remember from the last lesson: data
flows **down** from server to client.)

```tsx
// app/profile/page.tsx  (Server Component)
import EditableName from "./EditableName";

export default async function ProfilePage() {
  const user = await db.user.findFirst(); // fetched on the server

  // Hand the server data to the client component.
  return <EditableName initialName={user.name} />;
}
```

```tsx
// app/profile/EditableName.tsx  (Client Component)
"use client";
import { useState } from "react";

export default function EditableName({ initialName }: { initialName: string }) {
  const [name, setName] = useState(initialName); // seeded by server data

  return (
    <input value={name} onChange={(e) => setName(e.target.value)} />
  );
}
```

## Sequential vs Parallel Fetching

When you need **two** pieces of data, the order of your `await`s matters.

### Sequential (often slower than it needs to be)

```tsx
// app/dashboard/page.tsx
export default async function Dashboard() {
  // These run one after another. Total time = first + second.
  const user = await getUser();     // wait... done
  const orders = await getOrders(); // only NOW does this start

  return <Summary user={user} orders={orders} />;
}
```

If `getUser` takes 1 second and `getOrders` takes 1 second, the page waits about
**2 seconds**, even though neither needs the other.

### Parallel (faster)

Start both requests, then `await` them together with `Promise.all`.

```tsx
// app/dashboard/page.tsx
export default async function Dashboard() {
  // Kick off BOTH at the same time...
  const userPromise = getUser();
  const ordersPromise = getOrders();

  // ...then wait for whichever finishes last.
  const [user, orders] = await Promise.all([userPromise, ordersPromise]);

  return <Summary user={user} orders={orders} />;
}
```

Now the total wait is about **1 second**, the time of the slowest one. Use
parallel fetching whenever the requests do not depend on each other. Use
sequential only when the second request needs the result of the first (for
example, fetching orders for a user ID you got from the first call).

## Handling Errors

Two tools cover most cases: `try/catch` for unexpected failures, and `notFound()`
for "this thing does not exist".

```tsx
// app/products/[id]/page.tsx
import { notFound } from "next/navigation";

export default async function ProductPage({
  params,
}: {
  params: Promise<{ id: string }>;
}) {
  const { id } = await params; // params is awaited in Next 15

  const res = await fetch(`https://api.example.com/products/${id}`);

  // If the API says "not found", show the 404 page.
  if (res.status === 404) {
    notFound(); // stops here and renders not-found.tsx
  }

  if (!res.ok) {
    // For other failures, throw so error.tsx can catch it.
    throw new Error("Failed to load product");
  }

  const product = await res.json();
  return <h1>{product.name}</h1>;
}
```

`notFound()` immediately stops rendering and shows your `not-found.tsx` page.
Throwing an error triggers the nearest `error.tsx`. (Both files come from the
[Loading and Error UI](../02-routing/06-loading-and-error-ui.md) lesson.)

## Worked Example: List Page and Detail Page

Let's tie it together with a tiny blog. We have a list of posts and a page for a
single post.

```tsx
// app/blog/page.tsx  (the list)
import Link from "next/link";

type Post = { id: number; title: string };

export default async function BlogPage() {
  const res = await fetch("https://api.example.com/posts");
  if (!res.ok) throw new Error("Could not load posts");

  const posts: Post[] = await res.json();

  return (
    <main>
      <h1>Blog</h1>
      <ul>
        {posts.map((post) => (
          <li key={post.id}>
            {/* Link to the detail page for this post */}
            <Link href={`/blog/${post.id}`}>{post.title}</Link>
          </li>
        ))}
      </ul>
    </main>
  );
}
```

```tsx
// app/blog/[id]/page.tsx  (the detail)
import { notFound } from "next/navigation";

type Post = { id: number; title: string; body: string };

export default async function PostPage({
  params,
}: {
  params: Promise<{ id: string }>;
}) {
  const { id } = await params;

  const res = await fetch(`https://api.example.com/posts/${id}`);
  if (res.status === 404) notFound();
  if (!res.ok) throw new Error("Could not load post");

  const post: Post = await res.json();

  return (
    <article>
      <h1>{post.title}</h1>
      <p>{post.body}</p>
    </article>
  );
}
```

The list page links to each detail page, and the detail page fetches just the one
post it needs. Clean, no `useEffect`, errors handled.

## Common Mistakes

- **Reaching for `useEffect` to load the initial data.** In a Server Component
  you do not need it; just `await` the fetch.
- **Awaiting fetches one by one when they do not depend on each other.** This
  makes pages slow. Use `Promise.all` to run them in parallel.
- **Forgetting to check `res.ok`.** A failed `fetch` does not throw on its own,
  so you must check and handle the bad response yourself.
- **Using `notFound()` for every error.** Use it only for "does not exist"; throw
  for real failures so `error.tsx` can show a proper message.
- **Trying to fetch the same way inside a Client Component.** Client Components
  can not be `async`; fetch on the server and pass the data down.

## Exercises

1. Make a Server Component page that fetches a list from any public API (for
   example `https://jsonplaceholder.typicode.com/users`) and renders the names.
2. Add a detail page at `app/users/[id]/page.tsx` that fetches and shows a single
   user, using `notFound()` when the user does not exist.
3. Take a page that fetches two independent things sequentially and rewrite it to
   fetch them in parallel with `Promise.all`.
4. Fetch a value on the server and pass it as a prop into a Client Component that
   lets the user edit it.
5. Add `try/catch`-style handling: check `res.ok`, throw on failure, and create an
   `error.tsx` to confirm it shows.

## Key Takeaways

- Server Components can be `async`, so you fetch data with a simple `await`.
- You do not need `useEffect` for the initial data, which removes the old
  loading flicker.
- You can fetch from an external API or query your database directly on the
  server.
- Run independent requests in **parallel** with `Promise.all`; only go
  **sequential** when one depends on another.
- Handle "does not exist" with `notFound()` and real failures by throwing to
  `error.tsx`.

---

[← Previous: Server and Client Components](01-server-and-client-components.md) | [Back to Course Index](../README.md) | [Next: Caching and Revalidation →](03-caching-and-revalidation.md)
