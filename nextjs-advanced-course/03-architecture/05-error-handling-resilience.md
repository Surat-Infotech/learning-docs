# Error Handling and Resilience Patterns

Most error handling in tutorials is a `try/catch` that logs and re-throws. That's fine for a script and dangerous for a product. At scale, the question isn't "how do I catch errors" but "which errors are *expected business outcomes* I should model as data, which are *unexpected failures* I should let bubble to a boundary, and how do I keep the app standing when an upstream is flaky?" Conflating these is how apps leak stack traces to users, render blank pages on a single failed widget, and throw away the information you needed to debug. This lesson draws those lines.

## What You'll Learn

- The expected vs unexpected error distinction and why it drives every other decision
- Modeling expected errors as return values (Result types) instead of throwing
- Error boundaries (`error.tsx`, `global-error.tsx`) and choosing their granularity
- `notFound()` and `redirect()` as control flow, not errors
- Retries, timeouts, and graceful degradation for flaky upstreams
- Never leaking internal details to users while preserving them for logs
- Wiring a centralized error-logging hook-in

## Expected vs Unexpected Errors

The foundational distinction:

- **Expected errors** are normal outcomes of a valid operation: a form fails validation, a login is wrong, an item is out of stock, a coupon is expired. These are *part of the domain*. They should be **modeled as return values** and handled in the UI as ordinary states.
- **Unexpected errors** are bugs and infrastructure failures: a null deref, a database connection drop, an out-of-memory. These you cannot meaningfully recover from in the calling code; they should **throw** and be caught by an error boundary that shows a fallback and logs the cause.

The rule of thumb from the Next docs: **don't use `try/catch` for expected errors — return them; let unexpected errors throw to a boundary.** Getting this split right is what makes the rest of the chapter coherent.

## Modeling Expected Errors as Result Types

For expected errors in Server Actions and the DAL, return a typed result instead of throwing. A `Result` (a.k.a. discriminated union) forces the caller to handle both branches at compile time and keeps validation errors flowing cleanly to the UI:

```ts
// lib/result.ts
export type Result<T, E = string> =
  | { ok: true; data: T }
  | { ok: false; error: E; fieldErrors?: Record<string, string[]> };
```

```ts
// features/checkout/actions.ts
"use server";
import { Result } from "@/lib/result";
import { purchaseSchema } from "./schema";
import { reserveStock } from "./queries";

export async function checkout(
  formData: FormData,
): Promise<Result<{ orderId: string }>> {
  const parsed = purchaseSchema.safeParse(Object.fromEntries(formData));
  if (!parsed.success) {
    // Expected: validation failure → return, don't throw.
    return { ok: false, error: "Invalid input", fieldErrors: parsed.error.flatten().fieldErrors };
  }

  const stock = await reserveStock(parsed.data.productId);
  if (!stock.available) {
    // Expected: business rule → return as data.
    return { ok: false, error: "This item is out of stock." };
  }

  // If db.createOrder throws here, that's UNEXPECTED — let it bubble to the boundary.
  const order = await createOrder(parsed.data);
  return { ok: true, data: { orderId: order.id } };
}
```

The client consumes the discriminated union and renders the expected-error branch as UI, never as a crash:

```tsx
// The "out of stock" case is a normal UI state, not an error screen.
const result = await checkout(formData);
if (!result.ok) {
  setMessage(result.error); // shown inline; app stays interactive
}
```

### Trade-offs

Result types add a tiny bit of ceremony versus throwing, and they only make sense for *expected* failures — wrapping genuinely unexpected errors in a Result just hides bugs that should surface and be logged. Use Results at the boundaries you own (DAL, Server Actions) for domain outcomes; let infrastructure failures throw. A small `safe()` wrapper that turns a throwing call into a Result is handy, but don't `safe()` everything — you'll swallow the bugs you most need to see.

## Error Boundaries and Granularity

Unexpected errors thrown during rendering are caught by the nearest `error.tsx`. Its placement determines blast radius — and that's a design decision, not an afterthought.

```tsx
// app/dashboard/error.tsx  — scoped boundary for the dashboard segment only
"use client"; // error boundaries must be client components

export default function DashboardError({
  error,
  reset,
}: {
  error: Error & { digest?: string };
  reset: () => void;
}) {
  // `error.digest` correlates this UI to the server-logged stack — show the digest, not the message.
  return (
    <div role="alert">
      <p>Couldn&apos;t load the dashboard.</p>
      <button onClick={reset}>Try again</button>
    </div>
  );
}
```

