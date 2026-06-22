# Rendering Strategies and Partial Prerendering

Every route in your app sits somewhere on a spectrum from "rendered once at build, served to everyone" to "rendered fresh for every request." Where it sits decides your latency, your server cost, and your data freshness. Many performance problems are really just a route that landed on the wrong point of that spectrum — usually dynamic when it should have been static. This lesson gives you the controls and the judgment to place each route deliberately.

## What You'll Learn

- Static vs dynamic rendering and exactly what triggers the switch.
- The segment config knobs: `dynamic`, `revalidate`, `fetchCache`, `runtime`.
- Incremental Static Regeneration (ISR) and how it gets you "static, but fresh."
- Partial Prerendering (PPR): the static shell + dynamic holes model.
- How to choose a strategy per route from real signals.
- The cost and latency trade-offs behind each choice.

## Static vs Dynamic: The Core Distinction

- **Static rendering** happens at build time (or on revalidation). The HTML and RSC payload are generated once and served from the cache/CDN to every user. Near-zero per-request cost, lowest latency, but the same content for everyone.
- **Dynamic rendering** happens at request time. The route renders fresh for each request, so it can use per-request data (the logged-in user, cookies, the current time). Higher latency and real server cost per visit.

By default, Next.js renders **statically** unless your code does something that *requires* request-time information. You usually don't choose dynamic — you trigger it, sometimes by accident.

### What triggers dynamic rendering

A route (and the segments above it) becomes dynamic when it uses any **Dynamic API** or opts out of caching:

- Reading **`cookies()`** or **`headers()`** (from `next/headers`).
- Reading **`searchParams`** in a page (the query string is per-request).
- The **`connection()`** API or otherwise reading the incoming request.
- A `fetch` with **`{ cache: "no-store" }`** or an uncached data read.
- Setting **`export const dynamic = "force-dynamic"`** explicitly.

The gotcha worth burning into memory: **these propagate upward.** If a layout reads `cookies()`, every route under that layout renders dynamically. A single misplaced `headers()` call in a shared layout can quietly turn your entire app dynamic and blow up your bill.

```tsx
// app/(app)/layout.tsx — reading cookies here makes ALL child routes dynamic
import { cookies } from "next/headers";

export default async function AppLayout({ children }: { children: React.ReactNode }) {
  const session = (await cookies()).get("session"); // ← opts the whole subtree dynamic
  // If only ONE route needs the session, read it there, not in the shared layout.
  return <Shell>{children}</Shell>;
}
```

## Segment Config: The Knobs

Each route segment (a `page.tsx`, `layout.tsx`, or `route.ts`) can export config constants that override defaults. Know these four.

```ts
// app/products/[id]/page.tsx

// "auto" (default) | "force-static" | "force-dynamic" | "error"
export const dynamic = "force-static";

// false (cache forever) | number of seconds | (with ISR) revalidate window
export const revalidate = 3600;

// "auto" | "default-cache" | "force-cache" | "force-no-store" | "only-cache"
export const fetchCache = "default-cache";

// "nodejs" (default) | "edge"
export const runtime = "nodejs";
```

- **`dynamic`** forces the rendering mode. `force-static` makes Dynamic APIs return empty/default values instead of opting into dynamic (use carefully); `force-dynamic` guarantees per-request rendering; `"error"` makes any dynamic usage a build error — a great guard rail for routes you *intend* to keep static.
- **`revalidate`** sets the ISR window in seconds: the cached output is served until it's this old, then regenerated. `false` (or absent) means cache indefinitely.
- **`fetchCache`** overrides the default caching of `fetch` calls within the segment. Rarely needed; reach for it only when you must override per-`fetch` settings wholesale.
- **`runtime`** picks Node vs Edge for the segment. Edge for ultra-low-latency, globally distributed, lightweight routes; Node when you need full APIs or your ORM.

## ISR: Static, But Fresh

Incremental Static Regeneration is the pragmatic middle ground and the workhorse for content that changes occasionally. You get static performance with a freshness guarantee.

```tsx
// app/blog/[slug]/page.tsx
export const revalidate = 60; // serve cached HTML, regenerate at most once per 60s

export default async function Post({ params }: { params: Promise<{ slug: string }> }) {
  const { slug } = await params;
  const post = await getPost(slug); // cached read, refreshed on revalidation
  return <Article post={post} />;
}
```

How it behaves: the first request after the window expires gets the *stale* page instantly, while Next.js regenerates in the background; subsequent requests get the fresh version. This is **stale-while-revalidate** — readers never wait for a render.

You can also revalidate **on demand** (after a content edit, via `revalidatePath`/`revalidateTag`) instead of, or in addition to, a timer — covered in the caching module. On-demand is strictly better when you control the write path: pages stay static and only regenerate when the data actually changes.

## Partial Prerendering: Static Shell, Dynamic Holes

The old model forced an all-or-nothing choice per route: one dynamic piece (say, a personalized greeting) made the *whole page* dynamic. **Partial Prerendering (PPR)** breaks that false choice. Within a single route, you get:

- A **static shell** — layout, nav, product description, anything that's the same for everyone — prerendered at build time and served instantly from the edge.
- **Dynamic holes** — the personalized or request-specific parts — wrapped in `<Suspense>`, streamed in at request time.

