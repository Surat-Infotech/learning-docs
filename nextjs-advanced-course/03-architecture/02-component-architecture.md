# Component Architecture and Composition

In the App Router, a component isn't just a UI unit — it's a decision about *where code runs*. Every `"use client"` is a boundary that ships JavaScript to the browser and pulls everything below it onto the client. Getting composition right is the difference between an app that ships a 60 KB interactive island and one that ships 600 KB because someone marked a layout `"use client"`. This lesson is about composing server and client components so interactivity stays surgical and bundles stay small.

## What You'll Learn

- The "client components as leaves" rule and why it minimizes bundle cost
- How passing server-rendered `children` into client wrappers keeps subtrees on the server
- The container/presentational split, reframed for an RSC world
- Designing prop contracts that survive the server/client serialization boundary
- Eliminating prop drilling without making the whole tree a client component
- The real bundle cost of each `"use client"` boundary, and how to measure it
- A concrete refactor turning an all-client page into a mostly-server one

## Client Components as Leaves

The single most important compositional rule: **push `"use client"` as far down the tree as possible.** A client component and everything it imports gets bundled and shipped. Critically, every component *rendered inside* a client component is also a client component — the boundary is contagious downward through JSX children written in the same module.

So a `"use client"` near the root of a page poisons the entire page; a `"use client"` on a single button poisons only the button. Interactivity is almost always local — a toggle, a form input, a dropdown — while the surrounding structure is static. Keep the static structure on the server and make only the interactive bits client leaves.

```tsx
// Bad: the whole product page is now a client bundle for one button.
"use client";
export default function ProductPage({ product }) {
  return (
    <article>
      <ProductGallery images={product.images} />   {/* forced client */}
      <ProductDescription html={product.html} />   {/* forced client */}
      <AddToCartButton id={product.id} />          {/* the only interactive part */}
    </article>
  );
}
```

```tsx
// Good: page stays a Server Component; only the button is client.
// app/products/[id]/page.tsx
export default function ProductPage({ product }) {
  return (
    <article>
      <ProductGallery images={product.images} />     {/* server */}
      <ProductDescription html={product.html} />     {/* server */}
      <AddToCartButton id={product.id} />            {/* client leaf */}
    </article>
  );
}
```

```tsx
// components/AddToCartButton.tsx
"use client";
import { useState } from "react";
export function AddToCartButton({ id }: { id: string }) {
  const [pending, setPending] = useState(false);
  // ...interactivity stays isolated here
  return <button disabled={pending}>Add to cart</button>;
}
```

## Passing Server Children into Client Wrappers

The "contagion is downward through your JSX" rule has a crucial exception that unlocks most real layouts: **a client component can *receive* server-rendered content through props (including `children`) and render it without turning that content into client code.** The client component sees an already-rendered React node; it never imports or executes the server component's code, so nothing of the child gets bundled into the client.

This is the pattern for interactive shells around static content — accordions, tabs, modals, sidebars, theme providers:

```tsx
// components/Collapsible.tsx
"use client";
import { useState, type ReactNode } from "react";

// `children` is an opaque, already-rendered node from the server.
export function Collapsible({ children }: { children: ReactNode }) {
  const [open, setOpen] = useState(false);
  return (
    <section>
      <button onClick={() => setOpen((o) => !o)}>Toggle</button>
      {open && <div>{children}</div>}
    </section>
  );
}
```

```tsx
// app/page.tsx (Server Component)
import { Collapsible } from "@/components/Collapsible";
import { HeavyServerReport } from "@/features/reports/HeavyServerReport";

export default async function Page() {
  // HeavyServerReport renders on the server, does its DB calls, ships zero JS.
  // Collapsible (client) just wraps the already-rendered node.
  return (
    <Collapsible>
      <HeavyServerReport />
    </Collapsible>
  );
}
```

`HeavyServerReport` can be async, hit the database, and pull in heavy server-only libraries — none of it reaches the browser. The client boundary holds at `Collapsible`; the report passes *through* it as data, not as imported code. Internalize this: the `children` slot is the seam that lets interactive wrappers coexist with server-heavy content.

### Trade-offs

This only works when the wrapper doesn't need to *inspect* or *transform* the children's React internals — it just renders them in a slot. If a client component needs to map over typed data and render server components per item, that doesn't compose; you'd instead pass the data to the server parent and have it render the list, wrapping each item's interactive part as a client leaf. Design wrappers to take `children`/`ReactNode` slots rather than reaching into structure.

## Container/Presentational in an RSC World

The old container/presentational split (smart components fetch, dumb components render) maps almost perfectly onto server/client — but cleaner, because the "container" no longer needs hooks or effects to fetch.

- **Container = async Server Component.** It does data access (via the DAL), composes children, passes plain data down. No `useEffect`-fetch, no loading-spinner plumbing.
- **Presentational = either server or client.** Pure rendering for static parts (server), or client leaves for the interactive parts.

```tsx
// features/orders/OrderListContainer.tsx  (Server Component — the "container")
import { getOrders } from "./queries";
import { OrderRow } from "./OrderRow";

export async function OrderListContainer({ userId }: { userId: string }) {
  const orders = await getOrders(userId); // data access here
  return (
    <ul>
      {orders.map((o) => (
        <OrderRow key={o.id} order={o} /> // presentational, gets plain data
      ))}
    </ul>
  );
}
```

The container owns *what data* and *which children*; presentational components own *how it looks*. This keeps data access out of leaf components and lets you reuse presentational components in Storybook, tests, and multiple containers.