Granularity guidance:

- **Per-segment `error.tsx`** isolates failures: if the analytics widget throws, the rest of the dashboard survives because the boundary sits around just that segment. Place boundaries at the seams where partial failure is acceptable.
- A boundary catches errors in its child segments and below — but **not** in the layout at its own level. To protect a layout, the boundary must live one level up.
- **`global-error.tsx`** is the last resort: it replaces the root layout (so it renders its own `<html>`/`<body>`) and catches errors the root layout itself throws. It only runs in production. Keep it minimal and dependency-free — it runs when everything else has failed.

Combine boundaries with Suspense for graceful partial UIs: wrap an independently-failing widget in both its own Suspense (loading) and an error boundary (failure), so one slow or broken widget degrades to a fallback while the page renders.

### Trade-offs

One boundary at the root is simplest but turns any single failure into a whole-page error screen. Fine-grained boundaries preserve more of the UI but add files and fallback components to maintain. Put boundaries where *partial failure is tolerable and isolation has value* — around independent dashboard cards, async lists, embedded third-party widgets — and don't litter them where a failure should genuinely take down the whole view.

## `notFound()` and `redirect()` as Control Flow

`notFound()` and `redirect()` work by throwing a special internal error that Next intercepts — so they are *control flow*, not errors to catch. Two consequences matter:

1. Call them to express intent: a missing record → `notFound()` (renders the nearest `not-found.tsx`); an unauthenticated user → `redirect("/login")`.
2. **Never wrap them in a `try/catch` that swallows the thrown signal.** If you `catch` around a `redirect()`, you intercept its control-flow throw and the redirect silently dies. Call them outside `try/catch`, or re-throw in the catch.

```tsx
// app/posts/[slug]/page.tsx
import { notFound } from "next/navigation";
import { getPost } from "@/features/posts/queries";

export default async function PostPage({ params }: { params: Promise<{ slug: string }> }) {
  const { slug } = await params;
  const post = await getPost(slug);
  if (!post) notFound(); // control flow → renders not-found.tsx, not an error
  return <Article post={post} />;
}
```

## Retries, Timeouts, and Graceful Degradation

Upstreams fail transiently — a payment gateway hiccups, a recommendations service times out. Resilience means bounding the wait, retrying the transient, and degrading gracefully when neither works.

```ts
// lib/http.ts
// Bound every external call with a timeout; never wait indefinitely.
export async function fetchWithTimeout(url: string, ms = 3000, init?: RequestInit) {
  const controller = new AbortController();
  const timer = setTimeout(() => controller.abort(), ms);
  try {
    return await fetch(url, { ...init, signal: controller.signal });
  } finally {
    clearTimeout(timer);
  }
}

// Retry only idempotent/transient failures, with backoff and a cap.
export async function withRetry<T>(fn: () => Promise<T>, attempts = 3): Promise<T> {
  let lastErr: unknown;
  for (let i = 0; i < attempts; i++) {
    try {
      return await fn();
    } catch (err) {
      lastErr = err;
      await new Promise((r) => setTimeout(r, 2 ** i * 100)); // exponential backoff
    }
  }
  throw lastErr;
}
```

Graceful degradation means a non-critical failure shouldn't take down the page. If the recommendations service is down, render the page *without* recommendations rather than erroring the whole route:

```tsx
// Degrade: a failed non-critical fetch returns a fallback, page still renders.
async function getRecommendations(id: string) {
  try {
    return await withRetry(() => fetchRecs(id));
  } catch {
    return []; // empty rather than fatal; log it, render the rest of the page
  }
}
```

### Trade-offs

Retries help only transient, *idempotent* failures — never blindly retry a non-idempotent mutation (a double-charge is worse than a failed charge). Aggressive retries can amplify an outage into a thundering herd; cap attempts, use backoff, and consider a circuit breaker for a persistently-down dependency. Timeouts must be tuned: too short and you fail healthy-but-slow calls, too long and you tie up resources. Choose per-dependency based on its SLO.

## Don't Leak Internals; Do Log Them

