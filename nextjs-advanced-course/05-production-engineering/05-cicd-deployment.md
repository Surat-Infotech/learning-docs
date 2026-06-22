# CI/CD and Deployment Strategies

Shipping Next.js is more than `git push`. A production pipeline catches broken types before they merge, runs migrations in the right order, promotes the same artifact through environments, and lets you roll back in seconds when something slips through. The deployment target shapes everything: Vercel makes most of this invisible, while self-hosting hands you the full responsibility (and control). This lesson builds a CI pipeline, contrasts Vercel with a self-hosted Docker setup, and covers the operational pieces — migrations, feature flags, and rollback — that separate a hobby deploy from a production one.

## What You'll Learn

- A CI pipeline that gates merges on typecheck, lint, test, and build (with a GitHub Actions example)
- Why per-PR preview deployments are worth the setup, and how environment promotion works
- Managing environment variables safely across preview, staging, and production
- Vercel versus self-hosting: `output: 'standalone'`, a Dockerfile sketch, and reverse-proxy concerns
- Running database migrations safely inside the deploy pipeline
- Decoupling deploy from release with feature flags and safe rollout patterns
- Building a rollback strategy you can execute in seconds, under pressure

## A CI Pipeline That Gates Merges

CI exists to make broken code *impossible to merge*, not just to notice it later. The four gates, in increasing cost order, so you fail fast and cheap:

```yaml
# .github/workflows/ci.yml
name: CI
on:
  pull_request:
  push:
    branches: [main]

jobs:
  verify:
    runs-on: ubuntu-latest
    services:
      # A real Postgres for DAL/integration tests — not a mock.
      postgres:
        image: postgres:16
        env: { POSTGRES_PASSWORD: postgres, POSTGRES_DB: test }
        ports: ["5432:5432"]
        options: >-
          --health-cmd pg_isready --health-interval 10s
          --health-timeout 5s --health-retries 5
    env:
      DATABASE_URL: postgres://postgres:postgres@localhost:5432/test
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 20, cache: npm }
      - run: npm ci # exact lockfile install; never `npm install` in CI

      # Gate 1: types. Cheapest, catches the most. `tsc` with no emit.
      - run: npx tsc --noEmit
      # Gate 2: lint. Style + likely-bug rules.
      - run: npm run lint
      # Gate 3: tests. Unit + integration against the service DB above.
      - run: npm run test -- --run
      # Gate 4: build. The real production build — catches RSC/bundling errors
      # that nothing else does.
      - run: npm run build
```

Make these gates **required status checks** on the protected branch so a red pipeline blocks merge. Order matters: `tsc` in seconds before a multi-minute `build` means most failures surface fast. Cache `node_modules` and the Next build cache (`.next/cache`) between runs to keep it quick. E2E (Playwright) usually runs in a separate, parallel job or a later stage so it doesn't block fast feedback on the cheap gates.

## Preview Deployments Per PR

A **preview deployment** is a live, isolated URL for every PR, built from that branch. Their value is enormous and under-appreciated:

- **Reviewers test the real thing**, not a description. Design, product, and QA can click through before merge.
- **It catches environment-specific bugs** — things that only break in a production build (the `build` gate plus a running deploy).
- **It's a shareable artifact** for stakeholders without local setup.

On Vercel this is automatic per PR. Self-hosting, you replicate it with ephemeral environments (a per-branch deploy, a preview namespace in Kubernetes, or a containerized branch deploy behind a wildcard subdomain). The principle is the same: every PR gets a disposable, production-built environment, torn down on merge/close.

## Environment Promotion and Variables

Promote the **same build artifact** through stages rather than rebuilding per environment — rebuilding risks "works in staging, breaks in prod" drift:

```text
PR branch  →  preview     (ephemeral, per-PR, seeded/test data)
main merge →  staging     (production-like, integration-tested, prod-shaped data)
release    →  production  (the same artifact, promoted)
```

### Environment variable management

The discipline that prevents most deploy incidents:

- **Never commit secrets.** Use the platform's secret store (Vercel env vars, GitHub Actions secrets, a vault). Commit only a `.env.example` documenting the *names*.
- **Scope variables per environment.** Preview points at a test DB; production at the real one. The same variable *name* (`DATABASE_URL`) resolves to different *values* per stage — that's the whole point.
- **Remember `NEXT_PUBLIC_` is build-time and baked in** (from the security lesson). If a public var differs between staging and prod, you cannot promote the same artifact across them — it was inlined at build. Either keep public config identical across stages or rebuild per stage for those. Server-side (non-public) vars are read at *runtime* and can differ freely on the same artifact.
- **Validate env at startup.** Parse the environment with Zod on boot so a missing/malformed variable fails the deploy loudly instead of at the first request.

