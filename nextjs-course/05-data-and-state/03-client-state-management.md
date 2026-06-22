# Client State Management

So far most of our data has come from the **server** — the database, an API,
the URL. But some data only matters in the browser, right now, while the user
is clicking around: is a menu open? what did they type into a search box? which
tab is selected? That kind of "right now in the browser" data is called
**client state**.

Think of server data as the contents of a library (shared, stored, the source
of truth) and client state as the sticky note on the desk in front of one
reader (temporary, personal, gone when they leave). You manage them very
differently.

The big lesson of this whole module: **prefer server data and keep client state
small**. Reach for client state only when the UI is genuinely interactive.

## What You'll Learn

- When you actually need client state versus server data
- `useState` for simple local state and `useReducer` for complex state
- Sharing state with React **Context** — and what it costs
- Using the **URL** (searchParams) as state, often the best choice in Next.js
- When a state library (Zustand, Redux Toolkit, Jotai) earns its place
- What React Query and SWR do, briefly
- A rule of thumb to avoid over-engineering

## Server Data or Client State?

Ask one question: **does this value need to survive a refresh or be the same
for everyone?**

- **Yes** → it is server data. Keep it in the database / fetch on the server.
- **No, it is just this user's current screen** → it is client state.

```text
Examples of server data:        Examples of client state:
- The list of products          - Is the dropdown open?
- A user's saved profile        - The current value of an input
- Past orders                   - Which accordion panel is expanded
- Blog post content             - A "did you click yet" toggle
```

Client state lives in a **Client Component** (`"use client"`), because only the
browser can react to clicks and typing.

## Local State: `useState`

`useState` is React's tool for a single piece of state inside one component.
It gives you the value and a function to change it.

```tsx
// app/components/counter.tsx
"use client"; // client state needs a Client Component
import { useState } from "react";

export default function Counter() {
  // count = current value, setCount = how we change it. Start at 0.
  const [count, setCount] = useState(0);

  return (
    <button onClick={() => setCount(count + 1)}>
      Clicked {count} times
    </button>
  );
}
```

This is the right tool 90% of the time: a toggle, an input value, a counter.

## Complex State: `useReducer`

When several values change together by clear "actions", `useReducer` keeps the
logic tidy. A **reducer** is just a function that takes the current state and an
action, and returns the next state.

```tsx
// app/components/cart-quantity.tsx
"use client";
import { useReducer } from "react";

// The state shape and the actions we allow.
type State = { quantity: number };
type Action = { type: "increment" } | { type: "decrement" } | { type: "reset" };

// Given the old state + an action, return the new state.
function reducer(state: State, action: Action): State {
  switch (action.type) {
    case "increment":
      return { quantity: state.quantity + 1 };
    case "decrement":
      // Never go below zero.
      return { quantity: Math.max(0, state.quantity - 1) };
    case "reset":
      return { quantity: 0 };
  }
}

export default function CartQuantity() {
  const [state, dispatch] = useReducer(reducer, { quantity: 1 });

  return (
    <div>
      <button onClick={() => dispatch({ type: "decrement" })}>-</button>
      <span> {state.quantity} </span>
      <button onClick={() => dispatch({ type: "increment" })}>+</button>
      <button onClick={() => dispatch({ type: "reset" })}>reset</button>
    </div>
  );
}
```

Rule of thumb: a couple of independent values → `useState`. Several values that
change together with named actions → `useReducer`.

## Sharing State: React Context

`useState` lives in one component. What if many components, far apart in the
tree, need the same value (like the current theme)? Passing it down through
every layer as props ("prop drilling") gets painful. **Context** lets you
provide a value at the top and read it anywhere below.

```tsx
// app/theme-context.tsx
"use client";
import { createContext, useContext, useState } from "react";

// Create the context with a default.
const ThemeContext = createContext<{
  theme: string;
  toggle: () => void;
}>({ theme: "light", toggle: () => {} });

export function ThemeProvider({ children }: { children: React.ReactNode }) {
  const [theme, setTheme] = useState("light");
  const toggle = () => setTheme((t) => (t === "light" ? "dark" : "light"));

  // Everything inside can now read theme + toggle.
  return (
    <ThemeContext.Provider value={{ theme, toggle }}>
      {children}
    </ThemeContext.Provider>
  );
}

// A small hook so components can read the context cleanly.
export function useTheme() {
  return useContext(ThemeContext);
}
```

```tsx
// app/components/theme-button.tsx
"use client";
import { useTheme } from "../theme-context";

export default function ThemeButton() {
  const { theme, toggle } = useTheme(); // read from anywhere below the provider
  return <button onClick={toggle}>Theme: {theme}</button>;
}
```

**The cost of Context:** when the context value changes, **every** component
that reads it re-renders. So Context is great for values that change rarely
(theme, current user, language) and a poor fit for fast-changing values (like
text being typed). Use it sparingly.

## The URL as State (Often the Best Choice)

Here is a Next.js superpower beginners often miss. For things like the current
search term, filters, the active tab, or the page number — store them in the
**URL** as query parameters (`searchParams`) instead of in client state.

