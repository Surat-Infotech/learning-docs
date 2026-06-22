# Streaming and Suspense

Sometimes one slow piece of data holds up your whole page. The user stares at a
blank screen waiting for everything, even though most of the page was ready
ages ago. **Streaming** fixes this: it lets Next.js send the page in pieces, so
people see the fast parts immediately and the slow parts fill in as they arrive.

Think of a restaurant bringing out your bread and water right away while the main
course is still cooking. You are not stuck at an empty table; you have something
to enjoy while the slow dish finishes. Streaming does the same for a web page.

## What You'll Learn

- What streaming is and why it makes pages feel faster
- The difference between waiting for **everything** vs sending the page in pieces
- How React's `<Suspense fallback={...}>` streams in one slow part
- How to split a slow data fetch into its own component
- How `loading.tsx` uses streaming automatically for a whole route
- Why this improves **perceived performance** even when total time is the same
- A complete worked example with a fast part and a slow part

## The Problem: One Slow Fetch Blocks Everything

Imagine a dashboard with a quick header and a slow report.

```tsx
// app/dashboard/page.tsx
export default async function Dashboard() {
  const user = await getUser();          // fast: 50ms
  const report = await getSlowReport();  // slow: 3 seconds

  // The WHOLE page waits 3 seconds before anything shows.
  return (
    <main>
      <h1>Welcome, {user.name}</h1>
      <Report data={report} />
    </main>
  );
}
```

Even though the header was ready almost instantly, the user sees nothing for 3
seconds because the page can only be sent once everything is done.

## The Fix: `<Suspense>`

`<Suspense>` lets you wrap a slow part and give it a temporary placeholder (the
`fallback`). Next.js sends the rest of the page right away, shows the fallback
where the slow part will go, and **streams in** the real content when it is
ready.

```tsx
// app/dashboard/page.tsx
import { Suspense } from "react";

export default async function Dashboard() {
  const user = await getUser(); // still fast

  return (
    <main>
      <h1>Welcome, {user.name}</h1>

      {/* Show a placeholder now; stream the real report in later. */}
      <Suspense fallback={<p>Loading report...</p>}>
        <SlowReport />
      </Suspense>
    </main>
  );
}
```

Now the heading appears immediately with "Loading report..." underneath, and the
report pops in a few seconds later. The user is never staring at a blank page.

## Splitting the Slow Fetch Into Its Own Component

For `<Suspense>` to work, the slow `await` must live **inside** the wrapped
component, not in the parent. So we move the slow fetch into its own async Server
Component.

```tsx
// app/dashboard/SlowReport.tsx
export default async function SlowReport() {
  // This slow await is now INSIDE the suspended component.
  const report = await getSlowReport(); // 3 seconds

  return (
    <section>
      <h2>Monthly Report</h2>
      <p>Total sales: {report.total}</p>
    </section>
  );
}
```

The rule of thumb: **whatever you `await` inside a component, that component is
what `<Suspense>` waits on.** Keep the slow await in the suspended child, and keep
the parent fast.

### Why this is better

```text
Without Suspense:  [............ wait 3s ............] then show whole page
With Suspense:     show header now  ->  [stream report in after 3s]
```

The total work is the same 3 seconds, but the user *feels* the page as fast
because they see and can read content immediately. That feeling is called
**perceived performance**, and it matters a lot.

## Streaming Multiple Slow Parts

You can wrap several slow pieces independently. Each one streams in the moment it
is ready, not waiting for the others.

```tsx
// app/dashboard/page.tsx
import { Suspense } from "react";

export default async function Dashboard() {
  return (
    <main>
      <h1>Dashboard</h1>

      {/* These two load independently and appear as soon as each finishes. */}
      <Suspense fallback={<p>Loading sales...</p>}>
        <SalesPanel />
      </Suspense>

      <Suspense fallback={<p>Loading reviews...</p>}>
        <ReviewsPanel />
      </Suspense>
    </main>
  );
}
```

If sales takes 1 second and reviews takes 4, the sales panel shows after 1 second
without waiting for reviews. Each `<Suspense>` is its own little "bring it out
when ready" zone.

## How `loading.tsx` Uses Streaming Automatically

