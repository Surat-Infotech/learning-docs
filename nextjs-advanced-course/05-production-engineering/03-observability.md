# Observability: Logging, Tracing, and Monitoring

When a Next.js app is small, debugging is `console.log` and reloading the page. In production — multiple regions, serverless functions that cold-start, Server Components fetching from three services, a Server Action that intermittently times out — that approach is blind. Observability is the discipline of being able to answer "what happened to *this* request?" after the fact, without a debugger attached. This lesson covers the three pillars (logs, traces, metrics) as they apply specifically to the App Router, plus the part everyone skips: deciding what to alert on and how to keep the signal-to-noise (and the bill) sane.

## What You'll Learn

- Why `console.log` doesn't scale, and how to emit structured logs the server can actually query
- Wiring distributed tracing with OpenTelemetry through Next's built-in `instrumentation.ts`
- Capturing errors across all three surfaces: Server Components, client components, and Server Actions
- Collecting Web Vitals as Real User Monitoring (RUM) and why lab metrics aren't enough
- Correlating logs, traces, and errors with a single request ID
- Choosing what to alert on (SLOs over raw metrics) and avoiding alert fatigue
- The cost and noise trade-offs of high-cardinality data and verbose logging

## Why `console.log` Isn't Enough

`console.log("user logged in", userId)` produces a line of unstructured text. At scale this fails in three ways:

1. **You can't query it.** "Show me all failed logins for org X in the last hour" requires parsing free-form strings. Structured logs are JSON your platform indexes by field.
2. **You can't correlate it.** A single user action spans a Server Component render, two `fetch`es, and a DB query — possibly across functions. Loose log lines have no thread tying them together.
3. **It's lossy and unleveled.** No severity, no sampling, no context. In a serverless function, `console.log` may also be buffered or dropped on cold exit.

Replace it with a structured logger (Pino is the common choice — fast, JSON-native):

```ts
// lib/logger.ts
import "server-only";
import pino from "pino";

export const logger = pino({
  level: process.env.LOG_LEVEL ?? "info",
  // Redact secrets so they never hit your log store.
  redact: ["req.headers.authorization", "req.headers.cookie", "*.password"],
  base: { service: "web", env: process.env.VERCEL_ENV ?? "dev" },
});
```

```ts
// usage — structured fields, not interpolated strings
logger.info({ userId, orgId, requestId }, "user logged in");
logger.error({ err, invoiceId, requestId }, "invoice fetch failed");
```

Now "all errors for `invoiceId=123`" is a field filter, not a grep. Use levels deliberately: `error` for things that page someone, `warn` for degraded-but-handled, `info` for business events, `debug` for development (off in prod).

## Distributed Tracing with OpenTelemetry

A **trace** follows one request end to end as a tree of **spans** (render → fetch → DB query), each timed. It answers "where did the 800ms go?" — the question logs can't. Next.js has first-class OpenTelemetry support and auto-instruments its own internals (route rendering, `fetch`, metadata generation) once you register a tracer.

The hook is `instrumentation.ts` at the project root. Its `register()` runs once when the server boots:

```ts
// instrumentation.ts
export async function register() {
  // Only load the (Node-only) SDK in the Node runtime, not Edge.
  if (process.env.NEXT_RUNTIME === "nodejs") {
    await import("./instrumentation.node");
  }
}
```

```ts
// instrumentation.node.ts
import { NodeSDK } from "@opentelemetry/sdk-node";
import { OTLPTraceExporter } from "@opentelemetry/exporter-trace-otlp-http";
import { getNodeAutoInstrumentations } from "@opentelemetry/auto-instrumentations-node";
import { Resource } from "@opentelemetry/resources";
import { SemanticResourceAttributes } from "@opentelemetry/semantic-conventions";

const sdk = new NodeSDK({
  resource: new Resource({
    [SemanticResourceAttributes.SERVICE_NAME]: "web",
  }),
  // Export to any OTLP backend: Grafana Tempo, Honeycomb, Datadog, etc.
  traceExporter: new OTLPTraceExporter({ url: process.env.OTEL_EXPORTER_OTLP_ENDPOINT }),
  // Auto-instrument http, pg, redis, etc. — DB and outbound calls become spans for free.
  instrumentations: [getNodeAutoInstrumentations()],
});

sdk.start();
```

> The package name is `@vercel/otel` if you want a batteries-included setup that handles the runtime split and Next conventions for you — recommended unless you need the raw SDK.

Add your own spans around meaningful business operations so traces map to *your* concepts, not just framework internals:

