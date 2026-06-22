# Testing Strategy for Next.js

The App Router broke a lot of testing intuition. Async Server Components don't render in jsdom the way page-level components used to. Server Actions look like functions but behave like endpoints. The boundary between "unit" and "integration" got blurry because a single file can contain server data access *and* client interactivity. The result is teams that either test nothing meaningful or over-invest in slow, flaky E2E. This lesson lays out a pragmatic test pyramid for Next.js — what to test where, what's currently awkward and how to work around it, and how to keep the suite fast enough that people actually run it.

## What You'll Learn

- A pragmatic test pyramid tuned to the App Router's realities
- Unit-testing pure logic and the Data Access Layer in isolation
- Component testing with Vitest (or Jest) and React Testing Library
- Why async Server Components are awkward to unit-test today, and the practical workaround
- How to test Server Actions as the functions-plus-endpoints they are
- E2E with Playwright: auth setup, fixtures, and a real test database
- What to mock versus what to exercise for real, and keeping CI fast

## A Pragmatic Pyramid for Next

The classic pyramid (many unit, fewer integration, few E2E) still holds, but the *layers* map differently in Next:

```text
        /\        E2E (Playwright)          ← whole routes, real DB, real auth flow
       /  \         - the only honest test of a rendered Server Component
      /----\      Integration / component   ← client components + RTL, DAL against test DB
     /      \       - Server Actions called as functions
    /--------\    Unit (Vitest)             ← pure logic, validation schemas, RBAC, mappers
```

The shift from the SPA era: because async Server Components are hard to render in a unit test, **more of your "does this page work?" confidence moves into E2E**, and you compensate by pushing logic *out* of components into testable pure functions and a DAL you can test directly. Keep the pyramid bottom-heavy anyway — E2E is slow and flaky; lean on it for critical user journeys, not for every branch.

## Unit-Testing Pure Logic and the DAL

The cheapest, most valuable tests target code with no framework entanglement. Extract decisions into pure functions and test them exhaustively:

```ts
// lib/abac.test.ts
import { describe, it, expect } from "vitest";
import { canEditInvoice } from "./abac";

describe("canEditInvoice", () => {
  const base = { id: "u1", role: "member" as const, orgId: "org1" };

  it("denies cross-tenant access", () => {
    const invoice = { orgId: "org2", createdById: "u1", locked: false };
    expect(canEditInvoice(base, invoice)).toBe(false);
  });

  it("lets admins edit locked invoices", () => {
    const admin = { ...base, role: "admin" as const };
    const invoice = { orgId: "org1", createdById: "u9", locked: true };
    expect(canEditInvoice(admin, invoice)).toBe(true);
  });
});
```

Validation schemas deserve their own tests — they're a security boundary:

```ts
// lib/schemas.test.ts
import { it, expect } from "vitest";
import { UpdateProfile } from "./schemas";

it("rejects an over-long bio", () => {
  const r = UpdateProfile.safeParse({ displayName: "x", age: 30, bio: "a".repeat(600) });
  expect(r.success).toBe(false);
});
```

For the **DAL**, you have a choice: mock the DB, or run against a real test database. Mocking is fast but tests your mock, not your SQL. The higher-value approach is to point the DAL at an ephemeral test database (a Docker Postgres, or a fresh schema per run) so the query, constraints, and tenant scoping are exercised for real:

```ts
// lib/dal.integration.test.ts
import { it, expect, beforeEach } from "vitest";
import { getInvoice } from "./dal";
import { seed, resetDb, asUser } from "@/test/db";

beforeEach(async () => resetDb()); // truncate + reseed for isolation

it("returns 404 for another org's invoice", async () => {
  const { orgA, orgB } = await seed();
  await asUser(orgB.user, async () => {
    // Tenant scoping is the whole point — assert it against real SQL.
    await expect(getInvoice(orgA.invoiceId)).rejects.toThrow("Not found");
  });
});
```

This is where bugs actually live (the IDOR from the auth lesson), so it's worth the extra setup over a mock.

## Component Testing with Vitest + RTL

For **client** components — the interactive ones — Vitest with React Testing Library is the standard. Test behavior the user observes, not implementation details:

```tsx
// components/Counter.test.tsx
import { render, screen } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import { it, expect } from "vitest";
import { Counter } from "./Counter";

it("increments on click", async () => {
  const user = userEvent.setup();
  render(<Counter start={0} />);
  await user.click(screen.getByRole("button", { name: /increment/i }));
  expect(screen.getByText("Count: 1")).toBeInTheDocument();
});
```

Configure Vitest with the jsdom environment and the React plugin:

