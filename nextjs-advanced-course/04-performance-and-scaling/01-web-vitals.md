# Core Web Vitals and Performance Budgets

Performance is not a vibe — it is a set of measured thresholds that decide whether users stay, convert, and whether Google ranks you. At scale, a 200ms regression in LCP shows up in your funnel as lost revenue, and a 0.05 jump in CLS shows up in rage clicks. This lesson treats Web Vitals as engineering metrics with budgets, owners, and CI gates, and ties each one to the concrete Next.js levers that move it.

## What You'll Learn

- What LCP, CLS, INP, TTFB, and FCP actually measure and how they interact
- The most common causes of each regression and the Next-specific fix for each
- How rendering strategy (SSR, SSG, ISR, streaming, client) changes your vitals profile
- How to set and enforce performance budgets in CI, not just measure after the fact
- The difference between lab data (Lighthouse) and field data (CrUX/RUM), and when each lies
- How to wire `useReportWebVitals` to a real-user monitoring pipeline
- How to prioritize work on the critical rendering path instead of guessing

## The Metrics, Precisely

Core Web Vitals are three field metrics plus supporting diagnostics. The "good" thresholds (75th percentile of real users) are the only numbers that matter for ranking.

```text
Metric  Measures                          Good     Needs work   Poor
LCP     Largest contentful paint          <2.5s    2.5-4.0s     >4.0s
INP     Interaction to Next Paint         <200ms   200-500ms    >500ms
CLS     Cumulative Layout Shift           <0.1     0.1-0.25     >0.25
---- supporting diagnostics ----
TTFB    Time to First Byte                <800ms   800-1800ms   >1800ms
FCP     First Contentful Paint            <1.8s    1.8-3.0s     >3.0s
```

These are not independent. TTFB is a component of FCP, which is a component of LCP. If your server takes 1.2s to respond, your LCP can never beat 1.2s no matter how perfect your front end is. Fix the chain from the back.

### LCP — what the user came to see

LCP marks when the largest above-the-fold element (usually a hero image, heading, or video poster) finishes painting. Common causes and fixes:

- **Slow server response (TTFB).** The page is fast but the server is slow. Fix with caching: convert a dynamic route to `revalidate`-based ISR, or stream the shell while the slow part suspends.
- **Render-blocking resources.** Fonts and CSS block paint. Use `next/font` (self-hosts and inlines) and avoid large blocking client bundles.
- **The LCP image is lazy-loaded.** This is the single most common Next mistake. Mark the hero with `priority`.

```tsx
// app/page.tsx
import Image from "next/image";

export default function Home() {
  return (
    <section>
      {/* priority preloads this image and disables lazy-loading.
          Use it ONLY for the LCP element — overusing it floods
          the network and hurts everything else. */}
      <Image
        src="/hero.jpg"
        alt="Product hero"
        width={1200}
        height={600}
        priority
        sizes="100vw"
      />
    </section>
  );
}
```

### CLS — the page moving under the user's thumb

CLS sums every unexpected layout shift. The usual culprits are images and ads without reserved space, web fonts swapping, and content injected above existing content.

```tsx
// app/components/Avatar.tsx
import Image from "next/image";

// Always give next/image explicit width/height (or fill + a sized parent).
// This reserves the box BEFORE the image loads, so nothing jumps.
export function Avatar({ src }: { src: string }) {
  return <Image src={src} alt="" width={48} height={48} />;
}
```

Font swap is the other big one. A web font loading after FCP reflows text and shifts everything below it. `next/font` solves this by self-hosting, preloading, and applying `size-adjust` so the fallback metrics match. Reserve space for any banner or skeleton you inject — never push content down after the fact.

### INP — does the UI respond when I touch it

INP replaced FID in 2024 and is harsher: it measures the latency of *all* interactions across the page lifecycle, reporting the worst (near-worst) one. The cause is almost always too much JavaScript on the main thread. A click handler that triggers a synchronous re-render of a 5,000-row table blocks the next paint.

