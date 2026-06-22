# Migration and Incremental Adoption

Big-bang rewrites are how teams lose a quarter and ship the same product with new bugs. The far better move, and the one Next.js explicitly supports, is incremental migration: the Pages Router and App Router run side by side in the same application, so you move route by route, ship continuously, and de-risk as you go. The hard part isn't the mechanics — it's sequencing, risk management, and knowing when the migration isn't worth doing at all. This lesson is about doing it like someone responsible for the outcome.

## What You'll Learn

- How `pages/` and `app/` coexist in one app and the rules that govern routing precedence
- A sequencing strategy for migrating Pages Router → App Router incrementally
- Replacing `getServerSideProps`/`getStaticProps` with Server Component data fetching
- Migrating client-side data fetching (SWR/React Query) to RSC, and when to keep it
- Routing and data-fetching gotchas that bite during migration
- Running a low-risk incremental rollout with measurable checkpoints
- When NOT to migrate — recognizing negative ROI

## Coexistence: the foundation

Both routers can live in the same project. This is the single fact that makes incremental migration possible.

```text
my-app/
  app/
    dashboard/
      page.tsx        // new App Router route
  pages/
    settings.tsx      // legacy Pages Router route
    api/
      legacy.ts       // Pages API routes still work
```

The rules you must internalize:

- **`app/` wins on conflict.** If both `app/dashboard/page.tsx` and `pages/dashboard.tsx` exist, the App Router route serves. Don't leave both; it's a footgun.
- **They don't share layouts or providers.** A context provider in `pages/_app.tsx` does not wrap `app/` routes, and `app/layout.tsx` does not wrap `pages/`. Shared client state across the boundary requires duplication or a different mechanism.
- **Navigating between them is a full page load, not a client transition.** A `<Link>` from an `app/` route to a `pages/` route reloads the document. Plan your migration so high-traffic navigation paths don't repeatedly cross the boundary.

The practical implication: migrate in **cohesive sections**, not scattered leaf pages. Move an entire feature area (all of `/dashboard/*`) at once so users navigating within it stay on one router.

## Sequencing the migration

A sane order, roughly lowest-risk to highest:

1. **Upgrade Next.js and React first**, on the Pages Router, with no behavior change. Get to a current version that supports `app/` before you touch architecture.
2. **Create `app/layout.tsx` and a single, low-traffic route** (a settings page, an about page). Validate the build, deployment, and your CI on a real App Router page.
3. **Migrate shared infrastructure**: root layout, fonts, metadata, error boundaries, global styles. This is reusable groundwork for everything after.
4. **Migrate a cohesive feature area** end to end, including its data fetching and any nested layouts.
5. **Migrate API routes** (`pages/api/*`) to Route Handlers (`app/.../route.ts`) as you touch the features that use them.
6. **Repeat by feature area**, highest-value or highest-pain first, until `pages/` is empty.
7. **Delete `pages/`** and remove now-dead compatibility code.

Each step ships to production independently. There is no long-lived migration branch.

## Replacing `getServerSideProps` / `getStaticProps`

This is the conceptual core. The old data-fetching functions are gone; in the App Router you fetch directly inside Server Components and choose rendering mode declaratively.

```tsx
// BEFORE — pages/products/[id].tsx
export async function getServerSideProps({ params }) {
  const product = await getProduct(params.id);
  return { props: { product } };
}
export default function Product({ product }) {
  return <h1>{product.name}</h1>;
}
```

```tsx
// AFTER — app/products/[id]/page.tsx
export default async function Product({
  params,
}: {
  params: Promise<{ id: string }>;
}) {
  const { id } = await params;
  const product = await getProduct(id); // fetch right where you render
  return <h1>{product.name}</h1>;
}
```

The mapping you should keep in your head:

```text
getServerSideProps         → async Server Component (dynamic by default if
                             it reads cookies/headers/searchParams, or set
                             export const dynamic = "force-dynamic")
getStaticProps             → async Server Component + default static rendering
getStaticProps + revalidate→ export const revalidate = N  (ISR)
getStaticPaths             → generateStaticParams()
getInitialProps            → no equivalent; refactor to Server Component
```