```ts
// vitest.config.ts
import { defineConfig } from "vitest/config";
import react from "@vitejs/plugin-react";

export default defineConfig({
  plugins: [react()],
  test: {
    environment: "jsdom",
    setupFiles: ["./test/setup.ts"], // imports @testing-library/jest-dom matchers
    globals: true,
  },
});
```

Vitest is generally the better fit for new Next projects (fast, ESM-native, shares config with Vite); Jest still works and may already be in your stack. The testing approach is identical either way.

## The Async Server Component Problem

Here's the awkward truth: **you cannot cleanly unit-test an async Server Component today.** RTL's `render()` expects a synchronous component tree; an `async function Page()` returns a promise, and the React Server Component rendering path isn't wired into jsdom test renderers in a supported way. Hacks exist (awaiting the component and rendering its resolved output), but they're brittle and don't model RSC behavior (streaming, server-only APIs like `cookies()`).

The pragmatic, officially-aligned strategy is to **split the concern**:

1. **Push all logic out of the Server Component** into the DAL and pure functions — and unit/integration-test *those* directly (above). A good Server Component is nearly logic-free: it awaits data and arranges client components.
2. **E2E-test the rendered route** to verify the wiring — that the page actually fetches, authorizes, and renders. This is the only honest test of the full server render.

```tsx
// app/invoices/[id]/page.tsx — thin by design, so there's little to unit-test
import { getInvoice } from "@/lib/dal";        // tested via DAL integration test
import { formatMoney } from "@/lib/money";      // tested via pure unit test
import { InvoiceActions } from "./invoice-actions"; // client component, RTL-tested

export default async function Page({ params }: { params: Promise<{ id: string }> }) {
  const { id } = await params;
  const invoice = await getInvoice(id);
  return (
    <main>
      <h1>{invoice.number}</h1>
      <p>{formatMoney(invoice.total)}</p>
      <InvoiceActions id={invoice.id} />
    </main>
  );
}
```

Every meaningful behavior here is tested *somewhere testable*; the component itself is verified end-to-end. Don't fight the framework to unit-test the glue.

## Testing Server Actions

A Server Action is an exported async function, so you can import and call it directly — but its auth and DB dependencies are real. Two complementary approaches:

```ts
// app/invoices/actions.test.ts — integration: real test DB, mocked session
import { it, expect, vi, beforeEach } from "vitest";
import { resetDb, seed } from "@/test/db";

// Mock only the identity boundary; let the actual DB work run.
vi.mock("@/lib/dal", async (orig) => ({
  ...(await orig<typeof import("@/lib/dal")>()),
  verifySession: vi.fn(),
}));

import { deleteInvoice } from "./actions";
import { verifySession } from "@/lib/dal";

beforeEach(async () => resetDb());

it("forbids a viewer from deleting", async () => {
  const { viewer } = await seed();
  vi.mocked(verifySession).mockResolvedValue({ userId: viewer.id, role: "viewer", orgId: viewer.orgId });
  await expect(deleteInvoice({ invoiceId: "any" })).rejects.toThrow("Forbidden");
});

it("rejects malformed input before touching the DB", async () => {
  vi.mocked(verifySession).mockResolvedValue({ userId: "u", role: "admin", orgId: "o" });
  await expect(deleteInvoice({ invoiceId: 123 } as never)).rejects.toThrow(); // Zod fails
});
```

This covers the action's *logic* (authz, validation, the mutation). To prove it works as an actual endpoint wired into a form, cover the happy path in E2E. Mock the session boundary, but let validation and the DB run for real — those are the parts that break.

## E2E with Playwright

Playwright drives a real browser against a running app. It's your only honest test of Server Components, navigation, and the full auth flow. The setup details that matter:

### Authenticated state via a setup project

Logging in through the UI in every test is slow and flaky. Do it **once**, save the storage state, and reuse it:

```ts
// e2e/auth.setup.ts
import { test as setup } from "@playwright/test";

const file = "e2e/.auth/user.json";

setup("authenticate", async ({ page }) => {
  await page.goto("/login");
  await page.getByLabel("Email").fill(process.env.E2E_USER_EMAIL!);
  await page.getByLabel("Password").fill(process.env.E2E_USER_PASSWORD!);
  await page.getByRole("button", { name: "Sign in" }).click();
  await page.waitForURL("/dashboard");
  // Persist cookies/localStorage so other tests start logged in.
  await page.context().storageState({ path: file });
});
```

```ts
// playwright.config.ts
import { defineConfig } from "@playwright/test";

export default defineConfig({
  // Start the app once for the whole run.
  webServer: { command: "npm run start", url: "http://localhost:3000", reuseExistingServer: !process.env.CI },
  projects: [
    { name: "setup", testMatch: /auth\.setup\.ts/ },
    {
      name: "chromium",
      dependencies: ["setup"], // run setup first
      use: { storageState: "e2e/.auth/user.json", baseURL: "http://localhost:3000" },
    },
  ],
});
```

