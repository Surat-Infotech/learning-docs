# Scaling and Infrastructure Considerations

When traffic goes from thousands to millions of requests, the bottlenecks move from your code to your infrastructure: where the ISR cache lives, whether your database survives a serverless connection storm, how far data sits from compute, and which of compute, cache, or bandwidth is quietly eating your budget. This lesson is about the architecture decisions that determine whether your Next.js app scales smoothly or falls over under load — and the self-hosting-vs-Vercel trade-offs that shape all of them.

## What You'll Learn

- How ISR behaves at scale and why the cache needs shared storage
- Implementing a custom `cacheHandler` for self-hosted, multi-instance deployments
- Why serverless creates connection storms and how to fix them with pooling/serverless drivers
- Multi-region and data-locality strategies, and the latency trap they hide
- CDN strategy for static, ISR, and dynamic responses
- Concurrency, memory limits, and horizontal scaling considerations
- A cost framework across compute, cache, and bandwidth — and self-hosting vs Vercel

## ISR at Scale and Shared Cache Storage

ISR (`revalidate`) serves a cached page and regenerates it in the background after the interval. On a single server this is simple. The problem appears the moment you run **multiple instances**: by default each instance keeps its own on-disk cache, so a revalidation on instance A is invisible to instances B and C. Users get inconsistent content, and you regenerate the same page N times instead of once — multiplying load on your data layer.

The fix is a **shared cache handler** backed by external storage (Redis, S3, etc.) that all instances read and write. Vercel provides this automatically; self-hosting requires you to configure it.

```ts
// next.config.ts
const config: NextConfig = {
  // Point Next's cache (ISR/data cache) at a shared store instead of local disk.
  cacheHandler: require.resolve("./cache-handler.js"),
  // Disable the default in-memory cache so memory doesn't balloon per instance.
  cacheMaxMemorySize: 0,
};
```

```js
// cache-handler.js — minimal shared handler backed by Redis (sketch).
// Next calls get/set with tags so revalidateTag/revalidatePath work cluster-wide.
const { createClient } = require("redis");

module.exports = class CacheHandler {
  constructor() {
    this.client = createClient({ url: process.env.REDIS_URL });
    this.client.connect();
  }

  async get(key) {
    const data = await this.client.get(key);
    return data ? JSON.parse(data) : null;
  }

  async set(key, data, ctx) {
    // Persist value + tags so they're shared across all instances.
    await this.client.set(key, JSON.stringify({ value: data, tags: ctx.tags }));
    for (const tag of ctx.tags || []) {
      await this.client.sAdd(`tag:${tag}`, key);
    }
  }

  async revalidateTag(tags) {
    // Invalidate every key associated with the tag, cluster-wide.
    const list = Array.isArray(tags) ? tags : [tags];
    for (const tag of list) {
      const keys = await this.client.sMembers(`tag:${tag}`);
      if (keys.length) await this.client.del(keys);
      await this.client.del(`tag:${tag}`);
    }
  }
};
```

With a shared handler, `revalidateTag("products")` invalidates the entry once and every instance serves the fresh copy on the next request — and you regenerate once, not per instance.

## Serverless Cold Starts and Database Connections

Serverless functions scale by spinning up many short-lived instances. Two problems follow.

**Cold starts:** a new instance must boot before it serves. Keep functions small (the bundle work from earlier lessons pays off here), avoid heavy top-level initialization, and prefer the Edge runtime for latency-critical paths since isolates boot in milliseconds.

**Connection storms:** this is the one that takes down databases. A traditional Postgres/MySQL connection is a long-lived TCP socket. Under a traffic spike, serverless spins up hundreds of concurrent function instances, each opening its own connection — and a Postgres instance configured for ~100 connections falls over at 500.

Two fixes, often used together:

```ts
// 1) A serverless driver that talks to the DB over HTTP/WebSocket,
//    so there's no raw TCP connection to exhaust. Works at the edge too.
// lib/db.ts
import { neon } from "@neondatabase/serverless";

const sql = neon(process.env.DATABASE_URL!);
export async function getUser(id: string) {
  // HTTP query — no persistent socket, safe under massive fan-out.
  const rows = await sql`SELECT * FROM users WHERE id = ${id}`;
  return rows[0];
}
```