```text
┌─────────────────────────────────────┐
│  Static shell (prerendered, instant) │
│  ┌───────────────────────────────┐   │
│  │  <Suspense> dynamic hole       │   │ ← streamed per request
│  │  e.g. user cart, recommendations│  │
│  └───────────────────────────────┘   │
│  Static footer                        │
└─────────────────────────────────────┘
```

```tsx
// app/products/[id]/page.tsx
import { Suspense } from "react";

export const experimental_ppr = true; // opt this route into PPR

export default async function ProductPage({ params }: { params: Promise<{ id: string }> }) {
  const { id } = await params;
  const product = await getProduct(id); // static: same for everyone, prerendered

  return (
    <main>
      <ProductDetails product={product} /> {/* part of the static shell */}

      {/* Dynamic hole: reads cookies/user, streams in at request time */}
      <Suspense fallback={<CartSkeleton />}>
        <PersonalizedCart productId={id} />
      </Suspense>
    </main>
  );
}
```

The mechanism: anything that uses a Dynamic API must be inside a `<Suspense>` boundary. Next.js prerenders everything *outside* those boundaries into the static shell and defers the inside to request time. The user gets an instant first paint (the shell) with the dynamic bits filling in — the best of both worlds.

PPR is enabled per-route via `experimental_ppr` once `experimental.ppr: "incremental"` is set in `next.config`. It's the direction the framework is heading; learn to think in shell-plus-holes terms even before it's stable in your version.

### Trade-offs: choosing a strategy per route

Decide from two questions: *how often does the content change* and *who needs it personalized*.

| Strategy | Latency | Server cost | Freshness | Use when |
| --- | --- | --- | --- | --- |
| **Static** | Lowest | ~Zero | At build only | Marketing pages, docs, anything identical for all users |
| **ISR** | Lowest (cached) | Low (periodic) | Within window / on demand | Blogs, product catalogs, content that changes occasionally |
| **Dynamic** | Higher | High (per request) | Always fresh | Dashboards, per-user feeds, anything truly request-specific |
| **PPR** | Lowest shell | Low + per-hole | Shell at build, holes fresh | Pages mixing shared shell with personalized parts |

The decision tree in practice:

1. Is it the same for every user and rarely changes? → **Static** (or ISR if "rarely" means hours/days).
2. Mostly shared but with a few personalized pieces? → **PPR** (or static shell + client-fetched islands if PPR isn't available).
3. Fundamentally per-request / per-user end to end? → **Dynamic**, and consider edge runtime to cut latency.

## Pitfalls

- **Accidental dynamic via a shared layout.** A `cookies()`/`headers()` call in a root or section layout makes everything beneath it dynamic. Push per-request reads down to the routes that need them.
- **Reading `searchParams` and not realizing it's dynamic.** Any page that reads the query string opts out of full static rendering. With PPR, isolate that read inside a Suspense boundary.
- **`force-static` masking bugs.** It makes Dynamic APIs return defaults silently — handy, but you can ship a "personalized" feature that's actually showing everyone the build-time default.
- **Setting `revalidate` too low.** `revalidate = 1` is nearly dynamic in cost while still risking staleness; pick a window that matches real change frequency, and prefer on-demand revalidation when you own the writes.
- **Choosing edge runtime then importing Node-only code.** It builds fine until a Node API or unsupported library blows up at runtime. Verify library compatibility before opting into edge.

## Exercises

1. **Diagnose accidental dynamic.** Given an app where every page is unexpectedly dynamic, find the single cause (hint: a shared layout) and propose the minimal fix that restores static rendering for the routes that don't need request data.
2. **Strategy matrix.** For a SaaS app's routes — marketing home, pricing, docs, dashboard, per-user settings, a public blog — assign each a strategy and justify it from change-frequency and personalization.
3. **Build a PPR page.** Implement a product page with a static shell (details, reviews) and a dynamic hole (the current user's recently-viewed list) using `<Suspense>` and `experimental_ppr`.
4. **ISR vs on-demand.** For a CMS-backed marketing site, design a revalidation approach. When would you use a time window, when on-demand, and when both?
5. **Cost comparison.** For a route at 500k requests/day, estimate renders/day under dynamic, under ISR with a 5-minute window, and under static. Discuss the freshness you trade away in each.

## Key Takeaways

- Static is the default; you *trigger* dynamic by using cookies/headers/searchParams or uncached fetches — and those triggers propagate up through layouts.
- Segment configs (`dynamic`, `revalidate`, `fetchCache`, `runtime`) let you override rendering and caching behavior per segment; use `dynamic = "error"` as a guard rail.
- ISR gives static performance with stale-while-revalidate freshness; prefer on-demand revalidation when you control the writes.
- PPR ends the all-or-nothing choice: prerender a static shell and stream dynamic holes wrapped in `<Suspense>`.
- Choose per route from two questions — how often it changes and who needs it personalized — then pick static / ISR / PPR / dynamic accordingly.

---

[← Previous: The RSC Architecture in Depth](01-rsc-architecture.md) | [Back to Course Index](../README.md) | [Next: Streaming and Suspense Orchestration →](03-streaming-orchestration.md)