## Designing Prop Contracts Across the Boundary

Props passed from a Server Component to a Client Component must be **serializable** — they cross the network as part of the RSC payload. That means plain objects, arrays, strings, numbers, dates, and Server Action references are fine; class instances, functions (other than Server Actions), `Map`/`Set` in some cases, and Symbols are not.

Two contract rules pay off at scale:

1. **Pass the minimum.** Don't hand a client leaf the entire `user` object when it needs `user.name`. Narrowing props shrinks the serialized payload and decouples the leaf from your data model — a schema change to `user` doesn't ripple into the button.
2. **Pass primitives or narrow DTOs, not ORM models.** A Prisma model may carry methods or lazy relations that won't serialize. Map to an explicit DTO at the container boundary so the contract is exactly what the client needs and nothing more.

```tsx
// Narrow, serializable contract — not the whole ORM object.
type CartButtonProps = { productId: string; inStock: boolean };
```

## Killing Prop Drilling Without Going All-Client

Prop drilling tempts teams into wrapping the tree in a client Context — which forces everything under the provider to be a client component. In RSC, you usually don't need to. Three better moves:

1. **Composition / slots.** Instead of drilling a prop through five layers to a deep leaf, render the leaf at the top and pass it down as `children`. The prop is resolved where it's known.
2. **Server-side data access at the leaf.** A deep server component can just *call the DAL itself* (requests are deduped/cached), rather than receiving data drilled from the root. This is genuinely new — data "drilling" disappears because any server component can fetch.
3. **Scoped client Context only around the interactive island.** When you do need shared client state (a wizard's step, a selected tab), wrap only the smallest interactive subtree in a provider — not the page.

```tsx
// Scope the client Context to the island, not the route.
// The page stays server; only WizardShell + its inputs are client.
<WizardShell>            {/* "use client" provider — small island */}
  <StepIndicator />      {/* client, consumes context */}
  <StepFields />         {/* client, consumes context */}
</WizardShell>
```

## The Cost of Each `"use client"` Boundary

Each boundary has measurable cost: the component's own code, its imports (which is where it bites — a date library, an icon set, a charting lib all get pulled into the client chunk), and the hydration work React does on the client to attach interactivity. A `"use client"` that imports a 100 KB library and sits high in the tree is a real performance regression, even if the component itself is small.

Measure it. After a build, inspect chunk sizes with `@next/bundle-analyzer`:

```bash
# next.config: withBundleAnalyzer({ enabled: process.env.ANALYZE === "true" })
ANALYZE=true next build
```

Look for client chunks that are surprisingly large and trace them to a high boundary or a heavy import that should be `dynamic()`-imported or moved server-side.

### Trade-offs: when to use what

- **All-server (no boundary):** static content, data display. Default to this.
- **Server children into a client wrapper:** interactive shell around static content. Preferred for layout-level interactivity.
- **Client leaf:** localized interactivity (buttons, inputs, toggles). The workhorse.
- **Scoped client Context:** shared state across a few interactive siblings. Use sparingly, scoped tight.
- **`dynamic(() => import(...), { ssr: false })`:** heavy client-only widgets (maps, editors) you want lazy-loaded and out of the initial bundle.

## Pitfalls

- **`"use client"` on a layout or page.** Poisons the whole subtree. Almost never correct; move it to a leaf.
- **Wrapping the tree in a Context provider for one deep value.** Forces the subtree client-side. Prefer composition/slots or DAL-at-leaf.
- **Passing ORM models or non-serializable props across the boundary.** Causes runtime serialization errors or silently bloats the payload. Map to DTOs.
- **Heavy imports inside small client components.** The component is tiny; its import graph isn't. Audit with the bundle analyzer.
- **Forgetting that server `children` passed into a client component stay server.** Teams needlessly mark content client because they didn't know the slot pattern.
- **Over-narrowing into prop soup.** Passing twelve scalar props instead of one small DTO. Balance minimalism with a coherent contract.

## Exercises

1. Take a page where the root is `"use client"` for a single search input. Refactor so the page is a Server Component and only the input is a client leaf. Diagram the new boundary.
2. Design a `Tabs` component (client) that renders server-rendered panels passed as `children`. Show how a panel can be an async server component doing a DB read while staying off the client bundle.
3. Given a `getDashboard()` that returns a deeply nested object, design the DTO contract for a `RefreshButton` client leaf so it receives only what it needs. Justify each field.
4. Identify three places in a typical dashboard where a Context provider would force unnecessary client rendering, and rewrite each using composition or DAL-at-leaf.
5. Run `ANALYZE=true next build` on a sample app, find the largest client chunk, and write a one-paragraph plan to shrink it (move boundary down, `dynamic()` import, or server-side the work).

## Key Takeaways

- Push `"use client"` to the leaves; the boundary is contagious downward through your JSX.
- A client component can render server-rendered `children` without making them client — the slot pattern is the key to interactive shells around static content.
- Container = async Server Component doing data access; presentational = server or client leaves doing rendering.
- Props crossing the boundary must be serializable; pass narrow DTOs, not ORM models or the whole object.
- Beat prop drilling with composition, DAL-at-leaf, and tightly scoped Context — not a root-level client provider.
- Every boundary ships JS (mostly via its imports); measure with the bundle analyzer and treat a high, heavy boundary as a perf bug.

---

[← Previous: Project Architecture at Scale](01-project-architecture.md) | [Back to Course Index](../README.md) | [Next: State Management at Scale →](03-state-management.md)