Why the URL is often better:

- The page is **shareable** — copy the link and the same view appears.
- It survives **refresh** and the **back button** works naturally.
- A Server Component can read it and fetch the right data on the server.

```tsx
// app/products/page.tsx  (Server Component)
// Next reads ?category=shoes from the URL for us.
export default async function ProductsPage({
  searchParams,
}: {
  searchParams: Promise<{ category?: string }>;
}) {
  const { category } = await searchParams; // read the URL state
  // Fetch on the server based on the URL — no client state needed.
  const products = await getProducts(category);

  return <ProductList products={products} />;
}
```

To change the URL from a Client Component, use the router:

```tsx
// app/components/category-picker.tsx
"use client";
import { useRouter, useSearchParams } from "next/navigation";

export default function CategoryPicker() {
  const router = useRouter();
  const params = useSearchParams();

  function pick(category: string) {
    // Update the URL; the server re-runs and returns new data.
    const next = new URLSearchParams(params);
    next.set("category", category);
    router.push(`/products?${next.toString()}`);
  }

  return <button onClick={() => pick("shoes")}>Shoes</button>;
}
```

> Guideline: if a value describes "what the user is looking at", try the URL
> first. Keep `useState` for transient UI like "is this menu open".

## When to Reach for a State Library

Libraries like **Zustand**, **Redux Toolkit**, and **Jotai** manage global
client state. They are useful when:

- Many unrelated components share complex state that changes often.
- You want state to live **outside** React's component tree.
- Context would cause too many re-renders or get unwieldy.

But in a Next.js app where most data is server data and most UI state is local,
you often need **none** of them. Do not add a library on day one "just in case".

```ts
// store.ts  — a tiny Zustand example (only if you truly need global state)
import { create } from "zustand";

// Global state any component can read/update, without a provider.
export const useCartStore = create<{
  items: number;
  add: () => void;
}>((set) => ({
  items: 0,
  add: () => set((s) => ({ items: s.items + 1 })),
}));
```

Among the three: **Zustand** is the smallest/simplest, **Redux Toolkit** suits
large teams wanting strict structure, **Jotai** models state as tiny atoms.
Start with built-ins and add a library only when pain appears.

## React Query and SWR (Brief)

Sometimes a **Client Component** must fetch server data itself (for example,
data that updates live after the page loads). Doing this by hand means juggling
loading, error, and caching. **React Query** (TanStack Query) and **SWR** do
that juggling for you — they fetch, cache, and refresh server data on the
client.

```tsx
// app/components/live-price.tsx
"use client";
import useSWR from "swr";

const fetcher = (url: string) => fetch(url).then((r) => r.json());

export default function LivePrice() {
  // SWR handles loading, caching, and re-fetching for us.
  const { data, isLoading } = useSWR("/api/price", fetcher);

  if (isLoading) return <p>Loading...</p>;
  return <p>Price: {data.price}</p>;
}
```

Important distinction: this is **not** the same as `useState`. It is a cache for
**server data** that happens to live on the client. For most pages, fetching on
the **server** in a Server Component is simpler and you will not need this.

## Common Mistakes

- **Copying server data into `useState`.** It quickly goes stale. Fetch on the
  server, or use SWR/React Query if it must be client-side.
- **Using Context for fast-changing values.** Every reader re-renders. Keep
  rapidly-changing values local with `useState`.
- **Storing shareable view state (filters, page) only in `useState`.** It is
  lost on refresh and not shareable. Use the URL instead.
- **Reaching for Redux/Zustand too early.** Start with `useState`, the URL, and
  server data; add a library only when there is real pain.
- **Forgetting `"use client"`** on a component that uses hooks like `useState`.
  Hooks only run in Client Components.

## Exercises

1. For each, say where it should live (useState, URL, or server/database): a
   modal's open/closed flag, the search query, a user's saved address, the
   currently selected pricing tab.
2. Rewrite the `Counter` to also have a "reset to zero" button, still using
   `useState`.
3. Convert a hardcoded `category` filter into URL state: write the Server
   Component signature that reads `searchParams.category`.
4. Explain in one or two sentences why Context causes re-renders and when that
   is a problem.
5. Describe a situation where SWR or React Query is the right tool, and one
   where fetching on the server is better.

## Key Takeaways

- Client state is temporary, browser-only UI data; server data is the stored
  source of truth.
- Use `useState` for simple local state and `useReducer` for grouped,
  action-driven state.
- Context shares state across the tree but re-renders all readers — use it for
  rarely-changing values.
- In Next.js, prefer the **URL** for shareable view state like filters and tabs.
- Add a state library (Zustand, Redux Toolkit, Jotai) only when built-ins hurt.
- React Query/SWR cache **server** data on the client; often server fetching is
  simpler. Overall: prefer server data, keep client state minimal.

---

[← Previous: Environment Variables](02-environment-variables.md) | [Back to Course Index](../README.md) | [Next: Error Handling →](../06-production/01-error-handling.md)