A key mental shift: in Pages, data fetching was a *separate* function feeding props into a single component. In App Router, **any Server Component can fetch its own data**, so you co-locate fetching with the component that needs it and let React stream them in parallel. No more lifting every query into one top-level function.

```tsx
// app/products/[id]/page.tsx — parallel, co-located fetching
import { Suspense } from "react";

export default async function Product({ params }: { params: Promise<{ id: string }> }) {
  const { id } = await params;
  // These two components fetch independently and stream in parallel.
  return (
    <>
      <ProductHeader id={id} />
      <Suspense fallback={<ReviewsSkeleton />}>
        <Reviews id={id} /> {/* slow query no longer blocks the header */}
      </Suspense>
    </>
  );
}
```

## Migrating client data fetching to RSC

Many Pages apps fetch on the client with SWR or React Query in `useEffect`. A lot of that becomes a server fetch with no library at all.

```tsx
// BEFORE — client fetch in a Pages component
"use client";
import useSWR from "swr";
function Profile() {
  const { data } = useSWR("/api/me", fetcher); // loading flash, waterfall
  if (!data) return <Spinner />;
  return <h1>{data.name}</h1>;
}
```

```tsx
// AFTER — Server Component, no client JS, no spinner
async function Profile() {
  const me = await getMe(); // runs on the server, data in initial HTML
  return <h1>{me.name}</h1>;
}
```

But **don't migrate everything to RSC reflexively.** Keep client data fetching when you genuinely need:

- Polling, real-time updates, or websockets-driven state.
- Optimistic UI and cache mutation after user actions (React Query/SWR shine here).
- Data that depends on client-only state (scroll position, viewport).

The senior call: server-fetch the initial render for SEO and speed, and keep a client library only for the interactive, mutating, or live parts. Hybrid is normal and correct.

## Routing and data-fetching gotchas

- **`useRouter` moved.** Import from `next/navigation` in App Router, not `next/router`. The API differs: `useRouter()` no longer exposes `query`/`pathname`; use `useParams`, `usePathname`, `useSearchParams` instead.
- **`params`/`searchParams` are async** in Next 15. Awaiting them is required; this trips up copied-over code.
- **No `_app.tsx`/`_document.tsx`.** Their roles move to `app/layout.tsx` (and `<html>`/`<body>` live in the root layout now).
- **Reading `searchParams` makes a page dynamic.** A statically-rendered Pages page that read query strings may become dynamic after migration; verify your rendering mode didn't silently change.
- **Data Cache semantics changed across versions.** In Next 15 `fetch` is no longer cached by default. If you relied on implicit caching, set it explicitly with `cache`/`next.revalidate`.
- **Providers must move.** Context providers from `_app.tsx` need re-wrapping in `app/layout.tsx` (often as a `"use client"` providers component).

## Risk management and incremental rollout

Treat each migrated route as a release with a blast radius.

- **Ship behind the URL, not a branch.** Because routers coexist, a migrated route can go to production the day it's done. No multi-month integration branch to merge later.
- **Use the rollout tools you already have.** Canary/percentage rollouts and feature flags can route a fraction of traffic to the migrated version. If your platform supports it, A/B the old `pages/` route against the new `app/` one and compare Core Web Vitals and error rates.
- **Establish checkpoints per route.** Define "done" as: feature parity verified, Web Vitals equal-or-better, error rate flat, and the legacy route deleted. A migration that leaves both versions live is not done; it's debt.
- **Watch the bundle and the boundaries.** RSC migration should *reduce* client JS. If a migrated page's bundle grew, you likely over-applied `"use client"`. Track bundle size as a migration KPI.
- **Keep a rollback path.** Until the old route is deleted, you can revert by restoring routing precedence. Don't delete the legacy code until the new route has soaked in production.

### Trade-offs: when to use what

