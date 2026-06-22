# The Next.js Caching Layers

Caching in the App Router is not a single feature you turn on. It is four distinct caches operating at different layers of the stack, each with its own lifetime, scope, and invalidation rules. When you understand them as separate systems, the "why is my data stale?" and "why is this re-rendering?" questions stop being mysteries and become a matter of asking *which cache is involved*.

At scale this matters enormously: the difference between a route that hits a database on every request and one served from a pre-rendered HTML cache is the difference between paying for thousands of DB connections and serving from the edge for free. But the same caches that make you fast will happily serve a user yesterday's bank balance if you don't model them deliberately.

## What You'll Learn

- The four caches Next.js maintains and the layer each one lives at
- Where each cache stores data, its scope, and its lifetime
- How a single request flows through all four caches in order
- Why Request Memoization is not the same as the Data Cache
- How the Full Route Cache and Router Cache interact during navigation
- The Next 15 default change: `fetch` and GET Route Handlers are no longer cached by default
- Both mental models — opt-in (Next 15) and opt-out (Next 13/14) — so you can reason about either version

## The Four Caches at a Glance

```text
                         ┌─────────────────────────────────────────┐
   BROWSER               │   Client-side Router Cache               │
                         │   • in-memory, per session               │
                         │   • caches RSC payloads for visited routes│
                         └─────────────────────────────────────────┘
                                          ▲
                                          │  navigation / prefetch
══════════════════════════════════════════════════════════════════ network
                                          │
                         ┌─────────────────────────────────────────┐
   SERVER (build/runtime)│   Full Route Cache                       │
                         │   • persistent on server/build artifacts │
                         │   • caches rendered HTML + RSC payload    │
                         └─────────────────────────────────────────┘
                                          ▲
                                          │  render a route segment
                         ┌─────────────────────────────────────────┐
                         │   Request Memoization                    │
                         │   • per single render pass (one request) │
                         │   • dedupes identical fetch() calls       │
                         └─────────────────────────────────────────┘
                                          ▲
                                          │  fetch() / cached function
                         ┌─────────────────────────────────────────┐
                         │   Data Cache                             │
                         │   • persistent across requests/deploys   │
                         │   • stores the result of data fetches    │
                         └─────────────────────────────────────────┘
                                          ▲
                                          │
                                   ORIGIN (DB, API, CMS)
```

Two of these live on the client (Router Cache), or are about rendered output (Full Route Cache), and two are about *data* (Request Memoization, Data Cache). Keep that split in your head: **route caches** vs **data caches**.

## 1. Request Memoization

Request Memoization deduplicates identical `fetch()` calls **within a single render pass**. If five components in one tree all call `fetch('/api/user/42')` with the same options, the actual network request fires once; the other four get the memoized result.

```tsx
// app/lib/get-user.ts
// This function may be called from layout, page, and several
// components during ONE render — fetch() is memoized for all of them.
export async function getUser(id: string) {
  // React extends fetch; identical calls in one request dedupe.
  const res = await fetch(`https://api.example.com/users/${id}`);
  return res.json();
}
```

Key properties:

- **Scope:** a single server request / render pass. It does not persist between requests.
- **What it caches:** only `fetch` with identical URL + options (React patches `fetch`).
- **Lifetime:** the lifetime of the render. Discarded when the response is sent.
- **Why it exists:** so you can colocate data fetching in the components that need it without manually threading props or worrying about N duplicate calls.

For non-`fetch` data sources (a database client, an ORM), use React's `cache()` to get the same per-request dedupe:

```ts
// app/lib/queries.ts
import { cache } from 'react';
import { db } from './db';

