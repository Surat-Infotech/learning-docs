# Loading and Error UI

Real apps fetch data, and fetching takes time. Real apps also hit problems: a
server is down, a post does not exist, a value is missing. A polished site
handles both gracefully, showing a loading spinner while it waits and a friendly
message when something breaks, instead of a blank screen or a scary crash.
Next.js gives you special files for exactly this. Think of them like a restaurant:
`loading.tsx` is the "your food is on the way" note, `error.tsx` is the manager
who apologizes when the kitchen messes up, and `not-found.tsx` is the sign for a
table that does not exist.

## What You'll Learn

- How `loading.tsx` shows an instant loading state while a route loads
- How loading uses React Suspense behind the scenes
- How `error.tsx` catches errors in a route segment
- Why `error.tsx` must be a client component, and its `error` and `reset` props
- How `not-found.tsx` and the `notFound()` function handle missing pages
- What `global-error.tsx` is for
- Worked examples of each

## Loading UI with `loading.tsx`

Add a `loading.tsx` next to a `page.tsx`, and Next.js shows it **automatically**
while that page is loading (for example, while its data is being fetched on the
server). The moment you navigate, the loading UI appears instantly, then swaps to
the real page when it is ready.

```tsx
// app/blog/loading.tsx
export default function Loading() {
  // shown while /blog is preparing
  return <p>Loading posts...</p>;
}
```

```tsx
// app/blog/page.tsx
async function getPosts() {
  // pretend this is a slow data fetch
  await new Promise((r) => setTimeout(r, 1500));
  return ["First", "Second"];
}

export default async function BlogPage() {
  const posts = await getPosts(); // takes ~1.5s
  return (
    <ul>
      {posts.map((p) => (
        <li key={p}>{p}</li>
      ))}
    </ul>
  );
}
```

For about 1.5 seconds you see "Loading posts...", then the list replaces it. You
wrote no spinner logic and no state; the file's presence is enough.

### How It Works: Suspense

Under the hood, Next.js wraps your page in a React **Suspense boundary** using
your `loading.tsx` as the fallback. Suspense is React's way of saying "show this
placeholder until the real content is ready." You do not write the Suspense code;
`loading.tsx` becomes the fallback for that route segment for free.

Because it is tied to the segment, a `loading.tsx` in `app/blog/` covers `/blog`
and everything nested under it.

## Error UI with `error.tsx`

If a page (or its data fetch) throws an error, Next.js catches it and shows the
nearest `error.tsx` instead of crashing the whole app. This file **must be a
client component**, so it starts with `"use client"`. It receives two props:
`error` (the error that happened) and `reset` (a function to try rendering the
segment again).

```tsx
// app/blog/error.tsx
"use client"; // error files must be client components

export default function Error({
  error,
  reset,
}: {
  error: Error;
  reset: () => void; // re-attempts rendering the segment
}) {
  return (
    <div>
      <h2>Something went wrong</h2>
      <p>{error.message}</p>
      <button onClick={() => reset()}>Try again</button>
    </div>
  );
}
```

```tsx
// app/blog/page.tsx
export default async function BlogPage() {
  const ok = false;
  if (!ok) {
    throw new Error("Failed to load posts"); // caught by error.tsx
  }
  return <h1>Posts</h1>;
}
```

When the page throws, the user sees the error UI with a "Try again" button.
Clicking it calls `reset()`, which re-renders the segment, useful if the error
was temporary (a flaky network, say).

Two things to remember:

- `error.tsx` catches errors in its **own segment and below**, but not errors in
  the layout *above* it in the same folder. To catch a layout's error, the
  `error.tsx` must sit one level up.
- It must be a client component because it uses interactivity (the reset button)
  and runs in the browser to recover.

## Handling Missing Pages: `not-found.tsx` and `notFound()`

When a resource does not exist, like a blog post with an unknown slug, you should
show a proper "404 Not Found" page rather than a broken one. Two pieces work
together:

