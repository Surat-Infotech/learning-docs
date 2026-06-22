# Caching and Revalidation

Fetching data is easy. The harder question is: how **fresh** does that data need
to be, and how often should Next.js go get it again? This is where caching and
revalidation come in.

Think of a printed menu at a cafe. A menu that almost never changes can be
printed once and reused for months (cheap and fast). A menu with today's specials
needs reprinting every morning. And a screen showing live wait times must update
constantly. Your pages are the same: some can be built once, some refreshed on a
schedule, and some must be freshly made on every visit.

## What You'll Learn

- The difference between **static** and **dynamic** rendering in plain words
- How `fetch` caching works with `cache: 'force-cache'` and `cache: 'no-store'`
- Time-based revalidation with `next: { revalidate: 60 }`
- The route-level settings `export const revalidate` and `export const dynamic`
- On-demand revalidation with `revalidatePath` and `revalidateTag`
- What **ISR** (Incremental Static Regeneration) means
- Practical advice that works whether you are on Next.js 14 or 15

## Static vs Dynamic Rendering

There are two ways Next.js can produce a page.

- **Static rendering**: the page is built **once** (at build time or the first
  request) and the same HTML is reused for everyone. Fast and cheap, like the
  printed menu.
- **Dynamic rendering**: the page is built **fresh on every request**. Slower,
  but always current, like the live wait-times screen.

```text
Static  ->  built once  ->  reused for many visitors      (fast, may be stale)
Dynamic ->  built each request  ->  always up to date      (slower, fresh)
```

Next.js tries to make pages static by default because it is faster. You choose
dynamic when the data must always be current.

## Caching With `fetch`

When you call `fetch` in a Server Component, Next.js can **cache** the result so
it does not have to ask the API again next time. You control this with the
`cache` option.

### `cache: 'no-store'` — always fresh

```tsx
// app/prices/page.tsx
export default async function PricesPage() {
  // Never cache. Go fetch the latest data on every request.
  const res = await fetch("https://api.example.com/prices", {
    cache: "no-store",
  });
  const prices = await res.json();

  return <p>Latest price: {prices.btc}</p>;
}
```

Use `no-store` for data that changes constantly or is personal to the user (like
a shopping cart or a stock price). This makes the page render dynamically.

### `cache: 'force-cache'` — reuse the result

```tsx
// app/about/page.tsx
export default async function AboutPage() {
  // Cache the result and reuse it. Good for data that rarely changes.
  const res = await fetch("https://api.example.com/company", {
    cache: "force-cache",
  });
  const info = await res.json();

  return <p>{info.description}</p>;
}
```

`force-cache` keeps the result around and serves it again instead of re-fetching.

> A note on defaults: the **default** caching behavior of `fetch` changed between
> Next.js 14 and 15 (older versions cached more aggressively by default). Do not
> rely on the default. If you care about the behavior, **state it explicitly**
> with `cache: 'no-store'` or `cache: 'force-cache'`. Then your code does the
> same thing on every version.

## Time-Based Revalidation

Often you want something in between: cache the data, but refresh it every so
often. That is **time-based revalidation**, set with `next: { revalidate }`.

```tsx
// app/news/page.tsx
export default async function NewsPage() {
  // Cache this, but consider it stale after 60 seconds.
  const res = await fetch("https://api.example.com/news", {
    next: { revalidate: 60 }, // seconds
  });
  const news = await res.json();

  return <h1>{news.headline}</h1>;
}
```

Here is what `revalidate: 60` does in plain terms: serve the cached version, and
at most once every 60 seconds, fetch a fresh copy in the background for the next
visitors. This is the printed-menu-reprinted-each-morning idea, but on a timer.

## Route Segment Configs

Instead of (or alongside) per-fetch options, you can set behavior for a whole
route by `export`ing special values from the page or layout file.

### `export const revalidate`

```tsx
// app/blog/page.tsx
// Refresh this whole route's cached data every 5 minutes.
export const revalidate = 300; // seconds

export default async function BlogPage() {
  const res = await fetch("https://api.example.com/posts");
  const posts = await res.json();
  return <PostList posts={posts} />;
}
```

### `export const dynamic`

