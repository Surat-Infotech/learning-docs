# The RSC Architecture in Depth

React Server Components are the foundation everything else in this course sits on. Most developers use them correctly by accident and then hit a wall — a serialization error, a mysteriously huge client bundle, a hydration mismatch — because they don't know the machinery underneath. This lesson takes the lid off: how the server render becomes a payload, what can and can't cross the boundary, and the composition patterns that let you keep your client bundle tiny.

## What You'll Learn

- The full RSC pipeline: server render → serialized payload → client reconciliation.
- The serialization boundary — exactly what props can and cannot cross.
- How `"use client"` splits your bundle and why it's a boundary, not a file label.
- The composition pattern that lets server-rendered content live *inside* a Client Component.
- Why importing a module into a Client Component drags its dependencies into the bundle.
- How to reason about and audit your client/server split.

## How RSC Actually Works

When a route renders, React runs your Server Components **on the server** and produces not HTML but an intermediate format called the **RSC payload** (sometimes "the flight stream"). Walk through what happens:

```text
1. Server renders the tree top-down.
   - Server Components execute: they await data, run logic, return elements.
   - When the renderer hits a Client Component, it does NOT run its code.
     It emits a reference: "render <ProductCard> here, with these props,
     and load its JS chunk."
2. The output is serialized into the RSC payload — a compact stream describing:
   - The rendered Server Component output (already-resolved UI).
   - Placeholders + props + module references for each Client Component.
3. The server also produces HTML (from the same render) for first paint/SEO.
4. The browser receives HTML, paints it, then downloads Client Component JS.
5. React reconciles: it reads the RSC payload, hydrates the Client Components
   into the existing HTML, and now holds the tree as its source of truth.
```

The key mental model: **the server already did the rendering work for Server Components.** Their code never ships. What ships for them is the *result*. For Client Components, the server ships a reference and props, and the browser downloads and runs the actual code.

### Why this matters for navigation

On a client-side navigation, Next.js doesn't fetch a new HTML document — it fetches a fresh RSC payload for the changed segments. React reconciles it against the current tree, swapping in new server-rendered output and reusing Client Component state where the tree shape is preserved. This is why navigations feel instant and why your Server Components can re-run on navigation without a full reload.

## The Serialization Boundary

Everything that crosses from a Server Component to a Client Component as props must be **serializable**, because it has to be encoded into the RSC payload and sent over the wire. This is the rule that trips people up most.

**Can cross:**

- Primitives: strings, numbers, booleans, `null`, `undefined`, `BigInt`.
- Plain objects and arrays of serializable values.
- `Date`, `Map`, `Set`, typed arrays, and other built-ins React's serializer supports.
- JSX / React elements (this is huge — see composition below).
- **Server Actions** — the one kind of function that can cross, because it serializes to a reference, not the function body.
- Promises (a Server Component can pass a promise into a Client Component, which then `use()`s it).

**Cannot cross:**

- Arbitrary functions, callbacks, event handlers (`onClick={someFn}`).
- Class instances with methods (an ORM model object, a `class` with behavior).
- Symbols (except well-known ones).
- Anything whose value is *behavior* rather than *data*.

```tsx
// app/products/page.tsx — Server Component
import { ProductCard } from "./_components/product-card"; // client
import { getProduct } from "@/features/catalog/server";

export default async function Page() {
  const product = await getProduct("abc"); // a plain serializable object

  return (
    <ProductCard
      product={product} // OK: plain object
      // onAddToCart={() => addToCart(product)} // ERROR: a function can't cross
    />
  );
}
```

If you need the client to *do* something, give it a Server Action (a function that serializes to a reference) or move the handler into the Client Component itself. The boundary forces you to be explicit about what is data and what is behavior.

### A subtle one: passing class instances

A frequent production bug: you fetch a row with an ORM that returns a rich model instance, then pass it to a Client Component. The methods silently vanish (or you get a serialization error). Map it to a plain object first:

```ts
// In server code, before crossing the boundary:
const dto = { id: user.id, name: user.name, email: user.email };
// Pass `dto`, not the live model instance.
```

## `"use client"` Is a Boundary, Not a File Label

This is the single most misunderstood directive in Next.js. `"use client"` does **not** mean "this file runs on the client." It marks the **entry point** where the tree crosses from the server graph into the client graph. Once you cross it:

- That component **and every module it imports** become part of the client bundle.
- Child components rendered *inside* it are still part of the client graph **unless** they're passed in as props/children from a Server Component (see composition).

```tsx
// app/_components/search-box.tsx
"use client"; // boundary starts HERE

import { useState } from "react";
import { fuzzySearch } from "@/lib/fuzzy"; // pulled into the CLIENT bundle
import { format } from "@/lib/dates";       // also pulled in — even if huge

export function SearchBox() {
  const [q, setQ] = useState("");
  // ...
}
```