```ts
// lib/env.ts
import { z } from "zod";

// Fails fast at boot if anything's missing — better than a 3am runtime crash.
export const env = z
  .object({
    DATABASE_URL: z.string().url(),
    AUTH_SECRET: z.string().min(32),
    NODE_ENV: z.enum(["development", "test", "production"]),
  })
  .parse(process.env);
```

## Vercel vs Self-Hosting

### Vercel

The first-party platform: zero-config preview deploys, automatic env scoping, edge network, and built-in support for every Next feature (ISR, streaming, image optimization, middleware). You trade money and vendor coupling for nearly zero ops. **Choose it** when team time is more valuable than infra control and your scale fits the pricing.

### Self-hosting with Docker

Full control and portability, at the cost of owning the ops. The key Next feature for this is `output: 'standalone'`, which traces exactly the files the server needs and emits a minimal, self-contained server — yielding a tiny image instead of shipping all of `node_modules`:

```ts
// next.config.ts
const nextConfig = {
  output: "standalone", // emit a minimal server bundle for Docker
};
export default nextConfig;
```

```dockerfile
# Dockerfile — multi-stage; ships only the standalone output
FROM node:20-alpine AS deps
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci

FROM node:20-alpine AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
# NEXT_PUBLIC_ vars are inlined HERE, at build time — pass them as build args if needed.
RUN npm run build

FROM node:20-alpine AS runner
WORKDIR /app
ENV NODE_ENV=production
# Run as non-root.
RUN addgroup -S nodejs && adduser -S nextjs -G nodejs
# standalone output + the assets it doesn't trace (static + public).
COPY --from=builder /app/.next/standalone ./
COPY --from=builder /app/.next/static ./.next/static
COPY --from=builder /app/public ./public
USER nextjs
EXPOSE 3000
ENV PORT=3000 HOSTNAME=0.0.0.0
CMD ["node", "server.js"] # the standalone entrypoint, not `next start`
```

### Behind a reverse proxy

Self-hosted, you'll run Nginx/Caddy/Traefik in front for TLS termination, gzip/brotli, and load balancing. Things to get right:

- **Forward the real client info:** `X-Forwarded-For`, `X-Forwarded-Proto`, `Host`. Next reads these for `headers()`, redirects, and the Server Action origin check — get them wrong and CSRF protection or HTTPS redirects misbehave.
- **Don't let the proxy strip `Host`** in a way that breaks the Server Action origin check; configure `serverActions.allowedOrigins` accordingly (from the security lesson).
- **Serve `/_next/static` with long cache headers** — those assets are content-hashed and immutable.

### When to use what

Vercel for speed-to-market, small/medium teams, and Next-native features you'd rather not reimplement. Self-hosting when you need data residency, cost control at scale, an existing Kubernetes/cloud footprint, or features the platform won't give you. Many teams start on Vercel and move specific workloads to self-hosting as scale and cost dictate.

## Database Migrations in the Pipeline

Migrations are the riskiest part of any deploy because they're stateful and hard to undo. The rules:

- **Migrations run as a discrete pipeline step, not at app boot.** Running them on every container start races across replicas and couples migration to scaling. Run them once, before the new app version takes traffic.
- **Make them backward compatible (expand/contract).** During a rollout, old and new code run simultaneously. A migration that drops a column the old code still reads breaks the old replicas mid-deploy.

```text
Expand/contract for a column rename (zero-downtime):
1. EXPAND   : add new column, backfill, dual-write from new code. Old code unaffected.
2. MIGRATE  : deploy code that reads/writes the new column.
3. CONTRACT : once no code references the old column, drop it in a later deploy.
```

```yaml
# deploy job step — migrate before promoting the app
- name: Run migrations
  run: npx prisma migrate deploy # applies pending migrations idempotently, no prompts
  env: { DATABASE_URL: ${{ secrets.DATABASE_URL }} }
```

`migrate deploy` (vs `migrate dev`) is non-interactive and only applies committed migrations — the correct command for CI/CD. Never auto-generate migrations in the pipeline.

## Feature Flags and Safe Rollouts

**Deploy is not release.** A feature flag lets you ship code to production *dark* and turn it on independently — decoupling the risky moment (enabling the feature) from the deploy:

```tsx
// app/checkout/page.tsx
import { getFlag } from "@/lib/flags";

export default async function Checkout() {
  // Evaluate per-user/segment. Roll out to 5%, watch, then ramp.
  const newFlow = await getFlag("new-checkout", { /* user/segment context */ });
  return newFlow ? <NewCheckout /> : <LegacyCheckout />;
}
```

