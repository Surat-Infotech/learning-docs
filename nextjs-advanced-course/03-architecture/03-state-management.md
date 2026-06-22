# State Management at Scale

"Which state library should we use?" is the wrong first question. The right one is "what *kind* of state is this, and where does it actually belong?" Most state in a Next.js app is not client state at all — it's server data, or it's URL state masquerading as component state. Reaching for Zustand or Redux before answering that question is how apps accumulate redundant client stores that fight the server, go stale, and bloat the bundle. This lesson gives you a taxonomy and a placement rule for each kind of state.

## What You'll Learn

- A taxonomy of state: server/data, URL, ephemeral UI, and global client state
- Why server state and URL state should absorb most of what you'd otherwise store client-side
- When global client state is genuinely justified — and the cost of getting there
- Zustand vs Jotai vs Redux Toolkit, with concrete trade-offs
- How SWR/React Query coexist with RSC instead of competing with it
- The real cost of React Context and the patterns that keep it cheap
- Keeping the client component tree small as a guiding constraint

## A Taxonomy of State

Before choosing a tool, classify the state. There are four kinds, and they have different homes:

1. **Server / data state** — the source of truth lives in your database or an upstream API. Lists, records, the current user's orders. *This is not client state.* It's a cache of server data.
2. **URL state** — anything that should survive a refresh, be shareable, and back-buttonable: the active tab, filters, pagination, search query, sort order. Belongs in the URL (`searchParams`).
3. **Ephemeral UI state** — transient, local, throwaway: is this dropdown open, is this input focused, the in-progress value of a controlled field. Belongs in `useState`/`useReducer` in the nearest client component.
4. **Global client state** — genuinely cross-cutting, client-only, and not derivable from the above: a multi-step form spanning routes, an optimistic shopping cart before checkout, client-side feature flags. The smallest and rarest category.

The mistake at scale is collapsing all four into one global store. Each misplacement has a tax: server data in a client store goes stale and duplicates fetching; URL state in `useState` breaks refresh and sharing; ephemeral state in a global store creates needless re-renders and coupling.

## Prefer Server State and URL State

In the App Router, the server is your primary state container. Server Components read from the DAL on each request; the "store" is the database plus Next's request/data cache. You don't fetch-into-state-and-render; you render from data directly. That eliminates an entire class of client state — and the loading/error/stale-sync machinery that comes with it.

URL state is the second pillar and chronically underused. The active filter set, current page, and sort order should be `searchParams`, not React state:

```tsx
// app/products/page.tsx  (Server Component)
// URL state drives a server re-render with fresh, filtered data.
export default async function Products({
  searchParams,
}: {
  searchParams: Promise<{ category?: string; sort?: string; page?: string }>;
}) {
  const { category, sort, page = "1" } = await searchParams;
  const products = await getProducts({ category, sort, page: Number(page) });
  return <ProductGrid products={products} />;
}
```

```tsx
// components/CategoryFilter.tsx  (client leaf)
"use client";
import { useRouter, useSearchParams, usePathname } from "next/navigation";

export function CategoryFilter() {
  const router = useRouter();
  const pathname = usePathname();
  const params = useSearchParams();

  function setCategory(category: string) {
    const next = new URLSearchParams(params);
    next.set("category", category);
    // Updates the URL → server re-renders the page with new data. No client store.
    router.push(`${pathname}?${next.toString()}`);
  }
  return <select onChange={(e) => setCategory(e.target.value)}>{/* ... */}</select>;
}
```

The payoff: this state is shareable (paste the URL), survives refresh, is back-button-friendly, and needs zero client state library. It's the most underrated state-management technique in Next. For heavy filter UIs, libraries like `nuqs` give typed, debounced `searchParams` hooks while keeping the URL as the source of truth.

### Trade-offs

URL state has limits: it's stringly-typed, visible to the user, and length-bounded. Don't put large blobs or secrets there. For complex but still shareable state, encode compactly (or use `nuqs`). For state that shouldn't be in the URL but must survive navigation within a session, that's where client state earns its place.

## When Global Client State Is Justified