```tsx
// app/dashboard/page.tsx
// Force this route to render dynamically (fresh on every request).
export const dynamic = "force-dynamic";

export default async function Dashboard() {
  const res = await fetch("https://api.example.com/me");
  const me = await res.json();
  return <p>Hello, {me.name}</p>;
}
```

Common values for `dynamic`:

- `"force-dynamic"` — always render fresh on each request (nothing cached).
- `"force-static"` — render once and reuse, even when the data could be dynamic.

Use these when you want to set the policy for the entire page in one place.

## On-Demand Revalidation

Time-based revalidation is great, but sometimes you know **exactly when** data
changed, for example right after a user submits a form. Then you do not want to
wait for a timer; you want to refresh **now**. That is on-demand revalidation.

```tsx
// app/actions.ts
"use server";
import { revalidatePath, revalidateTag } from "next/cache";

export async function publishPost(formData: FormData) {
  await savePost(formData); // imagine this writes to the database

  // Tell Next.js: the cached data for these is stale, rebuild it.
  revalidatePath("/blog");      // refresh a specific route
  revalidateTag("posts");       // refresh anything tagged "posts"
}
```

- `revalidatePath("/blog")` clears the cache for that route so the next visit
  rebuilds it with fresh data.
- `revalidateTag("posts")` clears every `fetch` you tagged with `"posts"`. You
  add a tag like this:

```tsx
// Tag a fetch so you can revalidate it later by name.
const res = await fetch("https://api.example.com/posts", {
  next: { tags: ["posts"] },
});
```

We will use these in depth in the [Server Actions](05-server-actions.md) lesson.

## What Is ISR?

**ISR** stands for **Incremental Static Regeneration**. It is just the combination
you have already seen: a static page that **regenerates itself over time** (with
`revalidate`) or **on demand** (with `revalidatePath`/`revalidateTag`).

```text
ISR = static speed  +  the ability to refresh without a full rebuild
```

So you get the speed of a printed menu, but it quietly reprints when the timer
hits or when you tell it to. You do not redeploy the whole site to update one
page.

## Common Mistakes

- **Relying on the default caching behavior.** It differs between Next.js 14 and
  15. Always state `cache` or `revalidate` explicitly when it matters.
- **Using `no-store` everywhere.** This makes every page dynamic and slow. Cache
  what can be cached; only opt out for truly fresh or personal data.
- **Setting a tiny `revalidate` like `1` to "stay fresh".** That nearly defeats
  caching. If you need instant freshness after a change, use on-demand
  revalidation instead.
- **Forgetting to revalidate after a mutation.** If you write data but never call
  `revalidatePath`/`revalidateTag`, users may keep seeing the old cached page.
- **Mixing conflicting settings.** `cache: 'force-cache'` on a fetch inside a
  `force-dynamic` route is confusing; pick one clear policy.

## Exercises

1. Build a page that fetches with `cache: 'no-store'` and another that fetches
   with `cache: 'force-cache'`. Note how often each re-fetches when you reload.
2. Add `next: { revalidate: 30 }` to a fetch and explain in a comment what that
   means in your own words.
3. Use `export const revalidate = 60` at the top of a route and remove the
   per-fetch option. Confirm the route still refreshes on the timer.
4. Tag a fetch with `next: { tags: ["items"] }`, then write a small Server Action
   that calls `revalidateTag("items")`.
5. Write a one-paragraph note describing when you would choose static vs dynamic
   rendering for a real page (for example, a blog post vs a live scoreboard).

## Key Takeaways

- **Static** pages are built once and reused (fast); **dynamic** pages are built
  fresh each request (always current).
- Control `fetch` caching with `cache: 'force-cache'` (reuse) or
  `cache: 'no-store'` (always fresh).
- Refresh cached data on a timer with `next: { revalidate: seconds }` or
  `export const revalidate`.
- Force whole-route behavior with `export const dynamic = 'force-dynamic' |
  'force-static'`.
- Refresh immediately after a change with `revalidatePath` and `revalidateTag`;
  this on-timer-or-on-demand refresh of static pages is called **ISR**.
- Defaults differ across Next.js versions, so set your caching options
  explicitly.

---

[← Previous: Data Fetching](02-data-fetching.md) | [Back to Course Index](../README.md) | [Next: Streaming and Suspense →](04-streaming-and-suspense.md)