```tsx
// app/components/Search.tsx
"use client";
import { useState, useTransition } from "react";

export function Search({ items }: { items: string[] }) {
  const [query, setQuery] = useState("");
  // useTransition marks the expensive filter as non-urgent.
  // The input stays responsive (low INP) while the list updates
  // in the background, instead of blocking paint on every keystroke.
  const [isPending, startTransition] = useTransition();
  const [results, setResults] = useState(items);

  function onChange(e: React.ChangeEvent<HTMLInputElement>) {
    setQuery(e.target.value);
    startTransition(() => {
      setResults(items.filter((i) => i.includes(e.target.value)));
    });
  }

  return (
    <>
      <input value={query} onChange={onChange} />
      {isPending ? <p>Updating…</p> : <List results={results} />}
    </>
  );
}

function List({ results }: { results: string[] }) {
  return <ul>{results.map((r) => <li key={r}>{r}</li>)}</ul>;
}
```

The structural fix for INP is shipping less client JS — covered in the next lesson.

## Rendering Strategy Drives Your Vitals Profile

Each rendering mode produces a different shape of vitals, and choosing the wrong one is the root cause behind many "we optimized everything and it's still slow" situations.

| Strategy | TTFB | LCP | INP risk | Best when |
| --- | --- | --- | --- | --- |
| Static (SSG) | Excellent (CDN) | Excellent | Low | Marketing, docs, content that rarely changes |
| ISR | Excellent (cached) | Excellent | Low | Catalogs, blogs — fresh-ish data, high traffic |
| SSR (dynamic) | At mercy of your data layer | Tied to TTFB | Low–med | Per-request personalization that can't be cached |
| Streaming SSR | Fast shell, slow parts suspend | Good if LCP is in the shell | Med | Pages with one slow data dependency |
| CSR / client-heavy | Fast HTML, slow paint | Poor | High | Avoid for first paint; fine for app dashboards behind auth |

The lever most teams miss: **streaming with Suspense**. If your LCP element does not depend on the slow query, stream the shell (with the hero) immediately and suspend only the slow region. Your LCP detaches from your worst data dependency.

```tsx
// app/dashboard/page.tsx
import { Suspense } from "react";

export default function Dashboard() {
  return (
    <>
      <Hero /> {/* fast, static — this is the LCP, paints immediately */}
      <Suspense fallback={<StatsSkeleton />}>
        {/* slow DB query — streamed in after, does not block LCP */}
        <SlowStats />
      </Suspense>
    </>
  );
}
```

## Setting Performance Budgets

A budget is a number you fail CI against. Without one, performance only ever degrades. Set budgets per route class (landing pages are stricter than the internal admin).

```jsonc
// lighthouse-budget.json — enforce per-route resource and timing limits
[
  {
    "path": "/*",
    "timings": [
      { "metric": "largest-contentful-paint", "budget": 2500 },
      { "metric": "interactive", "budget": 3500 }
    ],
    "resourceSizes": [
      { "resourceType": "script", "budget": 170 },
      { "resourceType": "total", "budget": 500 }
    ]
  }
]
```

```bash
# Run Lighthouse CI against the budget in your pipeline
npx lhci autorun --collect.url=https://staging.example.com \
  --assert.budgetsFile=./lighthouse-budget.json
```

The 170KB script budget is deliberate: it roughly maps to the JS a mid-tier mobile phone on slow 4G can parse without blowing the interactivity budget. Treat it as a ratchet — never let it climb.

## Measuring: Lab vs Field

- **Lighthouse / PageSpeed (lab):** one synthetic run, simulated throttling. Great for reproducible CI gates and debugging a specific regression. It does **not** capture INP well (no real interactions) and does not reflect your actual user devices.
- **CrUX / field data:** the 28-day rolling 75th percentile of real Chrome users. This is what Google ranks on. It lags by weeks and only covers Chrome with enough traffic.
- **RUM (your own):** real-user monitoring you collect yourself. Real-time, segmentable by route/device/geo, captures INP properly.

