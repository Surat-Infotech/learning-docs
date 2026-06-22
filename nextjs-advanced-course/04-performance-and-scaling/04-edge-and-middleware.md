# Edge Runtime, Middleware, and Personalization

Middleware runs before every matched request, on every region, on the hot path — which makes it the most powerful and most dangerous performance lever in Next.js. A few lines there can gate auth globally with near-zero latency, or they can add 50ms to every single request and silently break your CDN caching. This lesson covers the Edge vs Node.js runtime decision, what middleware is genuinely good for, the things you must never do in it, and how to personalize at the edge without throwing away the caching that makes a fast site fast.

## What You'll Learn

- The capability and limit differences between the Edge and Node.js runtimes
- Cold-start and latency characteristics of each, and how to choose
- The legitimate use cases for `middleware.ts`: auth gating, redirects, A/B tests, geo, rewrites
- What NOT to do in middleware and why it's a per-request tax
- How to personalize at the edge without destroying cacheability
- Reading and writing cookies/headers in middleware correctly
- How middleware interacts with the Next.js cache and CDN

## Edge vs Node.js Runtime

Next.js can run server code in two runtimes. They are not interchangeable.

| | Edge runtime | Node.js runtime |
| --- | --- | --- |
| API surface | Web standards (`fetch`, `Request`, `Response`, Web Crypto) | Full Node (`fs`, `net`, `crypto`, native addons) |
| Cold start | Near-zero (lightweight isolate) | Higher (full Node process) |
| Geographic model | Runs at the edge, close to the user | Runs in fixed region(s) |
| Bundle/size limits | Tight (a few MB), no most npm native deps | Generous |
| TCP database connections | No raw TCP — HTTP/serverless drivers only | Yes (with pooling) |
| Best for | Lightweight, latency-sensitive, globally distributed logic | Heavy compute, native deps, raw DB connections |