- `not-found.tsx` defines the 404 UI for a segment.
- The `notFound()` function (from `next/navigation`) triggers it from your code.

```tsx
// app/blog/[slug]/not-found.tsx
export default function NotFound() {
  return <h2>That post could not be found.</h2>;
}
```

```tsx
// app/blog/[slug]/page.tsx
import { notFound } from "next/navigation";

const posts = [{ slug: "hello-world", title: "Hello World" }];

export default async function PostPage({
  params,
}: {
  params: Promise<{ slug: string }>;
}) {
  const { slug } = await params;
  const post = posts.find((p) => p.slug === slug);

  if (!post) {
    notFound(); // stops rendering, shows not-found.tsx
  }

  return <h1>{post!.title}</h1>;
}
```

Visiting `/blog/does-not-exist` calls `notFound()`, which halts rendering and
shows the `not-found.tsx` UI (and sends a real 404 status). Like `redirect()`,
`notFound()` throws internally, so code after it does not run.

A `not-found.tsx` placed directly in `app/` also serves as the site-wide 404 for
URLs that match no route at all.

## Catching Everything: `global-error.tsx`

Regular `error.tsx` files cannot catch an error in the **root layout** itself,
because they render *inside* that layout. For that rare case, add a
`global-error.tsx` in `app/`. It replaces the entire page, so it must include its
own `<html>` and `<body>` tags.

```tsx
// app/global-error.tsx
"use client";

export default function GlobalError({
  error,
  reset,
}: {
  error: Error;
  reset: () => void;
}) {
  return (
    <html>
      <body>
        <h2>Something went very wrong.</h2>
        <button onClick={() => reset()}>Reload</button>
      </body>
    </html>
  );
}
```

You will rarely need this; it is the last line of defence for catastrophic,
whole-app errors. Most of the time, a per-segment `error.tsx` is enough.

## Common Mistakes

- **Forgetting `"use client"` in `error.tsx`.** Error and global-error files must
  be client components, or they will not work.
- **Expecting `error.tsx` to catch its sibling layout's error.** It cannot catch
  the layout at the same level. Place it one folder up to do that.
- **Writing code after `notFound()` or `redirect()` expecting it to run.** Both
  throw internally and stop execution at that line.
- **Omitting `<html>`/`<body>` from `global-error.tsx`.** It replaces the whole
  document, so it needs its own root tags.
- **Building a manual spinner with state when `loading.tsx` would do.** For
  route-level loading, the file is simpler and instant.

## Exercises

1. Add a `loading.tsx` to a route whose page `await`s a 2-second fake fetch.
   Navigate to it and confirm the loading UI shows, then the content.
2. Make a page that `throw`s an error, add an `error.tsx` with a "Try again"
   button, and confirm clicking `reset()` re-renders the segment.
3. Build a `app/blog/[slug]/page.tsx` that calls `notFound()` for unknown slugs,
   plus a `not-found.tsx`. Test a valid slug and an invalid one.
4. Add a top-level `app/not-found.tsx` and visit a completely made-up URL to see
   it serve as the site 404.
5. Move an `error.tsx` one folder up and trigger an error in the parent layout to
   see the difference in what gets caught.

## Key Takeaways

- `loading.tsx` shows an **instant loading state** for a route segment, powered
  by React **Suspense**, with no spinner code required.
- `error.tsx` catches errors in its segment, **must be a client component**, and
  receives `error` plus a `reset()` function to retry.
- `not-found.tsx` defines the 404 UI, and `notFound()` (from `next/navigation`)
  triggers it; both `notFound()` and `redirect()` stop rendering when called.
- A top-level `app/not-found.tsx` serves as the site-wide 404 for unmatched URLs.
- `global-error.tsx` is the last-resort handler for errors in the root layout and
  must include its own `<html>` and `<body>`.

---

[← Previous: Route Groups and Nested Layouts](05-route-groups-nested-layouts.md) | [Back to Course Index](../README.md) | [Next: Server and Client Components →](../03-rendering-and-data/01-server-and-client-components.md)
