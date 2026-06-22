# Streaming and Suspense Orchestration

Streaming is what makes a data-heavy page feel fast: instead of holding the whole response until the slowest query finishes, you send the page in pieces as they become ready. But streaming is not free magic — done carelessly it produces layout shift, spinner soup, and a worse experience than a single clean paint. This lesson is about orchestrating Suspense boundaries deliberately, as a UX and performance design decision, not a reflex.

## What You'll Learn

- How streaming works in the App Router and what a Suspense boundary actually does.
- Designing boundary granularity: what to stream versus what to block on.
- Avoiding layout shift while content streams in.
- `loading.tsx` versus explicit `<Suspense>` — when each fits.
- Streaming with data dependencies and how to parallelize fetches.
- Preloading data to start fetches early and cut waterfalls.
- The UX trade-off between progressive streaming and a single coherent paint.

## How Streaming Works

When a Server Component tree contains a `<Suspense>` boundary, Next.js can send the response in stages over a single HTTP connection:

```text
1. Server renders everything it can immediately (outside Suspense).
2. It flushes that HTML to the browser right away — first paint happens.
   In place of each suspended boundary, it sends the fallback.
3. As each suspended boundary's data resolves, the server renders it and
   flushes the real content, plus a tiny script that swaps it in.
4. The browser replaces fallbacks with content as chunks arrive.
```

The win is **time-to-first-byte and first paint decouple from your slowest data source.** A page with a fast header and a slow report no longer makes the user stare at a blank screen while the report loads; they see the header instantly and the report streams in.

A Suspense boundary is the unit of streaming. Anything that suspends (an async Server Component awaiting data, a `use()` of a pending promise) inside a boundary shows the fallback until ready, then streams in independently of its siblings.

## Boundary Granularity: What to Stream vs Block

This is the central design skill. Too few boundaries and you're back to all-or-nothing blocking. Too many and the page becomes a flickering mess of independently-popping skeletons.

### Block on the essential, stream the rest

Block (await *before* the first paint, outside any Suspense) on whatever defines the page's identity and what the user came for:

- The page title and primary content shape.
- Anything that, if it popped in late, would feel broken.

Stream the secondary, slower, or below-the-fold material:

- Recommendations, "people also bought," activity feeds.
- Anything backed by a slow third-party API.
- Content the user will scroll to, not see immediately.

```tsx
// app/dashboard/page.tsx
import { Suspense } from "react";

export default async function Dashboard() {
  // Block on the fast, essential data — it's part of the first paint.
  const user = await getUser();

  return (
    <main>
      <h1>Welcome back, {user.name}</h1>

      {/* Stream the slow widgets independently. Each resolves on its own. */}
      <div className="grid">
        <Suspense fallback={<CardSkeleton />}>
          <RevenueChart orgId={user.orgId} /> {/* slow analytics query */}
        </Suspense>
        <Suspense fallback={<CardSkeleton />}>
          <RecentActivity orgId={user.orgId} /> {/* slow third-party API */}
        </Suspense>
      </div>
    </main>
  );
}
```

### Group related slow things under one boundary

If three widgets are all part of one logical unit and pop in at roughly the same time, wrapping them in *one* Suspense boundary (rather than three) avoids three separate layout jumps and reads as a single coherent reveal. Granularity is a judgment call about coherence, not a rule to maximize.

## Avoiding Layout Shift

The cardinal sin of streaming is **content jumping around** as chunks arrive. The fix is that your fallback must reserve the same space and shape as the real content.

```tsx
// app/dashboard/_components/card-skeleton.tsx
export function CardSkeleton() {
  // Same dimensions, padding, and grid footprint as the real card.
  return <div className="card h-64 w-full animate-pulse rounded-lg bg-muted" />;
}
```

Principles:

- **Match dimensions.** A skeleton that's 64px tall feeding into 400px of content causes a violent jump. Size skeletons to the expected content.
- **Reserve grid/flex slots.** Keep the layout container stable so streamed children fill predetermined cells rather than reflowing the page.
- **Don't stream things above the fold that you can cheaply block on.** A shifting header is far more jarring than a shifting footer.

## `loading.tsx` vs Explicit `<Suspense>`

Next.js gives you two ways to declare a streaming boundary; they're for different scopes.

- **`loading.tsx`** is a route-segment convention. It automatically wraps the *entire page* of that segment in a Suspense boundary, showing your loading UI while the whole page renders. It's coarse-grained — one fallback for the whole route — and ideal for the instant feedback you want on navigation.
- **Explicit `<Suspense>`** is surgical. You place it around specific subtrees to stream them independently while the rest of the page is already interactive.

```text
loading.tsx          → "the whole page is loading"  (navigation-level UX)
<Suspense>           → "this part is loading, the rest is ready" (component-level UX)
```

Use them together: `loading.tsx` gives instant feedback when the user navigates in, and explicit `<Suspense>` boundaries within the page stream individual slow sections once the shell is up. A common, strong pattern is a lightweight `loading.tsx` shell plus a few targeted boundaries for the genuinely slow widgets.

## Streaming With Data Dependencies (Avoid Waterfalls)