```text
# 2) A connection pooler (PgBouncer) sits between functions and Postgres.
#    Hundreds of function instances connect to PgBouncer; PgBouncer
#    multiplexes them onto a small, fixed pool of real DB connections.
#
#    functions (×500) ──▶ PgBouncer (transaction pooling) ──▶ Postgres (×20)
#
# Connect your app to the POOLER's port (e.g. 6543), not Postgres directly.
# Use "transaction" pooling mode for serverless; avoid prepared statements
# that span transactions, which transaction pooling can't preserve.
```

Rule of thumb: on serverless, never connect directly to a raw Postgres/MySQL port. Use a serverless/HTTP driver, a pooler, or both. On a long-running Node server (self-hosted, always-on), a normal in-process connection pool is fine and simpler.

## Multi-Region and Data Locality

Running compute in multiple regions reduces latency for global users — but only if the data is also close. The classic trap: you deploy functions to 10 regions, but your database lives in `us-east-1`. Now a request in Singapore runs locally, then makes three sequential queries each crossing the Pacific at ~200ms — your "edge" deployment is *slower* than a single us-east function.

Strategies:

- **Co-locate compute with the primary DB.** The simplest correct answer: pin functions to the region your database is in. One region, no cross-ocean queries.
- **Read replicas near users + writes to the primary.** Route reads to a regional replica, writes to the primary. Adds replication lag to reason about.
- **Globally distributed databases** (Turso, DynamoDB global tables, Spanner-like) put data near compute everywhere, at the cost of consistency model complexity.
- **Cache at the edge.** Often the best move: serve ISR/static from the CDN globally and only hit the central DB on cache miss/revalidation, so most users never pay the data-distance cost at all.

The decision hinges on read/write ratio and consistency needs. Read-heavy, eventually-consistent content scales beautifully with edge caching + replicas; write-heavy, strongly-consistent systems usually want a single region with compute co-located.

## CDN Strategy

The CDN is what turns origin compute into edge-speed delivery. Map each response type to a caching behavior:

| Response | Cache behavior |
| --- | --- |
| Static assets (`/_next/static`) | `immutable`, cached forever at the edge |
| Static pages (SSG) | Cached at the edge until deploy |
| ISR pages | Cached at the edge with `s-maxage`/`stale-while-revalidate` matching `revalidate` |
| Dynamic (per-user) | `private`/`no-store` — never shared-cached |

```ts
// app/feed/route.ts — explicit CDN caching for a shared, semi-fresh response.
export async function GET() {
  return new Response(JSON.stringify(await getFeed()), {
    headers: {
      // Serve cached for 60s, then serve stale while revalidating in the
      // background for up to an hour. Keeps origin load flat under spikes.
      "Cache-Control": "public, s-maxage=60, stale-while-revalidate=3600",
      "Content-Type": "application/json",
    },
  });
}
```

`stale-while-revalidate` is the single most effective lever against traffic spikes: the edge keeps serving the slightly-stale copy instantly while one background request refreshes it, so the origin sees a trickle instead of a flood.

## Concurrency, Memory, and Horizontal Scaling

- **Memory limits:** serverless functions have a fixed memory ceiling (often 1-3GB). Streaming large responses, loading big payloads into memory, or leaking per-request state will OOM the instance. Stream and paginate.
- **Concurrency:** a single Node server has finite event-loop capacity; a CPU-bound handler (image processing, big JSON serialization) blocks all concurrent requests on that instance. Offload heavy CPU work to a queue/worker or the optimizer/CDN.
- **Horizontal scaling:** add more instances behind a load balancer. This works *only if your app is stateless* — no in-memory session store, no local file cache assumed to be shared. (Hence the shared `cacheHandler` above.) Anything stateful must live in external storage (Redis, DB, object store).
- **Autoscaling lag:** new instances take seconds to boot; a sudden spike can outpace scaling. `stale-while-revalidate` and CDN caching absorb the spike while autoscaling catches up.

## Cost Awareness: Compute vs Cache vs Bandwidth

At scale, your bill is dominated by three lines, and optimizing the wrong one wastes effort:

- **Compute** (function invocations × duration × memory): driven by cache *misses* and dynamic rendering. Every page you make cacheable removes compute. This is usually the biggest lever — a page served from cache costs near zero compute.
- **Cache/storage** (ISR cache reads/writes, Redis, image transforms): grows with cardinality — number of unique cached pages and image variants. A million-SKU catalog with 6 image sizes each is a lot of cache entries.
- **Bandwidth** (egress): driven by asset weight × traffic. This is where the image and bundle lessons pay off directly — smaller assets, smaller bill.