Two simultaneous obligations: the user must never see a stack trace, SQL string, or internal hostname, and you must never *lose* that detail for debugging. Next supports this split natively — in production, server errors sent to the client are stripped to a generic message plus a `digest` hash, while the full error is logged server-side. Honor the same discipline in your own code:

```ts
// Log the real error with context; return a sanitized, generic message.
import { logError } from "@/lib/logger";

export async function safeCharge(input: ChargeInput): Promise<Result<Receipt>> {
  try {
    return { ok: true, data: await charge(input) };
  } catch (err) {
    const ref = logError(err, { op: "charge", userId: input.userId }); // full detail + correlation id
    return { ok: false, error: `Payment failed. Reference: ${ref}` }; // user sees a ref, not internals
  }
}
```

The correlation id / `digest` is the bridge: the user reports it, you find the full trace in your logs. Never interpolate raw error messages into user-facing UI — they routinely contain connection strings, file paths, and table names.

## Centralized Error Logging

Scatter logging and you'll have inconsistent context and miss errors entirely. Centralize it:

- One `lib/logger` (or an `instrumentation.ts` `onRequestError` hook) is the single place that ships errors to your observability backend (Sentry, etc.), attaching request context, user id, and the digest.
- Next's `instrumentation.ts` exports `onRequestError`, called for every server-side error — wire your reporter there so even errors caught by boundaries are captured once, consistently.

```ts
// instrumentation.ts  — one hook captures all server errors with context.
export async function onRequestError(err: unknown, request: Request, context: object) {
  await sendToObservability({ err, url: request.url, context });
}
```

This guarantees every unexpected error reaches your dashboards with uniform metadata, regardless of which boundary rendered the fallback.

## Pitfalls

- **`try/catch` around expected failures.** Turns ordinary outcomes into control-flow noise. Return Results instead.
- **Catching `notFound()`/`redirect()`.** Swallows their control-flow throw; the navigation silently fails. Keep them out of `try/catch` or re-throw.
- **A single root error boundary for everything.** One widget failure blanks the whole app. Scope boundaries to tolerable-failure seams.
- **Leaking error messages into the UI.** Exposes internals (paths, SQL, hosts). Show a generic message plus a reference id; log the rest.
- **Blind retries on mutations.** Non-idempotent retries cause double effects (double charges). Retry only idempotent/transient operations, with backoff and a cap.
- **Unbounded external calls.** A hung upstream ties up your server. Always set a timeout.
- **Scattered logging.** Inconsistent context and missed errors. Centralize via a logger and `instrumentation.ts`.

## Exercises

1. Take a `login` Server Action that throws on bad credentials. Refactor it to return a `Result` with field errors, and update the form to render the expected-error branch as inline UI rather than an error screen.
2. Design the error-boundary placement for a dashboard with four independent widgets, two of which call flaky third-party APIs. Decide where `error.tsx` and Suspense go so a single widget failure degrades gracefully.
3. Audit a route that wraps `getUserOrRedirect()` (which calls `redirect()`) in a `try/catch`. Explain the bug and fix it two different ways.
4. Implement `fetchWithTimeout` + `withRetry` for a recommendations call, and wire graceful degradation so the product page renders without recs on failure. State which operations you would *not* retry and why.
5. Wire `instrumentation.ts`'s `onRequestError` to a logging function that attaches a correlation id, and show how that id appears to the user (in the fallback) and to you (in the log) for the same incident.

## Key Takeaways

- Split errors into expected (model as returned Results, handle as UI states) and unexpected (throw to a boundary, log, show a fallback) — this distinction drives everything else.
- Return discriminated-union Results from the DAL and Server Actions for domain outcomes; reserve throwing for genuine failures so bugs stay visible.
- Place `error.tsx` boundaries at seams where partial failure is tolerable; `global-error.tsx` is a minimal production-only last resort.
- `notFound()` and `redirect()` are control flow that throw internally — never swallow them in a `try/catch`.
- Bound external calls with timeouts, retry only idempotent/transient failures with backoff, and degrade non-critical features instead of failing the whole route.
- Show users a generic message plus a correlation id; log full detail centrally via a logger and `instrumentation.ts`'s `onRequestError`.

---

[← Previous: End-to-End Type Safety](04-type-safety.md) | [Back to Course Index](../README.md) | [Next: Core Web Vitals and Performance Budgets →](../04-performance-and-scaling/01-web-vitals.md)