```ts
// lib/dal.ts
import { trace } from "@opentelemetry/api";

const tracer = trace.getTracer("dal");

export async function getInvoice(id: string) {
  return tracer.startActiveSpan("getInvoice", async (span) => {
    span.setAttribute("invoice.id", id); // attributes, not raw values that explode cardinality
    try {
      const invoice = await db.invoice.findUnique({ where: { id } });
      return invoice;
    } catch (err) {
      span.recordException(err as Error);
      span.setStatus({ code: 2 }); // ERROR
      throw err;
    } finally {
      span.end();
    }
  });
}
```

The `instrumentation.ts` register hook is also the right place to start anything that must initialize once at boot (metrics SDKs, feature-flag clients).

## Error Monitoring Across All Three Surfaces

Errors happen in three different execution contexts, and a tool like Sentry needs configuring for each.

### Server Components and the server runtime

Unhandled errors in Server Components trigger `error.tsx`, but you want them *reported*, not just rendered. Capture them in the boundary and via global hooks:

```tsx
// app/dashboard/error.tsx
"use client"; // error boundaries must be client components
import { useEffect } from "react";
import * as Sentry from "@sentry/nextjs";

export default function Error({ error, reset }: { error: Error & { digest?: string }; reset: () => void }) {
  useEffect(() => {
    // `digest` correlates this client-visible error with the server log line.
    Sentry.captureException(error, { tags: { digest: error.digest } });
  }, [error]);
  return <button onClick={reset}>Try again</button>;
}
```

For errors thrown during render before any boundary (and in the Node runtime generally), Next exposes `onRequestError` in `instrumentation.ts`:

```ts
// instrumentation.ts (add to the existing file)
import * as Sentry from "@sentry/nextjs";

export async function onRequestError(err: unknown, request: unknown, context: unknown) {
  // Fires for server-side request errors across components, actions, and handlers.
  Sentry.captureException(err);
}
```

### Client components

Client-side errors (event handlers, effects) need browser instrumentation, initialized once. `@sentry/nextjs` wires this through the build so the client SDK loads automatically; you configure DSN, sample rate, and `tracesSampleRate` in its config files.

### Server Actions

Actions are easy to forget because they don't render UI on failure. Wrap them so failures are captured *and* a clean result is returned to the client:

```ts
// lib/with-monitoring.ts
import "server-only";
import * as Sentry from "@sentry/nextjs";
import { logger } from "./logger";

export function monitored<A extends unknown[], R>(name: string, fn: (...a: A) => Promise<R>) {
  return async (...args: A): Promise<R> => {
    try {
      return await fn(...args);
    } catch (err) {
      logger.error({ err, action: name }, "server action failed");
      Sentry.captureException(err, { tags: { action: name } });
      throw err; // rethrow so the action's own error handling still runs
    }
  };
}
```

```ts
// app/invoices/actions.ts
"use server";
import { monitored } from "@/lib/with-monitoring";

export const deleteInvoice = monitored("deleteInvoice", async (id: string) => {
  /* ...auth, validate, delete... */
});
```

## Web Vitals as Real User Monitoring

Lighthouse scores are *lab* data — one machine, one network profile. Your actual users are on mid-range phones and flaky networks, and that's what Google's Core Web Vitals (LCP, INP, CLS) score for ranking. Capture them from real sessions (RUM) with Next's `useReportWebVitals`:

```tsx
// app/web-vitals.tsx
"use client";
import { useReportWebVitals } from "next/web-vitals";

export function WebVitals() {
  useReportWebVitals((metric) => {
    // Send to your RUM endpoint. Use `navigator.sendBeacon` so it survives page unload.
    const body = JSON.stringify({ name: metric.name, value: metric.value, id: metric.id });
    navigator.sendBeacon?.("/api/vitals", body) ||
      fetch("/api/vitals", { body, method: "POST", keepalive: true });
  });
  return null;
}
```

Drop `<WebVitals />` in the root layout. Aggregate by the **75th percentile** (the CWV threshold metric), segmented by route and device class — averages hide the slow tail that actually hurts users and rankings.

## Correlating Everything with a Request ID