// cache() memoizes by arguments for the duration of one request.
export const getPostById = cache(async (id: string) => {
  return db.post.findUnique({ where: { id } });
});
```

## 2. The Data Cache

The Data Cache is the persistent one. It stores the *result* of data fetches **across requests, and even across deployments**. This is what lets a route serve data without touching the origin for minutes, hours, or until you explicitly invalidate it.

```tsx
// Persisted in the Data Cache (Next 13/14 default, or explicit in 15).
const res = await fetch('https://api.example.com/posts', {
  cache: 'force-cache',          // store and reuse
  next: { revalidate: 3600 },    // refresh at most once per hour
});
```

Key properties:

- **Scope:** global to the deployment (server-side), shared across all users and requests.
- **Lifetime:** persistent. Survives requests and redeploys until revalidated.
- **Invalidation:** time-based (`revalidate`) or on-demand (`revalidatePath`, `revalidateTag`). Covered in the next lesson.
- **Relationship to Request Memoization:** Memoization sits *in front* of the Data Cache. A repeated fetch in one render hits memoization first; a fresh request goes to the Data Cache; a miss there goes to the origin.

## 3. The Full Route Cache

After a route renders, Next.js can cache the **rendered output** — both the HTML and the React Server Component (RSC) payload. This is the Full Route Cache, and it is why a statically rendered route costs essentially nothing to serve.

A route lands in the Full Route Cache only if it is **statically rendered**. The moment a route opts into dynamic rendering — by reading `cookies()`, `headers()`, `searchParams`, or using uncached data — it is rendered per-request and skips the Full Route Cache.

```tsx
// app/blog/[slug]/page.tsx
// Statically rendered: cached in the Full Route Cache at build time.
export default async function Page({ params }: { params: { slug: string } }) {
  const post = await getPost(params.slug); // cached fetch -> static
  return <Article post={post} />;
}

// Force this segment dynamic — it will NOT be in the Full Route Cache.
export const dynamic = 'force-dynamic';
```

Key properties:

- **Scope:** server-side, per route. Built from build-time and runtime rendering.
- **Lifetime:** persistent until the route's data is revalidated or you redeploy.
- **Depends on the Data Cache:** revalidating the Data Cache for data a route uses also invalidates that route's entry in the Full Route Cache.

## 4. The Client-side Router Cache

The Router Cache lives **in the browser**. As the user navigates, Next.js stores the RSC payload for visited and prefetched routes in memory, so going "back" or revisiting a route is instant — no network round trip.

Key properties:

- **Scope:** a single user's browser session. Cleared on full page reload.
- **Lifetime:** in-memory for the session; entries have a staleness window (and prefetched entries a shorter one). You can opt segments out with `dynamic = 'force-dynamic'` plus appropriate config, and you invalidate it imperatively.
- **Invalidation:** `router.refresh()`, calling `revalidatePath`/`revalidateTag` from a Server Action (which also refreshes the current route's client cache), or a hard reload.

This is the cache that surprises people most: you mutate data on the server, the Data Cache and Full Route Cache are correct, but the user still sees stale UI because the Router Cache held an old RSC payload. That is why Server Actions that mutate should call `revalidatePath`/`revalidateTag` — it busts both server caches *and* refreshes the client's Router Cache for the current route.

## How a Request Flows Through All Four

```text
User clicks a link
   │
   ▼
Router Cache hit?  ── yes ──▶ render instantly from in-memory RSC payload
   │ no
   ▼
Request hits server
   │
   ▼
Full Route Cache hit (static route)? ── yes ──▶ return cached HTML/RSC
   │ no (dynamic, or revalidated)
   ▼
Render route → components call fetch()/queries
   │
   ▼
Request Memoization (dedupe within this render)
   │ miss
   ▼
Data Cache hit? ── yes ──▶ return stored data
   │ miss
   ▼
ORIGIN (DB / API / CMS) → store in Data Cache → maybe Full Route Cache
```

## The Next 15 Default Change (Read This Carefully)

The single biggest source of confusion is that **caching defaults flipped between Next 14 and Next 15**. You must know which version you are on.

### Next 13/14 mental model: opt-out (cached by default)

- `fetch()` defaulted to `cache: 'force-cache'` — cached in the Data Cache automatically.
- `GET` Route Handlers were cached by default.
- You *opted out* of caching: `cache: 'no-store'`, `export const dynamic = 'force-dynamic'`, or reading dynamic functions.

```tsx
// Next 13/14: this was CACHED automatically.
const res = await fetch('https://api.example.com/posts');