Wire RUM with `useReportWebVitals`:

```tsx
// app/_components/web-vitals.tsx
"use client";
import { useReportWebVitals } from "next/web-vitals";

export function WebVitals() {
  useReportWebVitals((metric) => {
    // Use sendBeacon so the request survives page unload.
    const body = JSON.stringify({
      name: metric.name,      // LCP | INP | CLS | TTFB | FCP
      value: metric.value,
      id: metric.id,
      rating: metric.rating,  // "good" | "needs-improvement" | "poor"
      path: window.location.pathname,
    });
    navigator.sendBeacon("/api/vitals", body);
  });
  return null;
}
```

```tsx
// app/layout.tsx
import { WebVitals } from "./_components/web-vitals";

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body>
        <WebVitals />
        {children}
      </body>
    </html>
  );
}
```

Alert on the 75th percentile, not the average — averages hide the long tail that field data actually scores.

### Trade-offs: where to spend your optimization budget

You cannot fix everything at once. Prioritize by the critical rendering path:

1. **TTFB first if it's >800ms.** Nothing downstream can be fast if the byte stream is slow. Cache or stream.
2. **LCP element second.** Find it in Lighthouse's "Largest Contentful Paint element" diagnostic. Preload it, right-size it, stop lazy-loading it.
3. **CLS third.** Usually a handful of unsized images and one font — cheap, high-impact fixes.
4. **INP last but ongoing.** It's a JS-budget problem that the bundle and asset lessons address structurally.

Do not micro-optimize a metric that's already green. Moving LCP from 2.3s to 2.1s scores identically; moving a poor route from 4.1s to 2.4s changes your ranking and your funnel.

## Pitfalls

- **Marking everything `priority`.** It only helps the true LCP element; on every other image it steals bandwidth and *raises* LCP.
- **Trusting Lighthouse for INP.** Lab tools barely test interactions. INP regressions only show up in field/RUM data — monitor it there.
- **Optimizing the average, ignoring p75.** Google scores the 75th percentile. A great median with a bad tail still fails Core Web Vitals.
- **Client-rendering the hero.** A `"use client"` component that fetches its own data on mount delays LCP behind hydration plus a round trip. Render it on the server.
- **Reserving no space for async content.** Ads, embeds, and skeletons that resize after load are the top CLS source. Fix the dimensions up front.
- **Setting a budget but not gating CI on it.** A budget you only look at in dashboards never holds the line.

## Exercises

1. Open your highest-traffic route in PageSpeed Insights. Identify the LCP element from the diagnostics and confirm whether it has `priority`. If not, add it and re-measure.
2. Add the `useReportWebVitals` collector above, build a `/api/vitals` route that logs to your DB or analytics, and chart p75 INP by route after a day of traffic.
3. Write a `lighthouse-budget.json` for your two most important route classes and add `lhci autorun` as a required CI check that fails the build on a budget breach.
4. Take one client component on a first-paint route and move its data fetch to the server (or wrap it in `Suspense` and stream it). Measure LCP before and after.
5. Audit one page for CLS: list every image without explicit dimensions and every async-injected block, then reserve space for each and verify CLS drops below 0.1 in the field tool.

## Key Takeaways

- LCP, INP, and CLS are the ranking-relevant trio; TTFB and FCP are diagnostics on the same critical path.
- Fix the chain back-to-front: TTFB gates everything, then the LCP element, then layout stability, then interactivity.
- Rendering strategy is your biggest vitals lever — streaming detaches LCP from slow data; client rendering the first paint is a trap.
- Budgets enforced in CI are the only thing that stops gradual regression; a ~170KB script budget is a sane default.
- Lab tools (Lighthouse) are for reproducible debugging; field/RUM data (p75) is what you're actually scored on — especially for INP.

---

[← Previous: Error Handling and Resilience Patterns](../03-architecture/05-error-handling-resilience.md) | [Back to Course Index](../README.md) | [Next: Bundle Optimization →](02-bundle-optimization.md)
