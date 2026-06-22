# Advanced Data Fetching Patterns

The App Router lets you fetch data anywhere in the component tree, which is liberating until you realize you've accidentally built a request waterfall — a chain of fetches that each wait for the previous one, turning a 100ms page into an 800ms page for no good reason. Latency is additive when requests are sequential and bounded by the slowest when they're parallel. At scale, the difference between those two shapes is the difference between a snappy app and one users describe as "laggy."

This lesson is about controlling the *shape* of your data fetching: spotting waterfalls, parallelizing what's independent, sequencing only what's genuinely dependent, and using Suspense to hide the latency you can't remove.

## What You'll Learn

- How to recognize a request waterfall and read its timing
- Parallel fetching with `Promise.all` to collapse independent requests
- The preload pattern: starting fetches before you render the component that needs them
- Colocating fetches in components without reintroducing waterfalls
- Deduping repeated fetches via Request Memoization
- When sequential fetching is correct (genuine data dependencies)
- Using streaming + Suspense to overlap latency with rendering

## The Waterfall Problem

A waterfall happens when fetch B can't start until fetch A finishes — even when B doesn't actually depend on A. Each `await` in sequence is a new rung.

```tsx
// app/dashboard/page.tsx — WATERFALL (anti-pattern)
export default async function Dashboard() {
  const user = await getUser();          // 200ms
  const posts = await getPosts();        // 200ms — waits for user, but doesn't need it
  const notifications = await getNotifications(); // 200ms — waits for both
  return <Layout user={user} posts={posts} notifications={notifications} />;
}
```

```text
WATERFALL (sequential awaits)
getUser           [====]
getPosts               [====]
getNotifications            [====]
                  0   200  400  600   ← total ≈ 600ms
```

Three independent fetches cost the *sum* of their latencies. None of them needed the others' results.

## Parallel Fetching

If fetches are independent, fire them together and await as a group:

```tsx
// app/dashboard/page.tsx — PARALLEL (fixed)
export default async function Dashboard() {
  // Start all three immediately; await once.
  const [user, posts, notifications] = await Promise.all([
    getUser(),
    getPosts(),
    getNotifications(),
  ]);
  return <Layout user={user} posts={posts} notifications={notifications} />;
}
```

```text
PARALLEL (Promise.all)
getUser           [====]
getPosts          [====]
getNotifications  [====]
                  0   200          ← total ≈ 200ms (bounded by slowest)
```

The page now costs the *max* of the three, not the sum. This is the single highest-leverage fix for most slow App Router pages.

### A subtle gotcha: `Promise.all` is all-or-nothing

If one fetch rejects, `Promise.all` rejects. When partial data is acceptable, use `Promise.allSettled` and handle each result:

```tsx
const results = await Promise.allSettled([getUser(), getPosts()]);
const user = results[0].status === 'fulfilled' ? results[0].value : null;
```

## The Preload Pattern

`Promise.all` works when one component owns all the fetches. But fetches often belong to *different* components, and a parent may render a slow child before a fast one that also needs data. The preload pattern starts a fetch early — without awaiting it — so it's in flight by the time the component renders.

```ts
// app/lib/posts.ts
import { cache } from 'react';

export const getPost = cache(async (slug: string) => {
  const res = await fetch(`https://api.example.com/posts/${slug}`);
  return res.json();
});

// Kick off the request without awaiting. Thanks to cache()/memoization,
// the later real await returns the in-flight (or completed) promise.
export const preloadPost = (slug: string): void => {
  void getPost(slug);
};
```

```tsx
// app/blog/[slug]/page.tsx
import { getPost, preloadPost } from '@/app/lib/posts';
import { Comments } from './comments';

