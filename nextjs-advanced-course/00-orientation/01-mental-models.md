# Mental Models for Advanced Next.js

You already know how to build with Next.js. This course is about building so that it still works at a million requests a day, across a team, on a budget. That shift starts in your head: the same code means different things depending on *where* and *when* it runs. This lesson rebuilds your mental model of a Next.js request so every later decision — caching, streaming, actions, cost — has a frame to hang on.

## What You'll Learn

- The full request lifecycle: edge → server → React render → stream → hydration.
- Why the server/client boundary is the single most important design axis in your app.
- How the RSC payload differs from HTML, and why both are sent.
- The "server-first, push interactivity to the leaves" philosophy and when to break it.
- What changes about your thinking when latency, cost, and caching become first-class concerns.
- A vocabulary for reasoning about *where* and *when* code runs.

## The Request Lifecycle, End to End

When a user hits a route, the work flows through several distinct stages. Mentally separating them is what lets you reason about performance and correctness.

```text
1. Edge      Request hits CDN / edge network.
             - Static asset or cached page? Served immediately, done.
             - Middleware runs here (rewrites, auth gates, geo).
2. Server    The Next.js server (Node or Edge runtime) handles the route.
             - Resolves the matching layout + page segments.
             - Runs your Server Components top-down.
3. Render    React renders Server Components to an RSC payload (not HTML yet).
             - Async components await data; Suspense marks streaming holes.
4. Stream    The server flushes HTML + RSC payload progressively to the browser.
             - Shell first, then Suspense boundaries as their data resolves.
5. Hydrate   Client Components' JS arrives and attaches to the server HTML.
             - The page becomes interactive at the leaves.
```

The crucial insight: **stages 2–4 happen per request unless caching short-circuits them.** Every dynamic page you ship is server CPU you pay for. Every Client Component is JavaScript a phone has to download, parse, and hydrate. Holding the whole pipeline in mind is what separates "it works on my machine" from "it works at scale."

### The two runtimes

Server code runs in one of two runtimes, and the choice is architectural:

- **Node.js runtime** (default): full Node APIs, your ORM, `fs`, larger cold starts, runs in regions.
- **Edge runtime**: a constrained Web-standard environment, near-instant cold starts, runs geographically close to users — but no native Node APIs and limited library support.

Middleware always runs at the edge. Routes can opt into either. We'll return to this; for now, internalize that "the server" is not one place.

## The Server/Client Boundary: Your Central Design Axis

In the App Router, **everything is a Server Component by default.** A component becomes a Client Component only when it (or a module it imports) is marked `"use client"`. This single boundary determines:

- **What ships to the browser.** Server Components send zero JS for their own logic. Client Components ship their code plus their imports into the bundle.
- **What APIs you can use.** Server Components can `await` a database. Client Components can use `useState`, `onClick`, and browser APIs.
- **Where secrets are safe.** API keys, DB credentials, and business logic in a Server Component never reach the client.

Think of your component tree as having a *waterline*. Above it (toward the root) is server territory: data fetching, secrets, heavy logic. Below it (toward the leaves) is client territory: interactivity. Your job as an architect is to **draw that waterline as low as possible.**

```tsx
// app/dashboard/page.tsx — a Server Component (no directive needed)
import { getUser } from "@/lib/auth";
import { getMetrics } from "@/lib/metrics";
import { MetricChart } from "./_components/metric-chart"; // client leaf

export default async function DashboardPage() {
  // Runs on the server. Secrets and DB access stay here.
  const user = await getUser();
  const metrics = await getMetrics(user.orgId);

  return (
    <section>
      <h1>Welcome, {user.name}</h1>
      {/* Only the interactive chart is a Client Component. */}
      <MetricChart data={metrics} />
    </section>
  );
}
```

Here, all the data work and the user's name render on the server with no client JS. Only `MetricChart` — which needs hover tooltips and zoom — crosses into client territory, and it receives already-fetched, serialized data as props.

## RSC Payload vs HTML: Why You Get Both

A common misconception is that Server Components "just render HTML." They render two things, and understanding the difference unlocks a lot.

- **HTML** is the initial paint. It's what the browser shows before any JS runs, what crawlers read, and what makes the page feel instant.
- **The RSC payload** is a compact, serialized description of the React tree — a special format (not HTML, not JSON) that says "here is the component structure, here are the props, here is where the Client Components go." React on the client uses it to *reconcile*: it knows how to slot hydrated Client Components into the server-rendered output and how to apply later navigations without a full reload.

```text
HTML        → first paint, SEO, no-JS fallback
RSC payload → React's source of truth for reconciliation + client navigation
```

This is why a client-side navigation (e.g., clicking a `<Link>`) fetches an RSC payload, not a full HTML document: Next.js asks the server to re-render the relevant segments, ships the payload, and React patches the tree in place. No flash, no re-download of the shell. It's also why props passed across the server/client boundary **must be serializable** — they get encoded into this payload. A function (other than a Server Action) cannot survive the trip.

