# Bundle Optimization and Reducing Client JS

The App Router's default is to ship *nothing* to the client. Every kilobyte of JavaScript you do ship is a deliberate cost: it must be downloaded, parsed, compiled, and executed on the main thread before the page is interactive — and parse/compile is where mid-tier phones choke. At scale, bundle bloat is the dominant cause of poor INP and a quiet killer of LCP. This lesson is about auditing what you ship, understanding *why* it's there, and systematically removing it.

## What You'll Learn

- Why client JS costs more than its byte count suggests, and why it dwarfs other resources
- How to audit bundles with `@next/bundle-analyzer` and read the output
- When to code-split with `next/dynamic`, including when `ssr: false` is correct
- How to shrink `"use client"` boundaries so less of your tree ships to the browser
- The barrel-file tree-shaking trap and how `optimizePackageImports` fixes it
- A decision framework for heavy dependencies: lazy-load, move to server, or replace
- How to measure before/after so optimization claims are real

## Why Client JS Is the Enemy

A 300KB image and 300KB of JavaScript do not cost the same. The image decodes off the main thread and can be lazy. The JavaScript must be parsed and compiled on the main thread — roughly 1ms per KB on a median mobile device — and that work blocks every interaction. 300KB of JS is ~300ms of main-thread time before anything responds, which directly inflates INP and delays hydration (which can gate LCP).

In the App Router, Server Components ship zero JS. The bundle is built entirely from your `"use client"` components and their imports. So bundle optimization is mostly the discipline of keeping the client boundary small and the things inside it lean.

## Auditing With the Bundle Analyzer

You cannot fix what you can't see. Install and wire up the analyzer.

```bash
npm install --save-dev @next/bundle-analyzer
```

```ts
// next.config.ts
import type { NextConfig } from "next";
import withBundleAnalyzer from "@next/bundle-analyzer";

const config: NextConfig = {
  // ...your config
};

// Wraps the build to emit an interactive treemap when ANALYZE=true.
export default withBundleAnalyzer({ enabled: process.env.ANALYZE === "true" })(config);
```

```bash
# Build with analysis on — opens treemaps for client, edge, and node bundles
ANALYZE=true npm run build
```

Read the **client** treemap first — that's what users download. Look for: a single dependency taking a huge slice (a date library, an icon set, a charting lib), the same dependency duplicated across chunks, and large modules in the shared/framework chunk that load on every route. The build output table is your other quick signal:

```text
Route (app)                    Size     First Load JS
┌ ○ /                          1.2 kB        92 kB
├ ○ /dashboard                 4.8 kB       210 kB   ← investigate this
└ ○ /settings                  2.1 kB        96 kB
+ First Load JS shared by all                88 kB
```

"First Load JS" is the number that maps to perceived performance. Anything that pulls a route well above the shared baseline is your target.

## Code Splitting With `next/dynamic`

`next/dynamic` lazily loads a component so its code is fetched only when needed, splitting it out of the initial bundle.

```tsx
// app/dashboard/page.tsx
import dynamic from "next/dynamic";

// HeavyChart (and its charting dependency) is only downloaded when
// this route renders — and with a loading fallback, not blocking paint.
const HeavyChart = dynamic(() => import("./HeavyChart"), {
  loading: () => <ChartSkeleton />,
});

export default function Dashboard() {
  return (
    <main>
      <Summary /> {/* in the main bundle, paints fast */}
      <HeavyChart /> {/* code-split, loads after */}
    </main>
  );
}
```

### When `ssr: false` is correct (and when it's a mistake)

```tsx
// app/components/Map.tsx
import dynamic from "next/dynamic";

// A map widget that touches `window`/`document` and has no meaningful
// server render. ssr:false skips SSR entirely and loads it client-only.
// NOTE: ssr:false is only allowed in Client Components in Next 14+.
const Map = dynamic(() => import("./LeafletMap"), {
  ssr: false,
  loading: () => <div style={{ height: 400 }} />, // reserve space, avoid CLS
});
```