### Fixtures and a real test database

E2E needs deterministic data. Seed a dedicated test database before the run and reset it between specs (or use per-test transactions/orgs for isolation). Point the app's `DATABASE_URL` at it via `.env.test`. Never run E2E against staging data — tests that mutate shared state are flaky by construction.

```ts
// e2e/invoices.spec.ts
import { test, expect } from "@playwright/test";

test("user sees only their org's invoices", async ({ page }) => {
  await page.goto("/invoices");
  await expect(page.getByRole("heading", { name: "INV-1001" })).toBeVisible();
  await page.goto("/invoices/other-org-id"); // a known foreign invoice from seed data
  await expect(page.getByText("Not found")).toBeVisible(); // proves server-side authz
});
```

## What to Mock vs Not

The rule: **mock at the trust boundary, exercise everything inside it.**

- **Mock:** third-party network calls (payment providers, email, external APIs) — they're slow, flaky, cost money, and you don't own them. Mock the *identity boundary* (`verifySession`) in unit/integration tests so you control the actor.
- **Don't mock:** your own DB queries, validation schemas, and business logic. A mocked Prisma client tests your mock; a real test DB tests your SQL, constraints, and the tenant-scoping bugs that actually ship. Don't mock the framework.

Mocking the database is the most common over-mock — it produces green tests that pass while production breaks on a missing index or a wrong `where` clause.

## Keeping CI Fast

- **Parallelize.** Vitest and Playwright shard across workers; split E2E across CI machines. A 20-minute suite that runs in 4 minutes parallel is the difference between people running it and not.
- **Layer by speed in CI.** Run unit + component on every push (seconds). Run E2E on PRs and pre-merge (minutes). Don't gate every commit on the slow tier.
- **Reuse the build.** Build once, run E2E against `next start`, not `next dev` — dev mode is slower and behaves differently.
- **Isolate test data cheaply.** Truncate-and-reseed or per-test transactions beat dropping/recreating the schema every test.
- **Quarantine flakes.** A flaky E2E that's "usually green" erodes trust in the whole suite. Fix it or quarantine it; never let it train people to re-run blindly.

## Pitfalls

- **Trying to unit-test async Server Components.** It's not cleanly supported — push logic out and E2E the route instead.
- **Mocking the database.** Green tests, broken prod. Use a real test DB for the DAL and Server Actions.
- **Logging into the UI in every E2E test** — slow and flaky. Authenticate once via a setup project and reuse storage state.
- **E2E against shared/staging data** — non-deterministic and destructive. Use a dedicated, resettable test DB.
- **Testing implementation details** (state internals, class names) instead of user-observable behavior — brittle tests that break on every refactor.
- **All-E2E or all-unit.** Too much E2E is slow and flaky; too little misses the server render entirely. Balance the pyramid.
- **Gating every commit on the full E2E suite** — kills iteration speed. Layer tests by speed.

## Exercises

1. Extract one decision currently buried in a Server Component into a pure function and write exhaustive unit tests for it. Note how thin the component becomes.
2. Set up a Dockerized Postgres test DB and write a DAL integration test proving cross-tenant access returns 404. Then break the `where` clause and confirm the test catches it (a mocked DB wouldn't).
3. Test a Server Action three ways: a viewer is forbidden, malformed input is rejected pre-DB, and an admin succeeds and mutates the test DB.
4. Configure a Playwright setup project that authenticates once and reuses storage state, then write an E2E test that verifies server-side authorization on a route.
5. Profile your CI: split tests into a fast tier (every push) and a slow tier (PRs), parallelize each, and report the before/after wall-clock time.

## Key Takeaways

- Build a bottom-heavy pyramid, but expect more route-level confidence to live in E2E because async Server Components resist unit testing.
- Push logic out of Server Components into pure functions and the DAL, and test those directly — keep the component thin and verify it end-to-end.
- Don't mock your own database; run the DAL and Server Actions against a real test DB to catch the tenant-scoping bugs that actually ship.
- Test client components with Vitest + RTL by behavior; mock only the trust boundary (third parties, identity).
- For E2E, authenticate once via a Playwright setup project, use a dedicated resettable test database, and keep tests deterministic.
- Keep CI fast by layering tests by speed, parallelizing, building once, and ruthlessly fixing flakes.

---

[← Previous: Observability: Logging, Tracing, and Monitoring](03-observability.md) | [Back to Course Index](../README.md) | [Next: CI/CD and Deployment Strategies →](05-cicd-deployment.md)