- **Incremental migration** — The default for any non-trivial app. Ships continuously, de-risks per route, keeps the team delivering features alongside.
- **Big-bang rewrite** — Almost never. Justifiable only for a small app (a handful of pages) where a clean cut is faster than maintaining two routers.
- **Server-fetch everything vs hybrid** — Server-fetch initial render; keep client libraries for live/optimistic/interactive data. Pure-RSC dogmatism creates worse UX for genuinely interactive features.
- **Migrate API routes now vs later** — Migrate them lazily, alongside the features that consume them. There's no prize for an empty `pages/api` if nothing forced it.

## When NOT to migrate

Seniority is partly knowing when the answer is "don't." Skip or defer migration when:

- **The app is stable, low-traffic, and rarely changed.** The performance and DX upside doesn't justify regression risk on something that isn't actively developed.
- **You're mid-flight on a deadline-critical feature.** Don't blend a migration into a high-stakes delivery; you can't cleanly attribute regressions.
- **The Pages Router is meeting your needs and the team lacks RSC fluency.** Migrate *after* the team has learned the model on something small, not as the learning vehicle for a critical path.
- **The cost-benefit is negative.** A 20-page internal admin tool with no SEO needs and no perf complaints gains little. Spend the quarter elsewhere.

The Pages Router is supported and not deprecated. "Newer" is not a business case. Migrate where it buys you measurable performance, simpler data flow, or a foundation for features you actually plan to build.

## Pitfalls

- **Scattering migrated leaf pages across the app.** Cross-router navigation forces full page reloads. Migrate cohesive feature areas instead.
- **Leaving both `pages/` and `app/` versions of a route live.** `app/` silently wins; the dead `pages/` version is confusing debt. Delete on completion.
- **Forgetting providers don't cross the boundary.** Context set up in `_app.tsx` won't wrap `app/` routes; re-wrap in the root layout.
- **Copying `useRouter` from `next/router`.** Wrong import and wrong API in App Router. Use `next/navigation` and the dedicated hooks.
- **Assuming `fetch` is still cached by default.** In Next 15 it isn't; set caching explicitly or face surprise origin load.
- **Over-applying `"use client"` during migration.** Bloats the bundle and forfeits the RSC win. Push `"use client"` to the leaves.
- **Treating the migration as a long-lived branch.** Defeats the entire point. Ship per route to production.

## Exercises

1. Take a Pages Router app and migrate a single low-traffic page to `app/`, setting up `app/layout.tsx` and re-wrapping any providers. Confirm both routers build and deploy together.
2. Convert a `getServerSideProps` page and a `getStaticProps` + `getStaticPaths` page to Server Components with `generateStaticParams`, preserving the rendering mode (dynamic vs static).
3. Replace a `useSWR` client fetch with a Server Component fetch for the initial render, then identify one piece of that page that should *stay* client-fetched and justify it.
4. Migrate a `pages/api` route to a Route Handler and update its consumer, verifying behavior parity.
5. Write a one-page migration plan for a real or hypothetical app: sequencing by feature area, per-route "done" criteria, rollout/flagging approach, and an explicit list of routes you'd *not* migrate and why.

## Key Takeaways

- `pages/` and `app/` coexist, which is what makes safe incremental migration possible — `app/` wins on conflict and the boundary is a full reload.
- Migrate cohesive feature areas, not scattered leaf pages, and ship each to production independently rather than on a long-lived branch.
- `getServerSideProps`/`getStaticProps`/`getStaticPaths` map to async Server Components, `revalidate`, and `generateStaticParams`; data fetching co-locates with components.
- Move initial-render fetching to RSC, but keep client libraries for live, optimistic, and interactive data — hybrid is correct.
- Manage migration like a release program: per-route checkpoints on parity, Web Vitals, error rate, and bundle size, with a rollback path.
- Know when not to migrate; "newer" is not a business case, and the Pages Router remains supported.

---

[← Previous: Caching, CDN, and Multi-Region Delivery](02-cdn-multiregion.md) | [Back to Course Index](../README.md) | [Next: Design Systems and Component Libraries →](04-design-systems.md)