Use `ssr: false` when the component genuinely cannot or should not render on the server — it depends on browser-only globals, or it's below-the-fold and not worth the SSR cost. Do **not** use it for content that matters to LCP or SEO: skipping SSR means the user sees nothing until JS loads and runs, which is exactly the regression you're trying to avoid. Reserve a sized container so the deferred load doesn't shift layout.

## Shrinking `"use client"` Boundaries

The most impactful and most overlooked optimization. When you put `"use client"` at the top of a component, *that component and everything it imports* becomes client JS. Teams routinely mark a whole page client just to use one `onClick`, dragging the entire subtree into the bundle.

The fix is to push the boundary down to the smallest interactive leaf and keep the rest as Server Components, passing server-rendered content through as `children`.

```tsx
// BAD — app/products/page.tsx
"use client"; // makes the ENTIRE page + ProductList + everything a client bundle
import { useState } from "react";
import { ProductList } from "./ProductList"; // dragged to client unnecessarily

export default function Page() {
  const [open, setOpen] = useState(false);
  return (
    <>
      <button onClick={() => setOpen(!open)}>Filters</button>
      <ProductList /> {/* heavy, static — should NOT be client */}
    </>
  );
}
```

```tsx
// GOOD — app/products/page.tsx  (Server Component, ships no JS)
import { Filters } from "./Filters";       // small client leaf
import { ProductList } from "./ProductList"; // stays a Server Component

export default function Page() {
  return (
    <>
      <Filters /> {/* only THIS is client JS */}
      <ProductList /> {/* rendered on the server, zero JS */}
    </>
  );
}
```

```tsx
// app/products/Filters.tsx — the only client component
"use client";
import { useState } from "react";

export function Filters() {
  const [open, setOpen] = useState(false);
  return <button onClick={() => setOpen(!open)}>{open ? "Hide" : "Filters"}</button>;
}
```

The "children as a slot" pattern lets an interactive client wrapper enclose server-rendered content without that content becoming client JS:

```tsx
// A client component can RENDER server children passed as props.
// The server content is serialized as already-rendered output, not shipped as code.
"use client";
export function Collapsible({ children }: { children: React.ReactNode }) {
  const [open, setOpen] = useState(false);
  return (
    <>
      <button onClick={() => setOpen(!open)}>Toggle</button>
      {open && children} {/* `children` came from a Server Component */}
    </>
  );
}
```

## Tree-Shaking and the Barrel-File Trap

A barrel file (`index.ts` that re-exports everything) is convenient but defeats tree-shaking in many setups: importing one named export can pull the entire module graph into your bundle, because the bundler can't always prove the rest is unused.

```ts
// Importing from a barrel can drag in ALL of a library's icons/components
import { ChevronIcon } from "@acme/icons"; // might bundle 600 icons
```

Next ships `optimizePackageImports` to rewrite these imports so only the used modules are included — without you changing every import site.

```ts
// next.config.ts
const config: NextConfig = {
  experimental: {
    // Next transforms barrel imports from these packages into deep,
    // tree-shakeable imports automatically. lucide-react, @mui/*, and
    // many icon/UI libs are optimized by default; add your own here.
    optimizePackageImports: ["@acme/icons", "lodash-es", "date-fns"],
  },
};
```

If a package isn't covered, the fallback is to import the specific path directly: `import debounce from "lodash-es/debounce"` instead of `import { debounce } from "lodash-es"`.

## Heavy Dependencies: Lazy-Load, Move, or Replace

For any dependency dominating the treemap, run this decision flow:

| Option | Use when | Example |
| --- | --- | --- |
| **Move to server** | The work doesn't need the browser | Markdown parsing, syntax highlighting, date formatting, validation |
| **Replace** | A lighter equivalent exists | `moment` (~70KB) → `date-fns`/`dayjs` (~few KB) or native `Intl` |
| **Lazy-load** | It's genuinely interactive but not needed at first paint | Rich text editor, chart, modal, map |
| **Keep** | Small, used everywhere, on the critical path | Your design-system primitives |