Streaming exposes a trap: **sequential awaits create waterfalls** even inside a streamed tree. If component B's fetch depends on A's result, they must run in sequence. But independent fetches should run in parallel — and naive `await` code accidentally serializes them.

```tsx
// BAD: sequential — total time = a + b + c
const a = await getA();
const b = await getB(); // doesn't need `a`, but waits for it anyway
const c = await getC();
```

```tsx
// GOOD: parallel — total time = max(a, b, c)
const [a, b, c] = await Promise.all([getA(), getB(), getC()]);
```

When the components are separate, give each its own Suspense boundary and let it fetch independently — the framework runs them concurrently and streams each as it resolves. The only time you *should* serialize is a genuine dependency (you need the user's `orgId` before you can fetch the org's invoices).

### Preloading to start fetches early

A subtler optimization: kick off a fetch *before* you actually await it, so the network request is already in flight by the time the component needs the data. The pattern relies on request memoization (covered in the caching module) — calling the same cached fetch twice triggers one request.

```ts
// features/catalog/server/product.ts
import { cache } from "react";

export const getProduct = cache(async (id: string) => {
  // ... fetch; memoized per request so calling twice = one network call
});

export function preloadProduct(id: string) {
  void getProduct(id); // fire-and-forget: warms the cache, returns nothing
}
```

```tsx
// app/products/[id]/page.tsx
export default async function Page({ params }: { params: Promise<{ id: string }> }) {
  const { id } = await params;
  preloadProduct(id); // start the fetch NOW, before rendering children

  return (
    <Suspense fallback={<Skeleton />}>
      {/* By the time this awaits getProduct(id), it's likely already resolved. */}
      <ProductView id={id} />
    </Suspense>
  );
}
```

Preloading turns a render-then-fetch waterfall into fetch-while-rendering. It's most valuable when a slow fetch lives deep in the tree but you know the input early.

## Trade-offs: Streaming vs a Single Coherent Paint

Streaming is not universally better. Weigh it honestly.

**Stream when:**

- Parts of the page are meaningfully slower than others.
- The fast parts are useful on their own (the user can start reading/acting).
- You can design stable, non-shifting skeletons.

**Prefer blocking (single paint) when:**

- The page is small and fast enough that streaming overhead and skeleton flashes cost more than they save.
- Progressive reveal would feel *less* polished than one clean appearance — e.g., a tightly designed marketing hero where staggered loading looks broken.
- Layout shift is unavoidable and would be distracting.

The honest framing: streaming optimizes *perceived* performance and time-to-first-content, sometimes at the expense of visual cohesion. For an app dashboard, that trade is almost always worth it. For a pixel-perfect landing page, a single coherent paint may win.

## Pitfalls

- **Skeleton-doesn't-match-content layout shift.** Fallbacks that don't reserve the right space cause jarring jumps. Size skeletons to the real content.
- **Spinner soup.** Over-granular boundaries make the page a flickering mess. Group coherent units under shared boundaries.
- **Hidden waterfalls inside streamed trees.** Sequential `await`s serialize independent fetches. Use `Promise.all` for independent data and preload deep fetches.
- **Streaming above-the-fold essentials.** Don't stream the hero or primary content if you can cheaply block on it — late-arriving primary content feels broken.
- **Forgetting `loading.tsx` wraps the whole segment.** If you also want partial streaming, add explicit `<Suspense>` boundaries inside; `loading.tsx` alone is all-or-nothing for the segment.

## Exercises

1. **Boundary design.** For a dashboard with a fast user header, a slow analytics chart, a slow activity feed, and a fast settings link, decide what to block on and what to stream, and where to place Suspense boundaries. Justify the granularity.
2. **Kill a waterfall.** Given a page that sequentially awaits three independent fetches, refactor to parallelize them and estimate the latency improvement.
3. **Skeleton fidelity.** Build a skeleton for a card component that produces zero cumulative layout shift, and explain how you verified it matches the real content's footprint.
4. **Preload deep.** Implement React `cache()`-based preloading for a fetch used by a component three levels deep, starting the request at the route level. Explain why this requires request memoization to work.
5. **Stream-or-not judgment.** Pick two real pages (one app dashboard, one marketing landing) and argue for streaming on one and a single coherent paint on the other, citing the UX trade-offs.

## Key Takeaways

- A Suspense boundary is the unit of streaming: the server flushes the shell immediately and streams each boundary's content as its data resolves.
- Block on essential, fast, above-the-fold content; stream the slow and secondary — and group coherent units under shared boundaries to avoid flicker.
- Match skeleton dimensions to real content and reserve layout slots to prevent layout shift.
- `loading.tsx` is segment-level (whole page); explicit `<Suspense>` is component-level — use them together.
- Parallelize independent fetches with `Promise.all` and preload deep fetches with React `cache()` to eliminate waterfalls; streaming trades visual cohesion for perceived speed, so choose per page.

---

[← Previous: Rendering Strategies and Partial Prerendering](02-rendering-strategies.md) | [Back to Course Index](../README.md) | [Next: Server Actions in Depth →](04-server-actions-in-depth.md)