export default async function Page({ params }: { params: { slug: string } }) {
  // Start the post fetch AND warm anything a child needs, before awaiting.
  preloadPost(params.slug);
  preloadComments(params.slug); // child's data starts now, not when it renders

  const post = await getPost(params.slug);
  return (
    <article>
      <h1>{post.title}</h1>
      <Comments slug={params.slug} /> {/* its fetch is already in flight */}
    </article>
  );
}
```

The preload + `cache()` combination is what makes colocation safe: a child can own its own fetch, yet the parent can ensure it started early. Without preload, the child's fetch wouldn't begin until React reached it during render — a waterfall hidden inside the component tree.

## Colocating Fetches Without Waterfalls

The App Router's promise is that each component fetches what it needs. The risk is that nested `await`s become a waterfall down the tree. Two tools keep colocation fast:

1. **Request Memoization** dedupes identical fetches, so colocating the same query in three components is one request.
2. **Preload** ensures a deeply nested component's fetch starts at the top of the tree, not when render reaches it.

```tsx
// app/profile/[id]/page.tsx
export default async function ProfilePage({ params }: { params: { id: string } }) {
  // Warm every dependency up front; each component still owns its own await.
  preloadUser(params.id);
  preloadUserPosts(params.id);
  preloadUserStats(params.id);

  return (
    <>
      <ProfileHeader id={params.id} /> {/* awaits getUser */}
      <ProfileStats id={params.id} />  {/* awaits getUserStats */}
      <ProfileFeed id={params.id} />   {/* awaits getUserPosts */}
    </>
  );
}
```

All three fetches are in flight before any child renders; each child awaits its own, and memoization guarantees no duplicates.

## Deduping via Request Memoization

You don't always need preload. If multiple components in one render call the identical fetch, memoization already collapses them:

```tsx
// Both the layout and the page call getUser() — one network request.
// app/(app)/layout.tsx
const user = await getUser();   // network call
// app/(app)/page.tsx
const user = await getUser();   // memoized, no second call
```

For non-`fetch` data, wrap the query in React's `cache()` to get the same dedupe (covered in the caching layers lesson). This is why you should write fetch functions as small reusable units rather than inlining queries — reuse buys you free deduping.

## Sequential Fetching (When It's Actually Correct)

Not every waterfall is a bug. Sometimes B *genuinely* needs A's result:

```tsx
// app/orders/[id]/page.tsx — LEGITIMATELY sequential
export default async function OrderPage({ params }: { params: { id: string } }) {
  const order = await getOrder(params.id);          // need the order...
  const customer = await getCustomer(order.userId); // ...to know whose it is
  return <OrderDetail order={order} customer={customer} />;
}
```

Here you cannot parallelize: `customer` depends on `order.userId`. Two things to do anyway:

- **Parallelize the independent parts.** If you also need `getOrderItems(order.id)` and `getCustomer(order.userId)`, those two *can* run together once `order` resolves: `Promise.all([getCustomer(order.userId), getOrderItems(order.id)])`.
- **Stream the dependent part** so the rest of the page renders while the chain resolves (next section).

### Trade-offs: which shape when

| Shape | Use when | Cost |
|---|---|---|
| Parallel (`Promise.all`) | Fetches are independent | All-or-nothing failure |
| Sequential `await`s | B needs A's result | Latency sums — minimize the chain |
| Preload + colocated awaits | Different components own different data | Slightly more wiring |
| Streaming + Suspense | Some data is slow and non-critical | More boundaries to manage |

Default to **parallel**. Drop to sequential only at a real data dependency, and shorten the chain by parallelizing siblings. Reach for **streaming** when latency is irreducible and you'd rather show *something* now.

## Streaming + Suspense to Hide Latency

When a fetch is slow and you can't make it faster, don't make the whole page wait. Wrap the slow part in `Suspense`; Next.js streams the fast shell immediately and fills in the slow region when it's ready.

```tsx
// app/dashboard/page.tsx
import { Suspense } from 'react';

export default function Dashboard() {
  return (
    <>
      <Header />                              {/* renders instantly */}
      <Suspense fallback={<StatsSkeleton />}>
        <SlowStats />                         {/* streams in when ready */}
      </Suspense>
      <Suspense fallback={<FeedSkeleton />}>
        <SlowFeed />                          {/* independent boundary */}
      </Suspense>
    </>
  );
}

// Each component awaits its own data; Suspense unblocks the shell.
async function SlowStats() {
  const stats = await getStats(); // 600ms
  return <StatsView stats={stats} />;
}
```

```text
WITHOUT streaming                WITH streaming + Suspense
[ wait 600ms ][ paint all ]      [ paint shell at ~50ms ]
                                 SlowStats  ......[fill at 600ms]
                                 SlowFeed   ...[fill at 400ms]
TTFB/paint ≈ 600ms               First paint ≈ 50ms; regions fill independently
```

Streaming doesn't reduce total latency — `SlowStats` still takes 600ms — but it moves first paint forward dramatically and lets each region appear as soon as it's ready. Combine it with parallel fetching inside each boundary so the boundaries themselves don't waterfall.

## Pitfalls

- **Accidental tree waterfalls.** Colocated fetches feel clean but each nested `await` is a rung unless you preload. Profile the actual timing, don't assume.
- **`Promise.all` masking failures.** One rejection fails the whole group. Use `allSettled` when partial data is acceptable, and decide deliberately.
- **Awaiting in a loop.** `for (const id of ids) await getThing(id)` is a waterfall of length N. Use `Promise.all(ids.map(getThing))`.
- **Suspense without parallel fetches inside.** A boundary that runs three sequential awaits still streams slowly. Parallelize within the boundary.
- **Over-streaming.** Too many tiny Suspense boundaries cause layout shift and a flickery UI. Stream meaningful regions, not every element.
- **Forgetting memoization is per-request.** Dedupe only happens within one render pass; it does not persist across requests (that's the Data Cache).

## Exercises

1. **Profile a real page.** Add timing logs around each fetch on one of your routes and draw its timing diagram. Classify each fetch as independent or dependent.
2. **Kill the waterfall.** Refactor that page to `Promise.all` for the independent fetches and measure the before/after total time.
3. **Implement the preload pattern** for a page where a nested child owns a slow fetch. Verify (via timing) that the child's request starts at the top of the tree.
4. **Design Suspense boundaries** for a dashboard with one fast and two slow regions. Decide boundary granularity and justify it against layout-shift risk.
5. **Untangle a dependent chain.** Given `order → customer` and `order → items`, restructure so the two second-level fetches run in parallel, and stream the customer panel.

## Key Takeaways

- Sequential `await`s sum latency; parallel fetches are bounded by the slowest. Default to parallel.
- Use `Promise.all` for independent fetches owned by one component; `allSettled` when partial data is fine.
- The preload pattern (start a fetch without awaiting, backed by `cache()`/memoization) makes colocated fetching fast and waterfall-free.
- Sequential fetching is correct only at a genuine data dependency — minimize the chain and parallelize siblings.
- Streaming + Suspense hides irreducible latency by painting the shell first and filling regions as they resolve; parallelize fetches inside each boundary.

---

[← Previous: Revalidation and Invalidation Strategies](02-revalidation-strategies.md) | [Back to Course Index](../README.md) | [Next: Designing a Data Access Layer →](04-data-access-layer.md)