The payoff of observability is jumping from an error to its logs to its trace in seconds. The glue is one ID stamped on all three. Generate it (or reuse the platform's) in middleware and propagate it:

```ts
// middleware.ts
import { NextResponse } from "next/server";
import type { NextRequest } from "next/server";

export function middleware(req: NextRequest) {
  const requestId = req.headers.get("x-request-id") ?? crypto.randomUUID();
  const headers = new Headers(req.headers);
  headers.set("x-request-id", requestId); // available to handlers/components via headers()
  const res = NextResponse.next({ request: { headers } });
  res.headers.set("x-request-id", requestId); // echo to the client for support tickets
  return res;
}
```

Then attach `requestId` to every log line, set it as a span attribute, and add it as a Sentry tag. Now a user's support ticket ("error ID abc123") leads straight to the full picture.

## What to Alert On

The mistake is alerting on raw metrics ("CPU > 80%"). The discipline is alerting on **symptoms users feel**, expressed as SLOs:

- **Availability / error rate:** alert when the 5xx (or failed-action) rate over a window exceeds your budget — e.g., error ratio > 1% for 5 minutes.
- **Latency:** alert on p95/p99 of key routes and actions crossing a threshold, not the average.
- **Saturation:** the leading indicators that *predict* the above (queue depth, DB connection pool exhaustion, function concurrency limits).
- **Business KPIs:** checkout success rate, login success rate — these catch breakage that infra metrics miss entirely.

Frame these as a **Service Level Objective** with an **error budget**: "99.9% of checkouts succeed over 30 days." Burn-rate alerts (you're consuming the budget too fast) page far less noisily than threshold alerts on every blip.

### Cost and noise trade-offs

Observability bills scale with **volume** and **cardinality**:

- **Log volume.** `debug` everywhere in prod is expensive and drowns signal. Log business events and errors; sample high-frequency info logs.
- **Trace sampling.** Don't trace 100% of requests at scale — head- or tail-sample (e.g., keep all errors and slow traces, sample 5% of the rest). You keep the interesting traces without paying for every healthy one.
- **High-cardinality labels.** Never put a user ID or request ID in a *metric* label — each unique value is a new time series and costs explode. Cardinality belongs in logs and traces (where you query individual events), not metrics (which aggregate).
- **Alert fatigue is a cost too.** Every noisy alert trains the on-call to ignore alerts. Tune aggressively; a page that fires falsely is worse than no page.

### When to use what

Reach for **logs** to understand a specific event's details, **traces** to understand *where time/failure went* across services, and **metrics** to understand *aggregate trends and alert*. They're complementary: an alert (metric) tells you something's wrong, a trace tells you where, and logs tell you exactly what.

## Pitfalls

- **Unstructured `console.log` in prod** — unqueryable, uncorrelated, and sometimes dropped on serverless cold exit.
- **Forgetting the Edge/Node split** in `instrumentation.ts` — the Node OTel SDK crashes on the Edge runtime; gate on `NEXT_RUNTIME`.
- **Only instrumenting client errors** and missing Server Component / Action failures — they fail silently otherwise. Use `onRequestError` and wrap actions.
- **Logging secrets** — auth headers, cookies, tokens. Redact at the logger config level, not per-call.
- **High-cardinality metric labels** (user/request IDs) — blows up your metrics bill. Keep cardinality in logs/traces.
- **Tracing 100% of traffic** at scale — costly and unnecessary. Tail-sample to keep errors and slow requests.
- **Alerting on causes, not symptoms** — "CPU high" pages you for things users never notice; alert on SLO burn instead.
- **Averaging Web Vitals** — the p75/p99 tail is where the pain (and the ranking penalty) lives.

## Exercises

1. Replace `console.log` in your DAL with a Pino logger that emits JSON, redacts auth headers, and includes a `requestId`. Query your log store by `requestId`.
2. Set up `instrumentation.ts` with `@vercel/otel` (or the raw SDK), export traces to a local collector (Jaeger/Tempo), and add a custom span around one DAL function. Find where time goes on a slow route.
3. Configure Sentry for all three surfaces. Throw an error in a Server Component, a client handler, and a Server Action, and confirm all three appear with distinguishing tags.
4. Add `useReportWebVitals`, beacon the metrics to a Route Handler, and compute the p75 INP per route. Compare it to your Lighthouse INP.
5. Define one SLO (e.g., 99.5% of logins succeed) and write the burn-rate alert condition for it. Explain why it's quieter than a raw threshold alert.

## Key Takeaways

- Structured JSON logs (Pino) are queryable and redactable; `console.log` is none of that at scale.
- `instrumentation.ts` is Next's OpenTelemetry hook — auto-instrument framework internals and add custom spans for your own operations, gating the Node SDK on `NEXT_RUNTIME`.
- Error monitoring needs three configurations: error boundaries + `onRequestError` (server), the client SDK, and explicit wrapping for Server Actions.
- Capture Web Vitals as RUM and judge them at p75, not by lab averages.
- A single request ID across logs, traces, and errors turns minutes of forensics into seconds.
- Alert on user-felt symptoms via SLOs and error budgets; control cost and noise with sampling, low-cardinality metrics, and aggressive alert tuning.

---

[← Previous: Security Hardening](02-security.md) | [Back to Course Index](../README.md) | [Next: Testing Strategy →](04-testing-strategy.md)