The Edge runtime is a V8 isolate, not Node. It boots in milliseconds and runs physically close to the user, so TTFB is excellent — but you give up the Node API, raw TCP sockets (so most traditional DB drivers don't work), and you have a tight size budget. Choose Edge for thin, fast, geo-distributed work: auth checks, header rewriting, redirects, feature flags. Choose Node for routes that need native modules, large dependencies, or a pooled Postgres/MySQL connection.

```ts
// app/api/heavy/route.ts
// Opt a route into a runtime explicitly. Default is "nodejs".
export const runtime = "nodejs"; // needs `fs`, a Node DB driver, or big deps

export async function GET() {
  // ... heavy work that can't run in an isolate
  return Response.json({ ok: true });
}
```

`middleware.ts` runs on the Edge runtime by default in most deployments — which is exactly why it must stay thin.

## What `middleware.ts` Is For

Middleware intercepts requests before they hit a route. The `matcher` config restricts which paths it runs on — scope it tightly, because an unscoped middleware runs on every asset request too.

```ts
// middleware.ts
import { NextRequest, NextResponse } from "next/server";

export function middleware(req: NextRequest) {
  const { pathname } = req.nextUrl;

  // 1) AUTH GATING — redirect unauthenticated users away from protected paths.
  // Read the cookie only; do NOT verify a session against a DB here (see pitfalls).
  if (pathname.startsWith("/dashboard")) {
    const token = req.cookies.get("session")?.value;
    if (!token) {
      const url = req.nextUrl.clone();
      url.pathname = "/login";
      url.searchParams.set("from", pathname);
      return NextResponse.redirect(url);
    }
  }

  // 2) GEO ROUTING — rewrite by country (provided by the platform, no lookup cost).
  const country = req.headers.get("x-vercel-ip-country") ?? "US";
  if (pathname === "/" && country === "DE") {
    return NextResponse.rewrite(new URL("/de", req.url));
  }

  return NextResponse.next();
}

// Scope tightly: skip static assets and image optimizer requests entirely.
export const config = {
  matcher: ["/((?!_next/static|_next/image|favicon.ico).*)"],
};
```

The legitimate use cases all share a property: they're **fast, stateless decisions based on the request itself** — the URL, cookies, or platform-provided headers (geo, IP). Redirects, rewrites, header injection, A/B bucketing, and lightweight auth gating fit. The moment a decision needs a database, a slow API, or heavy computation, it does not belong in middleware.

### A/B testing and feature flags

A/B tests are a perfect middleware job: bucket the user once via a cookie, rewrite to the variant, and let the rest of the stack render normally.

```ts
// middleware.ts (excerpt)
export function middleware(req: NextRequest) {
  const res = NextResponse.next();

  let bucket = req.cookies.get("ab-bucket")?.value;
  if (!bucket) {
    bucket = Math.random() < 0.5 ? "a" : "b";
    // Persist the bucket so the user stays consistent across visits.
    res.cookies.set("ab-bucket", bucket, { path: "/", maxAge: 60 * 60 * 24 * 30 });
  }

  if (req.nextUrl.pathname === "/pricing") {
    return NextResponse.rewrite(new URL(`/pricing-${bucket}`, req.url));
  }
  return res;
}
```

## The Performance Cost: What NOT to Do

Middleware runs on **every matched request**, and it runs **before** the response can be served from cache. Whatever you do there is added to the latency of every page load, and it cannot be cached away. That makes the following anti-patterns expensive:

- **Don't query a database or call a slow API.** Edge middleware can't hold pooled TCP connections, and even an HTTP call adds round-trip latency to every request. A 40ms session-lookup call becomes a 40ms tax on the whole site.
- **Don't verify/decrypt heavy JWTs or do crypto-heavy work.** Check for *presence* of a session cookie in middleware for a redirect; do the cryptographic *verification* inside the route/server component where you'd query the user anyway. (Auth depth is covered in the next module.)
- **Don't run large dependencies.** The Edge bundle budget is tight and big imports inflate cold starts and per-request parse.
- **Don't fan out to multiple awaits.** Each `await` serializes onto the request's critical path.

```ts
// ANTI-PATTERN — a DB/auth-service call in middleware taxes EVERY request.
export async function middleware(req: NextRequest) {
  const user = await db.users.findSession(req.cookies.get("session")?.value); // ❌ slow, no pooling
  if (!user) return NextResponse.redirect(new URL("/login", req.url));
  return NextResponse.next();
}
```

The fix: middleware does the cheap gate (cookie present? redirect if not), and the protected route does the authoritative check.

## Personalization Without Killing Caching

This is the central tension. A fully static page is served from the CDN in single-digit milliseconds. The moment a response varies per user, it becomes uncacheable — and you've traded your best performance asset for personalization. The goal is to personalize *narrowly* while keeping the bulk of the page cached.

Three patterns, from cheapest to most invasive:

1. **Rewrite to a static variant (best).** Bucket-based or geo-based content where the number of variants is small. Each variant `/de`, `/pricing-a` is itself fully static and CDN-cacheable. Middleware only routes; it doesn't personalize the bytes.

2. **Cached shell + client-side personalization.** Serve a cached page and hydrate the personal bits (name, cart count) on the client from a small per-user API. The expensive layout stays cached; only a tiny dynamic fetch is per-user.

3. **Per-user dynamic render (most expensive).** `cookies()`/`headers()` in a Server Component makes the route dynamic — no full-page caching. Reserve this for pages that are inherently per-user (account, dashboard) where caching the whole page was never possible anyway.

```tsx
// PATTERN 2 — cached shell, personalize a slice on the client.
// app/page.tsx (static / ISR — fully cacheable)
import { Greeting } from "./Greeting";

export default function Home() {
  return (
    <main>
      <Hero /> {/* cached, identical for everyone */}
      <Greeting /> {/* client component, fetches /api/me for the name only */}
    </main>
  );
}
```

The key insight: keep personalization *small and late*. A cached page with a 2KB client-side personal slice is dramatically faster than a fully dynamic per-user render of the same page.

## Cookies and Headers in Middleware

Middleware can read request cookies/headers and set response cookies/headers. To pass derived data *forward* to your route, set a request header on the forwarded request.

```ts
// middleware.ts — read, derive, and forward via a request header.
export function middleware(req: NextRequest) {
  const country = req.headers.get("x-vercel-ip-country") ?? "US";

  // Clone request headers and inject a derived value the route can read.
  const requestHeaders = new Headers(req.headers);
  requestHeaders.set("x-user-country", country);

  const res = NextResponse.next({ request: { headers: requestHeaders } });

  // Set a response cookie/header (visible to the browser).
  res.headers.set("x-edge-handled", "1");
  return res;
}
```

```tsx
// app/page.tsx — read the forwarded header in a Server Component.
import { headers } from "next/headers";

export default async function Page() {
  const country = (await headers()).get("x-user-country");
  return <p>Serving content for {country}</p>;
}
```

Be aware: reading `cookies()` or `headers()` in a Server Component opts that route into dynamic rendering. That's fine for genuinely personal pages, but don't reach for them on a page you wanted cached.

## Middleware and the Cache

The ordering matters: **middleware runs before the response is resolved, including before a cached/static response is served.** Implications:

- Middleware itself is never cached — its cost is paid on cache hits *and* misses.
- A middleware `rewrite` changes which cache entry is used; rewriting `/` to `/de` serves (and caches) the `/de` static asset. This is how you get personalized routing with full caching.
- Setting `Cache-Control` or `Vary` from middleware affects how downstream CDNs cache. A careless `Vary: Cookie` will fragment your cache per user and effectively disable it.

### Trade-offs

- **Edge middleware vs route-level checks:** edge gives global low latency but no DB/heavy deps; route-level checks have full power but run after the request reaches your region. Gate cheaply at the edge, decide authoritatively in the route.
- **Rewrite-to-variant vs dynamic render:** rewrites preserve caching but only scale to a handful of variants; dynamic rendering handles unlimited personalization but forfeits full-page caching. Pick by variant cardinality.
- **More in middleware vs less:** every line is a global per-request tax. The default posture is "as little as possible, as fast as possible."

## Pitfalls

- **DB or auth-service calls in middleware.** No connection pooling at the edge and a latency tax on every request. Gate on cookie presence; verify in the route.
- **An unscoped `matcher`.** Middleware runs on static assets and image requests too, multiplying its cost. Exclude `_next/static`, `_next/image`, etc.
- **`Vary: Cookie` (or per-user headers) on cacheable responses.** Shatters the CDN cache into per-user entries — effectively no caching.
- **Reaching for `cookies()`/`headers()` on a page you wanted static.** It silently makes the route dynamic. Personalize a client slice instead.
- **Heavy crypto/JWT verification in middleware.** Multiplied across all traffic it's a real CPU and latency cost; do it where you already touch the user.
- **Assuming middleware is cached.** It isn't — its cost applies even when the page is a CDN hit.

## Exercises

1. Write a `middleware.ts` that gates `/dashboard` on the presence of a session cookie and redirects to `/login?from=...`. Confirm the matcher excludes static assets.
2. Implement a 50/50 A/B test that buckets users via a persistent cookie and rewrites `/pricing` to `/pricing-a` or `/pricing-b`, both static. Verify both variants are CDN-cacheable.
3. Take a page that's currently dynamic for personalization and refactor it to a cached shell plus a client-side personal slice. Compare TTFB before and after.
4. Add geo routing: read the platform country header, rewrite the homepage for one country, and forward the country to the route via a request header read with `headers()`.
5. Measure middleware overhead: add a timing log, deploy, and record the added latency on a cached route. Then move any expensive work out and measure again.

## Key Takeaways

- The Edge runtime is a fast, globally distributed isolate without Node APIs or raw DB connections; the Node runtime is full-power but region-bound — match the runtime to the work.
- Middleware is for fast, stateless request decisions: redirects, rewrites, geo, A/B bucketing, and cookie-presence auth gating.
- Never put DB calls, heavy crypto, or big dependencies in middleware — it's an uncacheable per-request tax on all traffic.
- Personalize narrowly: rewrite to static variants when you can, personalize a small client slice when you can't, and accept dynamic rendering only for inherently per-user pages.
- Middleware runs before caching; rewrites pick the cache entry, while `Vary: Cookie` and per-user responses can destroy cacheability.

---

[← Previous: Asset Optimization at Scale](03-asset-optimization.md) | [Back to Course Index](../README.md) | [Next: Scaling and Infrastructure →](05-scaling-infrastructure.md)