The pattern that wins on all three: **cache aggressively** (cuts compute), **right-size assets** (cuts bandwidth), and **bound cardinality** (cuts cache cost) by not generating image/page variants you don't serve.

### Trade-offs: self-hosting vs Vercel at scale

| | Vercel (managed) | Self-hosted (e.g. on K8s/containers) |
| --- | --- | --- |
| Shared ISR cache | Built in | You configure `cacheHandler` + Redis |
| CDN / edge network | Built in, global | You add a CDN (CloudFront/Cloudflare) and tune it |
| Autoscaling | Automatic | You configure HPA/instance scaling |
| Image optimizer | Managed (per-transform cost) | Runs on your CPU, or offload to a CDN loader |
| Edge/middleware | First-class | Supported but more setup; some features need extra work |
| Cost model | Predictable per-usage, can get steep at very high volume | Lower unit cost at high scale, higher ops burden |
| Ops effort | Minimal | Significant (you own caching, scaling, monitoring) |

The honest framing: Vercel trades money for operational simplicity, and that trade is excellent until volume is high enough that the bill exceeds the cost of a platform team. Self-hosting wins on unit economics at large scale but only if you correctly reproduce the shared cache, CDN, autoscaling, and connection-pooling pieces — get any of those wrong and you'll have an outage Vercel would have prevented. Most teams should start managed and revisit only when the bill, compliance, or control requirements clearly justify the ops investment.

## Pitfalls

- **Multi-instance ISR without a shared `cacheHandler`.** Inconsistent content across instances and N× regeneration load. Use shared storage.
- **Direct DB connections from serverless.** Connection storms crash the database under load. Use a pooler (PgBouncer transaction mode) or a serverless/HTTP driver.
- **Edge compute far from the database.** Local functions making cross-region queries are slower than a single co-located region. Co-locate or cache.
- **Assuming local state survives horizontal scaling.** In-memory sessions/caches break across instances. Externalize all state.
- **Dynamic-rendering pages that could be cached.** Every uncacheable page is pure compute cost and origin load. Make it ISR/static where possible.
- **Unbounded image/page variant cardinality.** Generating sizes/variants you never serve inflates cache and transform costs. Trim `deviceSizes`/`imageSizes`.
- **No `stale-while-revalidate` on shared responses.** Traffic spikes hit the origin directly and overwhelm autoscaling.

## Exercises

1. Configure a self-hosted-style setup: add a Redis-backed `cacheHandler`, run two app instances behind a load balancer, trigger `revalidateTag`, and verify both instances serve the fresh page.
2. Audit your database connection strategy. If you're on serverless connecting directly to Postgres, switch to a pooler endpoint or a serverless driver and load-test the connection count before and after.
3. Add `Cache-Control: s-maxage=..., stale-while-revalidate=...` to a high-traffic shared endpoint and measure origin request volume under a simulated spike (e.g. with `k6` or `autocannon`).
4. Map your app's routes to the CDN strategy table: classify each as static, ISR, or dynamic, and fix any per-user page that's accidentally shared-cached or any cacheable page that's dynamic.
5. Build a one-page cost model: estimate monthly compute (cache-miss invocations), cache storage (unique entries × variants), and bandwidth (asset weight × traffic). Identify your single biggest line and the one change that would reduce it most.

## Key Takeaways

- Multi-instance ISR requires a shared `cacheHandler` (Redis/S3); without it you get inconsistent content and multiplied regeneration load.
- Never connect serverless functions directly to a raw DB port — use a pooler (PgBouncer) and/or a serverless/HTTP driver to survive connection storms.
- Edge compute only helps if data is close; co-locate compute with the database, use replicas, or lean on edge caching so most requests never touch the origin.
- `stale-while-revalidate` plus aggressive CDN caching is your best defense against traffic spikes and autoscaling lag.
- At scale your bill is compute, cache, and bandwidth — caching cuts compute, right-sizing assets cuts bandwidth, bounding cardinality cuts cache; self-host only when the savings clearly outweigh reproducing Vercel's caching/scaling/pooling correctly.

---

[← Previous: Edge Runtime, Middleware, and Personalization](04-edge-and-middleware.md) | [Back to Course Index](../README.md) | [Next: Authentication and Authorization Patterns →](../05-production-engineering/01-auth-patterns.md)