Reach for a global client store only when *all* of these hold: the state is client-only (not server-derived), it must be shared across components that aren't in a parent/child relationship, and it shouldn't (or can't) live in the URL. Real examples: an optimistic cart that aggregates before a checkout submit, a multi-step wizard's accumulated answers across routes, complex client-side editor state.

If the state is server data, the answer is a data-fetching cache (below), not a store. If it's shareable view state, the answer is the URL. The residual is small — and that's the point.

### Zustand vs Jotai vs Redux Toolkit

| Library | Model | Best when | Cost |
| --- | --- | --- | --- |
| **Zustand** | Single store, hook-based selectors | Most apps. Minimal boilerplate, tiny bundle, no provider needed | Easy to over-centralize into one mega-store |
| **Jotai** | Atomic, bottom-up composition | Fine-grained, derived state; lots of independent pieces | Atom sprawl; mental model shift |
| **Redux Toolkit** | Centralized store, reducers, middleware | Large teams needing strict conventions, time-travel debugging, rich middleware/devtools | Most boilerplate and bundle; usually overkill in RSC apps |

```ts
// features/cart/store.ts  — Zustand, the pragmatic default
"use client";
import { create } from "zustand";

type CartState = {
  items: { id: string; qty: number }[];
  add: (id: string) => void;
};

export const useCart = create<CartState>((set) => ({
  items: [],
  add: (id) =>
    set((s) => ({ items: [...s.items, { id, qty: 1 }] })),
}));
```

For most teams, **Zustand is the right default**: smallest footprint, no provider wrapping the tree (so it doesn't force client boundaries), selector-based subscriptions that avoid over-rendering. Jotai shines when state decomposes naturally into many small atoms. Redux Toolkit is justified mainly by team-scale conventions and its devtools/middleware ecosystem — but in an RSC app where the server holds most state, its weight rarely pays off. Whatever you choose, keep the store on the client side of a small boundary and hydrate it from server-passed props rather than re-fetching.

## SWR / React Query Alongside RSC

RSC handles server data fetched *during render on the server*. But some data is inherently client-driven: it depends on client-only inputs, needs polling/refetch-on-focus, or powers a highly interactive widget that fetches after user actions without a navigation. That's where TanStack Query (React Query) and SWR fit — they're caches for *client-fetched* server data, with deduping, background revalidation, and mutation handling.

They don't compete with RSC; they complement it. The pattern that works at scale:

- **Initial data: server.** Fetch on the server in a Server Component, pass it as the client cache's initial/`fallbackData`. The first paint has real data, no client fetch waterfall.
- **Subsequent interactions: client cache.** Polling, refetch-on-focus, optimistic mutations, infinite scroll run client-side through the query cache.

```tsx
// Server Component fetches initial data, hands it to a client provider.
// app/feed/page.tsx
import { getFeed } from "@/features/feed/queries";
import { FeedClient } from "@/features/feed/FeedClient";

export default async function FeedPage() {
  const initial = await getFeed(); // server fetch, no waterfall
  return <FeedClient initialData={initial} />;
}
```

```tsx
// features/feed/FeedClient.tsx
"use client";
import { useQuery } from "@tanstack/react-query";

export function FeedClient({ initialData }: { initialData: Feed }) {
  // Hydrated from server data; refetches on focus / polling thereafter.
  const { data } = useQuery({
    queryKey: ["feed"],
    queryFn: () => fetch("/api/feed").then((r) => r.json()),
    initialData,
  });
  return <FeedList feed={data} />;
}
```

### Trade-offs

If your data needs are "render the current state of server data on navigation," plain RSC + `revalidatePath`/Server Actions covers it with *less* code and no client cache. Add React Query when you have real client-side fetching needs: focus revalidation, polling, optimistic updates, infinite queries. Adding it for data you could fetch on the server is needless client weight and a second source of truth to keep coherent.

## React Context Cost and Patterns

Context is not a state manager — it's a dependency-injection mechanism, and every consumer re-renders when the value changes (unless you split contexts or memoize). Two costs matter in Next: a provider makes its subtree a client boundary, and a broad context with a frequently-changing value re-renders everything beneath it.

Patterns that keep Context cheap:

- **Split by change frequency.** Put rarely-changing config (theme, locale) in one context and volatile values in another, so a fast-changing value doesn't re-render config consumers.
- **Memoize the value.** `useMemo` the provider's `value` so referential identity is stable.
- **Scope tight.** Wrap the smallest subtree that needs it, never the route. A provider at the layout level forces the layout client-side.
- **Prefer composition or a store for cross-cutting client state.** Zustand's selectors re-render only subscribed components, avoiding Context's broad-invalidation problem.

## Keeping the Client Tree Small

The unifying constraint behind every choice above: **minimize the client component tree.** Server state needs no client. URL state needs only a tiny client leaf to push the route. Ephemeral state lives in small leaves. Global stores and query caches sit on the client side of a small boundary, hydrated from server data. The smaller the client tree, the less JS ships, the less hydration runs, and the fewer sources of truth you maintain. Treat every new client-state mechanism as a request to grow that tree — and make it justify itself.

## Pitfalls

- **Mirroring server data into a client store.** Now you have two sources of truth that drift. Render from server data or cache it with React Query, don't copy it.
- **Filters/tabs/pagination in `useState`.** Breaks refresh, sharing, and back button. Put it in `searchParams`.
- **One global store for everything.** Couples unrelated features and causes broad re-renders. Classify state first.
- **Context provider at the layout/route level.** Forces a large client boundary. Scope it to the island.
- **React Query for data you fetch on the server anyway.** Redundant cache, extra bundle, waterfall risk. Use it only for client-driven fetching.
- **Unmemoized Context value.** Every render creates a new object, re-rendering all consumers.

## Exercises

1. Take a product-listing page with client `useState` for category, sort, and page. Move all three to `searchParams` and write the client filter leaf that updates the URL. Note what now works that didn't (refresh, share, back).
2. Classify the state in a checkout flow (cart contents, shipping form values, selected payment tab, the confirmed order) into the four categories and assign each a home. Justify the cart's placement specifically.
3. Design a Zustand cart store hydrated from server-passed initial items. Show the boundary placement so the store doesn't force the page client-side.
4. You have a live-updating notifications dropdown (polls every 30s). Decide between RSC+revalidation and React Query, and justify. Sketch the server-initial-data + client-cache wiring.
5. Audit a context that holds `{ theme, user, cartCount }` where `cartCount` changes often. Split it to avoid re-rendering theme consumers and explain the re-render behavior before and after.

## Key Takeaways

- Classify state into server, URL, ephemeral UI, and global client — then place each in its proper home.
- Server state and URL state should absorb most of what tutorials store in client state; they're shareable, refresh-safe, and library-free.
- Justify global client state only when it's client-only, cross-cutting, and not URL-appropriate; Zustand is the pragmatic default, Jotai for atomic state, Redux Toolkit mainly for team conventions.
- React Query/SWR complement RSC for client-driven fetching; seed them with server-fetched initial data, don't use them to re-fetch what the server already renders.
- Context is DI, not a store; split by change frequency, memoize, and scope tight to avoid re-renders and client boundaries.
- The governing constraint is a small client tree — every client-state mechanism must justify the JS and the extra source of truth it adds.

---

[← Previous: Component Architecture and Composition](02-component-architecture.md) | [Back to Course Index](../README.md) | [Next: End-to-End Type Safety →](04-type-safety.md)