## Server-First, Push Interactivity to the Leaves

The governing philosophy of modern Next.js:

> Render as much as possible on the server. Make Client Components small, focused islands of interactivity at the edges of your tree.

Why this is the right default:

- **Less JS shipped** → faster loads, cheaper for users on slow networks and weak devices.
- **Data stays close to its source** → fetch in the component that needs it, on the server, no API round-trip from the browser.
- **Security by default** → logic and secrets never leave the server.

The classic mistake (carried over from the Pages Router and SPA habits) is slapping `"use client"` at the top of a page because *something* inside needs `onClick`. That turns the entire subtree into client code. Instead, isolate the interactive bit.

### Trade-offs: when server-first isn't the answer

Server-first is a default, not a dogma. Push logic to the client when:

- **Interaction is the whole point** and round-tripping to the server would feel sluggish (a drag-and-drop canvas, a rich text editor, a live drawing tool).
- **You need browser-only state** that has no server meaning (ephemeral UI like an open/closed accordion, cursor position).
- **Real-time/local feedback** matters more than data freshness (optimistic UI, client-side form validation before submit).

Conversely, resist the urge to make something a Client Component just because it's "below the fold" or "small." The cost isn't the size of one component — it's that the boundary is *contagious*: everything a Client Component imports comes along for the ride.

## Thinking at Scale: Latency, Cost, Caching

When you move from "does it render" to "does it render for everyone, cheaply, fast," three concerns dominate every decision.

- **Latency** is about *where* compute and data live relative to the user. A dynamic page that hits a database in `us-east-1` is slow for a user in Sydney. Caching, edge rendering, and static generation are all latency tools.
- **Cost** is about *how often* you do work. A page rendered fresh on every request is server CPU per visitor; the same page cached for 60 seconds is one render shared by thousands. Static is nearly free; dynamic is not.
- **Caching** is the lever that connects the two — and it's the hardest thing to get right. Next.js has multiple cache layers (request memoization, the Data Cache, the Full Route Cache, the Router Cache). Each answers a different question about *when* work can be reused. We devote a whole module to it.

The mental shift: stop asking "how do I render this?" and start asking **"how often does this actually change, and who needs it fresh?"** That question, answered per route, drives your rendering strategy, your cache config, and ultimately your bill.

## Pitfalls

- **Treating `"use client"` as a file label instead of a boundary.** Once you cross it, every imported module joins the client bundle. Place it deliberately at the smallest possible subtree.
- **Assuming Server Components render HTML only.** They produce an RSC payload that drives reconciliation and navigation; forgetting this leads to confusion about serialization and client navigations.
- **Passing non-serializable props across the boundary.** Functions, class instances, Dates-as-behavior, and Symbols won't cross. You'll get a runtime error or silent breakage.
- **Defaulting to dynamic rendering everywhere.** Reading `cookies()` or `headers()` in a layout can opt your whole route into dynamic rendering, quietly destroying static performance and inflating cost.
- **Ignoring the runtime question.** Reaching for a Node-only library forces the Node runtime; assuming edge "just works" with your ORM will fail at deploy time.

## Exercises

1. **Map a real page.** Take an existing page from a project you know. Draw its component tree and mark the server/client waterline. Identify any Client Component that could be split so the waterline drops lower.
2. **Trace a navigation.** For a client-side `<Link>` navigation in your app, describe what the browser requests, what the server returns, and how React applies it. Where does the RSC payload fit?
3. **Classify by change frequency.** List five routes in a hypothetical e-commerce app (home, product, cart, order history, marketing landing). For each, answer "how often does it change and who needs it fresh," and propose static / ISR / dynamic.
4. **Cost estimate.** For a route doing one DB query per render at 1M requests/day, estimate the difference between rendering fresh versus caching for 60 seconds. What's the order-of-magnitude reduction in renders?
5. **Runtime decision.** Pick a middleware use case (geo-redirect, auth gate) and a route use case (Stripe checkout). Argue which runtime each belongs in and why.

## Key Takeaways

- A Next.js request flows edge → server → render → stream → hydrate; each stage is a place to optimize.
- The server/client boundary decides what ships, what's secure, and what's interactive — design it as your primary axis.
- Server Components emit both HTML (first paint, SEO) and an RSC payload (reconciliation, client navigation); props across the boundary must be serializable.
- Default to server-first and push interactivity to small leaf Client Components; break the rule only when interaction or browser-only state demands it.
- At scale, replace "how do I render this?" with "how often does it change and who needs it fresh?" — that question drives strategy, caching, and cost.

---

[← Back to Course Index](../README.md) | [Next: Production-Grade Project Setup →](02-project-setup.md)