The consequence: a single `"use client"` at the top of a component that imports a heavy library (a charting lib, a date library, a markdown renderer) ships all of it to every visitor. The fix is to push the boundary down to the smallest component that genuinely needs interactivity, and keep heavy/server-capable work above the line.

### The contagion model

```text
Server Component (no JS shipped)
  └─ imports → ClientComponent  ← "use client" boundary
                  └─ imports → helper.ts      ← client bundle
                  └─ imports → big-library    ← client bundle
                  └─ renders  → AnotherClient ← client bundle
```

Everything below and reachable-by-import from a `"use client"` entry point is client code. "Reachable by import" is the operative phrase — not "rendered inside."

## Composition: Server Components Inside Client Components

Here's the pattern that confuses everyone and, once understood, becomes your most powerful tool. **You can render a Server Component inside a Client Component — by passing it as `children` (or any prop), not by importing it.**

A Client Component cannot *import* a Server Component (importing would pull it into the client graph and it would lose its server powers). But it can accept already-rendered server output as a prop:

```tsx
// app/_components/collapsible.tsx
"use client";
import { useState, type ReactNode } from "react";

export function Collapsible({ children }: { children: ReactNode }) {
  const [open, setOpen] = useState(false);
  return (
    <div>
      <button onClick={() => setOpen((o) => !o)}>Toggle</button>
      {open && children} {/* children may be server-rendered content */}
    </div>
  );
}
```

```tsx
// app/page.tsx — Server Component
import { Collapsible } from "./_components/collapsible";
import { getReport } from "@/features/reports/server";

export default async function Page() {
  const report = await getReport(); // server-only data fetch

  return (
    <Collapsible>
      {/* This subtree renders on the server. Collapsible never sees its code,
          only the resulting RSC payload, slotted into `children`. */}
      <ServerReport data={report} />
    </Collapsible>
  );
}
```

`Collapsible` controls the *interactivity* (open/close) on the client, while `ServerReport` runs entirely on the server with full data access and ships zero JS of its own. The Client Component is a thin interactive wrapper around server-rendered content.

### Trade-offs: where to draw the boundary

| Approach | Client JS | Data access | Use when |
| --- | --- | --- | --- |
| Whole page `"use client"` | Largest | Via client fetch / API | Truly app-like, highly interactive surfaces |
| Client leaf, import-based | Subtree's worth | Props from server parent | A self-contained interactive widget |
| Client shell + server `children` | Smallest | Server children fetch directly | Interactive layout wrapping mostly-static content |

The third pattern — interactive shell, server children — should be your reflex whenever interactivity wraps content rather than *being* the content. It's how you keep tabs, modals, accordions, and layout chrome interactive without dragging their contents into the client bundle.

## Pitfalls

- **Importing a Server Component into a Client Component.** It silently becomes a Client Component (or errors if it uses server-only APIs). Pass it as `children` instead.
- **Passing functions or class instances across the boundary.** Only data and Server Actions cross. Map ORM models to plain DTOs first.
- **One heavy import behind `"use client"`.** A date or chart library imported at a client boundary ships to everyone. Audit your bundle and push boundaries down.
- **Assuming `"use client"` opts a single file out of SSR.** Client Components still render on the server for the initial HTML; the directive controls hydration and bundling, not "skip the server."
- **Reaching for `useEffect` to fetch data that a Server Component could fetch directly.** That's a client round-trip you don't need; fetch on the server and pass props.

## Exercises

1. **Trace a payload.** For a page with one Server Component fetching data and one interactive Client Component, write out (in prose) what the RSC payload contains versus what HTML is sent versus what JS is downloaded.
2. **Fix a serialization error.** Given a Server Component passing an ORM model instance with methods to a Client Component, refactor it to pass a serializable DTO and explain what would have broken.
3. **Lower a boundary.** Take a page-level `"use client"` (whole page interactive) and refactor it so only the genuinely interactive widget is a Client Component, with the rest server-rendered. Estimate the client-JS reduction.
4. **Composition refactor.** Build a `<Tabs>` Client Component whose tab *panels* are server-rendered content passed as children, each fetching its own data on the server.
5. **Bundle audit.** Run `next build` on a project and inspect the per-route JS sizes. Identify the largest client chunk and trace which `"use client"` boundary and which import is responsible.

## Key Takeaways

- Server Components render on the server into an RSC payload; their code never ships — only their result plus references and props for Client Components.
- Props crossing the boundary must be serializable; functions and class instances can't cross, but Server Actions and JSX can.
- `"use client"` marks a boundary entry point — it and everything it imports join the client bundle. It is contagious by import.
- Render Server Components *inside* Client Components by passing them as `children`/props, never by importing them.
- The "interactive client shell + server children" pattern is the key to keeping the client bundle small while preserving interactivity.

---

[← Previous: Production-Grade Project Setup](../00-orientation/02-project-setup.md) | [Back to Course Index](../README.md) | [Next: Rendering Strategies and PPR →](02-rendering-strategies.md)