```tsx
// MOVE TO SERVER — highlight code on the server; ship zero highlighter JS.
// app/post/[slug]/page.tsx (Server Component)
import { codeToHtml } from "shiki"; // runs at build/request time on the server

export default async function Post() {
  const html = await codeToHtml("const x = 1", { lang: "ts", theme: "github-dark" });
  return <div dangerouslySetInnerHTML={{ __html: html }} />;
}
```

```ts
// REPLACE — drop a 70KB date lib for native Intl, which ships 0 bytes.
const formatted = new Intl.DateTimeFormat("en-US", { dateStyle: "medium" }).format(date);
```

### Trade-offs

- **Code-splitting isn't free.** Each `next/dynamic` boundary is a network round trip and a potential loading flash. Split heavy, deferrable things — not tiny components, which just add request overhead and CLS risk.
- **`ssr: false` trades initial paint for a smaller initial bundle.** Right for below-the-fold/interactive widgets, wrong for primary content.
- **Server-side work shifts cost from client to compute.** Highlighting every code block per request can raise TTFB; cache it (build-time or ISR) so you don't trade INP for TTFB.
- **Over-splitting hurts.** Many tiny chunks can be slower than one well-sized chunk due to request overhead. Profile, don't guess.

## Measuring Before/After

Optimization without measurement is folklore. Capture the build output table before and after each change and diff the "First Load JS" for the affected routes.

```bash
# Capture a baseline, make the change, capture again, compare First Load JS.
npm run build | tee before.txt
# ...apply optimization...
npm run build | tee after.txt
diff before.txt after.txt
```

Pair this with the analyzer treemap (to confirm the dependency actually left the bundle) and a field/RUM check on INP a few days later (to confirm it moved the metric users feel). A smaller bundle that doesn't improve p75 INP means you optimized a route nobody waits on — redirect the effort.

## Pitfalls

- **`"use client"` at the top of a page.** The single biggest source of accidental bundle bloat. Push it to leaves.
- **Barrel imports without `optimizePackageImports`.** One icon import can bundle hundreds. Configure it or import deep paths.
- **`ssr: false` on primary content.** Users see blank space until JS executes; LCP and SEO both suffer.
- **Splitting trivially small components.** Adds request overhead and loading flashes for no real savings.
- **`import * as _ from "lodash"`.** Pulls the whole library. Use `lodash-es` deep imports or native equivalents.
- **Claiming a win from the treemap alone.** A smaller bundle that doesn't change p75 INP/LCP isn't a user-facing win. Verify in the field.

## Exercises

1. Enable `@next/bundle-analyzer`, run `ANALYZE=true npm run build`, and identify the three largest dependencies in your client bundle. Note which routes load each.
2. Find a page that's `"use client"` at the top level. Refactor it so only the interactive leaf is a client component and the rest are Server Components. Record First Load JS before and after.
3. Take the heaviest below-the-fold component and convert it to `next/dynamic` with a sized loading fallback. Confirm it left the initial bundle in the treemap.
4. Add `optimizePackageImports` for an icon or utility library you barrel-import, rebuild, and measure the bundle reduction.
5. Replace one heavy client dependency (a date or formatting lib) with a server-side or native (`Intl`) equivalent and document the byte savings.

## Key Takeaways

- Client JS costs main-thread parse/compile time, so it hurts INP far beyond its byte count — minimize it deliberately.
- The App Router ships zero JS by default; your bundle is your `"use client"` boundaries and what they import.
- Push `"use client"` to the smallest interactive leaf and pass server-rendered content through `children` — this is the highest-leverage fix.
- Use `next/dynamic` for heavy, deferrable, or browser-only code; reserve `ssr: false` for non-critical, browser-dependent widgets.
- Defeat the barrel-file trap with `optimizePackageImports` or deep imports, and always measure First Load JS before/after, confirming the win in field data.

---

[← Previous: Core Web Vitals and Performance Budgets](01-web-vitals.md) | [Back to Course Index](../README.md) | [Next: Asset Optimization at Scale →](03-asset-optimization.md)