You already met `loading.tsx` in the
[Loading and Error UI](../02-routing/06-loading-and-error-ui.md) lesson. Behind
the scenes, Next.js wraps your whole page in a `<Suspense>` and uses your
`loading.tsx` as the fallback.

```tsx
// app/dashboard/loading.tsx
export default function Loading() {
  // Next.js shows this instantly while the page's data loads,
  // then streams in the real page. No <Suspense> to write yourself.
  return <p>Loading dashboard...</p>;
}
```

So there are two levels:

- `loading.tsx` gives the **whole route** a loading screen, for free.
- `<Suspense>` gives you **fine-grained** control over one slow part while the
  rest of the page is already visible.

Use `loading.tsx` for the big picture, and `<Suspense>` when you want some of the
page to show before the slow bits finish.

## Worked Example: Product Page With Reviews

A product page where the product details are fast but the reviews are slow.

```tsx
// app/products/[id]/page.tsx
import { Suspense } from "react";
import Reviews from "./Reviews";

export default async function ProductPage({
  params,
}: {
  params: Promise<{ id: string }>;
}) {
  const { id } = await params;

  // Fast fetch: the product itself.
  const res = await fetch(`https://api.example.com/products/${id}`);
  const product = await res.json();

  return (
    <main>
      <h1>{product.name}</h1>
      <p>{product.description}</p>

      {/* The slow reviews stream in; the product is visible right away. */}
      <Suspense fallback={<p>Loading reviews...</p>}>
        <Reviews productId={id} />
      </Suspense>
    </main>
  );
}
```

```tsx
// app/products/[id]/Reviews.tsx
export default async function Reviews({ productId }: { productId: string }) {
  // Pretend this endpoint is slow.
  const res = await fetch(
    `https://api.example.com/products/${productId}/reviews`
  );
  const reviews: { id: number; text: string }[] = await res.json();

  return (
    <ul>
      {reviews.map((r) => (
        <li key={r.id}>{r.text}</li>
      ))}
    </ul>
  );
}
```

The shopper sees the product name, description, and price instantly, and the
reviews fade in a moment later under a friendly placeholder. That is streaming in
action.

## Common Mistakes

- **Leaving the slow `await` in the parent.** If the parent awaits the slow data,
  `<Suspense>` has nothing to stream and the whole page still waits. Move the
  slow fetch into the wrapped child.
- **Forgetting a `fallback`.** `<Suspense>` needs a `fallback` to show while it
  waits; without one the user sees nothing in that spot.
- **Wrapping everything in one giant `<Suspense>`.** Then the page only shows once
  the slowest thing finishes. Wrap slow parts separately so fast parts show
  first.
- **Thinking streaming makes data faster.** It does not change how long the fetch
  takes; it improves how soon the user sees *something*.
- **Writing `<Suspense>` and `loading.tsx` for the same goal.** Use `loading.tsx`
  for the whole route and `<Suspense>` for individual slow pieces.

## Exercises

1. Make a page with a fast heading and a slow component (simulate slowness with a
   short delay). Wrap the slow part in `<Suspense>` and watch the heading appear
   first.
2. Move a slow `await` out of the parent and into its own child component so
   `<Suspense>` can stream it.
3. Add two independent `<Suspense>` boundaries on one page and give each a
   different fallback. Make one slower and confirm the faster one shows first.
4. Add a `loading.tsx` to a route and explain in a comment how it relates to
   `<Suspense>`.
5. On the product example, add a second slow section (for example "Related
   products") in its own `<Suspense>` boundary.

## Key Takeaways

- **Streaming** sends a page in pieces so users see fast parts immediately.
- `<Suspense fallback={...}>` shows a placeholder for a slow part and streams in
  the real content when ready.
- The slow `await` must live **inside** the suspended component, so split it into
  its own async component.
- Separate `<Suspense>` boundaries load independently; each shows as soon as it is
  ready.
- `loading.tsx` is streaming for a whole route automatically; `<Suspense>` gives
  fine-grained control.
- Streaming improves **perceived performance**, the feeling of speed, even when
  total load time is unchanged.

---

[← Previous: Caching and Revalidation](03-caching-and-revalidation.md) | [Back to Course Index](../README.md) | [Next: Server Actions and Mutations →](05-server-actions.md)