This enables **safe rollout patterns**:

- **Percentage / canary rollout:** enable for 1% → 10% → 50% → 100%, watching error rates and SLO burn (from the observability lesson) at each step. Catch regressions at 1% of traffic, not 100%.
- **Kill switch:** a flag you flip *off* instantly when an alert fires — no redeploy, no rollback latency.
- **Segment targeting:** internal users first, then beta, then GA.

Flags carry a cost: every flag is a branch in your code and a combinatorial test burden. Treat them as temporary — once a feature is fully rolled out, **remove the flag and the dead branch**. Long-lived flags rot into permanent complexity.

## Rollback Strategy

When (not if) a bad deploy ships, the only thing that matters is how fast you recover. Plan it before you need it:

- **Prefer instant rollback over forward-fix under pressure.** Vercel: promote the previous deployment (an atomic, immutable artifact) — seconds. Self-hosted: redeploy the previous image tag (which is why you tag images immutably and keep the last N). Both are faster and safer than writing a hotfix while production burns.
- **The flag kill switch is the fastest rollback** for feature-level issues — toggle off, no deploy at all.
- **Migrations complicate rollback** — this is the catch. If you've already dropped a column, rolling back the *code* to a version that needs it fails. This is the real reason for expand/contract: it keeps the old code runnable, so a code rollback stays possible. Never make a destructive schema change in the same deploy as the code that depends on it.
- **Rehearse it.** A rollback procedure nobody has run is a rollback procedure that won't work at 3am. Practice promoting a previous deploy in staging.

### When to use what

Roll back (revert to the last good artifact) when the cause is unclear or the blast radius is large — restore service first, diagnose after. Forward-fix only when the bug is well understood and trivially patchable, *and* a rollback would itself be risky (e.g., a migration already ran). Flag kill switch when the problem is isolated to one flagged feature.

## Pitfalls

- **`npm install` instead of `npm ci` in CI** — ignores the lockfile and yields non-reproducible builds.
- **Skipping the `build` gate.** RSC and bundling errors surface nowhere else; a passing test suite with a broken build is a red herring.
- **Running migrations at app boot** — races across replicas and couples migration to scaling. Run them as a dedicated step.
- **Destructive migrations alongside dependent code** — breaks old replicas mid-rollout and makes code rollback impossible. Use expand/contract.
- **Differing `NEXT_PUBLIC_` values across stages while promoting one artifact** — they were inlined at build; the wrong values ship.
- **Committing secrets** or rebuilding per environment unnecessarily — leaks credentials and invites drift.
- **No rehearsed rollback.** The first time you roll back should not be during an incident.
- **Forgetting `X-Forwarded-*` headers** behind a self-hosted proxy — breaks HTTPS redirects and Server Action origin checks.

## Exercises

1. Write a GitHub Actions workflow with the four gates (typecheck, lint, test, build) plus a Postgres service for integration tests. Make them required checks on `main`.
2. Add a Zod-based `env.ts` that validates required variables at boot. Remove a required var and confirm the deploy/build fails loudly instead of crashing at runtime.
3. Build the standalone Docker image with the multi-stage Dockerfile, run it locally, and compare its size to a naive image that copies all of `node_modules`.
4. Perform a zero-downtime column rename using expand/contract across three deploys. At each step, confirm both old and new code paths work.
5. Add a feature flag to one feature, roll it out to a 10% segment, then practice flipping the kill switch and (separately) promoting the previous deployment as a rollback. Time both.

## Key Takeaways

- CI should make broken code unmergeable: gate on typecheck → lint → test → build, ordered cheapest-first, against a real test DB, as required checks.
- Per-PR preview deployments turn reviews into hands-on testing and catch production-build bugs early.
- Promote one immutable artifact through preview → staging → prod; scope env vars per stage and remember `NEXT_PUBLIC_` is baked in at build time.
- Vercel trades cost for near-zero ops; self-hosting with `output: 'standalone'` and a multi-stage Dockerfile gives control behind a properly configured reverse proxy.
- Run migrations as a discrete, backward-compatible (expand/contract) step — never at boot, never destructive alongside dependent code.
- Decouple deploy from release with feature flags for canary rollouts and kill switches, and rehearse an instant rollback before you need it.

---

[← Previous: Testing Strategy for Next.js](04-testing-strategy.md) | [Back to Course Index](../README.md) | [Next: Internationalization →](../06-senior-topics/01-internationalization.md)