// Opt OUT to always hit origin:
const fresh = await fetch('https://api.example.com/posts', {
  cache: 'no-store',
});
```

### Next 15 mental model: opt-in (uncached by default)

- `fetch()` is **no longer cached by default** — it behaves like `cache: 'no-store'`.
- `GET` Route Handlers are **no longer cached by default**.
- You now *opt in*: `cache: 'force-cache'` or `next: { revalidate }`.

```tsx
// Next 15: this is UNCACHED by default (hits origin each request).
const res = await fetch('https://api.example.com/posts');

// Opt IN to the Data Cache:
const cached = await fetch('https://api.example.com/posts', {
  cache: 'force-cache',
  next: { revalidate: 3600 },
});

// For a Route Handler in Next 15, opt in explicitly:
// app/api/posts/route.ts
export const dynamic = 'force-static';
```

The Request Memoization, Full Route Cache, and Router Cache all still exist in Next 15 — only the *default* Data Cache behavior for `fetch` and GET handlers changed. The practical effect: in Next 15 you must be deliberate about what you cache, which is safer (no accidental stale data) but means apps that "just worked" fast in 14 need explicit `force-cache`/`revalidate` to regain that.

### Trade-offs: opt-in vs opt-out

| | Opt-out (13/14) | Opt-in (15) |
|---|---|---|
| Default speed | Fast by default | Correct by default |
| Risk | Accidental staleness | Accidental origin load / cost |
| Mental overhead | Remember to disable | Remember to enable |
| Best for | Read-heavy, tolerant data | Apps with sensitive/fresh data |

The Next 15 default reflects hard-won experience: silent over-caching caused more production incidents (stale prices, stale auth state) than the extra origin calls cost. When you want speed in 15, you ask for it — explicitly, per fetch — which is exactly where caching decisions should live.

## Pitfalls

- **Confusing Memoization with the Data Cache.** Memoization is per-request and free; the Data Cache is persistent and is what causes cross-request staleness. "It's deduping" and "it's cached for an hour" are different layers.
- **Forgetting the Router Cache after a mutation.** Server caches can be perfectly fresh while the browser shows stale UI. Always revalidate from Server Actions so the client Router Cache refreshes too.
- **Assuming Next 14 caching behavior on Next 15** (or vice versa). Check your version before debugging "why isn't this cached." The defaults are opposite.
- **`cache()` vs `unstable_cache()` mix-up.** `cache()` (React) is per-request memoization for non-fetch data. `unstable_cache()` (Next) is the persistent Data Cache for non-fetch data. Don't reach for the wrong one.
- **Expecting `force-dynamic` to make data fresh per call.** It makes the *route* dynamic, but a `force-cache` fetch inside it still reads the Data Cache. Caches compose; one setting doesn't override another layer.

## Exercises

1. **Trace a request.** Draw, for one of your own routes, exactly which of the four caches each piece of data passes through on a cold request, a warm request, and a navigation back to it. Identify which layer would serve stale data.
2. **Version audit.** Take a `fetch`-heavy page and determine its caching behavior under both Next 14 and Next 15 defaults. List every change you'd need to keep behavior identical when upgrading to 15.
3. **Static vs dynamic boundary.** Add `cookies()` to a previously static route and predict (then verify) what happens to its Full Route Cache entry and to its build output.
4. **Router Cache demo.** Build a page that mutates data via a Server Action *without* calling `revalidatePath`, observe the stale client UI, then fix it. Explain which cache was responsible before and after.

## Key Takeaways

- There are four caches: Request Memoization (per-request fetch dedupe), the Data Cache (persistent data), the Full Route Cache (rendered routes), and the client Router Cache (in-browser navigation).
- Split them mentally into **data caches** (Memoization, Data Cache) and **route caches** (Full Route, Router).
- A request flows top-down: Router Cache → Full Route Cache → render → Request Memoization → Data Cache → origin.
- Caches compose; one setting rarely controls another layer. Debug by asking *which cache* holds the value.
- Next 15 flipped the default: `fetch` and GET Route Handlers are now uncached by default (opt-in), where 13/14 cached by default (opt-out). Know your version.

---

[← Previous: Server Actions in Depth](../01-deep-rendering/04-server-actions-in-depth.md) | [Back to Course Index](../README.md) | [Next: Revalidation and Invalidation Strategies →](02-revalidation-strategies.md)
