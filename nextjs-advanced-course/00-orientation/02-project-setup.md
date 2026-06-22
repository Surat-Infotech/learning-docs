# Production-Grade Project Setup

A toy project tolerates any structure. A codebase that ten engineers touch for three years does not. The conventions you set in week one — where code lives, how strict the types are, how env vars are validated — compound. This lesson is about making those decisions deliberately so the project scales with the team, not against it.

## What You'll Learn

- A scalable project structure built around features/domains, not file types.
- How `app/` routing colocation (`_components`, route groups) coexists with shared `lib/` and `components/`.
- A tooling baseline: TypeScript strict mode, ESLint, Prettier, path aliases.
- Type-safe environment variable management and why `NEXT_PUBLIC_` is a footgun.
- `next.config` essentials worth setting on day one.
- When a monorepo (Turborepo) earns its complexity — and when it doesn't.

## Structure: Organize by Domain, Not by Type

The default instinct is to group by *kind*: all components in `components/`, all hooks in `hooks/`, all utils in `utils/`. This scales poorly — a single feature ends up scattered across the tree, and you scroll through 200 files to find the three that belong together.

Organize by **feature/domain** instead, and let the `app/` directory hold routing concerns only.

```text
src/
  app/                      # routing layer ONLY — pages, layouts, route handlers
    (marketing)/            # route group: shared layout, no URL segment
      page.tsx
      layout.tsx
    (app)/                  # authenticated app shell
      dashboard/
        page.tsx
        _components/        # route-private components (underscore = not a route)
          metric-chart.tsx
      layout.tsx
    api/
      webhooks/route.ts
  features/                 # domain logic, grouped by business capability
    billing/
      components/
      server/               # server-only: queries, actions
      schemas.ts            # Zod schemas
      types.ts
    auth/
    metrics/
  components/               # truly shared, generic UI (Button, Modal, Card)
    ui/
  lib/                      # cross-cutting infra (db client, logger, env)
    db.ts
    env.ts
    utils.ts
```

The rules of thumb:

- **`app/` is routing.** Pages, layouts, loading/error boundaries, route handlers. Anything used by exactly one route lives in that route's `_components` (the underscore prefix makes the folder non-routable).
- **`features/` is your domain.** Each folder is a business capability with its own components, server logic, and schemas. This is where most code lives and most changes happen.
- **`components/` is shared UI** — generic, domain-agnostic primitives. If a component knows about "invoices," it belongs in `features/billing`, not here.
- **`lib/` is infrastructure** — the database client, logging, the env loader, generic utilities.

### Route groups and colocation

Two App Router features make this clean:

- **Route groups** `(name)` let you share a layout across routes without adding a URL segment. Use them to separate, say, marketing pages (one layout, public) from the app (another layout, authed).
- **Underscore folders** `_components` / `_lib` opt out of routing entirely. Colocate route-private code next to the route that uses it; promote to `features/` only when a second consumer appears.

### Enforcing server-only code

Mark modules that must never reach the client so a stray import fails loudly at build time:

```ts
// src/features/billing/server/queries.ts
import "server-only"; // throws if this module is bundled into client code
import { db } from "@/lib/db";

export async function getInvoices(orgId: string) {
  return db.invoice.findMany({ where: { orgId } });
}
```

The complementary `import "client-only"` guards browser-only modules. These are cheap insurance against the boundary mistakes from the previous lesson.

## Tooling Baseline

Set these once; they pay off forever.

### TypeScript strict mode

Non-negotiable. Turn it on before the codebase grows, because retrofitting strictness into thousands of lines is miserable.

```json
// tsconfig.json (excerpt)
{
  "compilerOptions": {
    "strict": true,
    "noUncheckedIndexedAccess": true, // arr[i] is T | undefined — catches real bugs
    "noImplicitOverride": true,
    "moduleResolution": "bundler",
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"] // absolute imports — no more ../../../
    }
  }
}
```

`noUncheckedIndexedAccess` in particular catches a whole class of "it's undefined in prod" bugs. The `@/*` alias kills relative-path spaghetti.

### ESLint + Prettier

Let ESLint own correctness and Prettier own formatting; don't make them fight. Extend `next/core-web-vitals`, add `next/typescript`, and run Prettier separately (or via `eslint-config-prettier` to disable conflicting style rules).

```bash
# package.json scripts worth standardizing across the team
# "lint": "next lint"
# "typecheck": "tsc --noEmit"
# "format": "prettier --write ."
```

Wire `typecheck` and `lint` into CI and a pre-commit hook (e.g., lint-staged + husky). A type error should never reach `main`.

## Environment Variables, Done Right

Next.js exposes env vars based on a prefix, and this is a frequent security incident:

- **No prefix** → server-only. `DATABASE_URL`, `STRIPE_SECRET_KEY`. Never reaches the browser.
- **`NEXT_PUBLIC_` prefix** → inlined into the client bundle at build time. **Visible to anyone.** Only for genuinely public values (a public analytics key, an API base URL).

The footgun: prefixing a secret with `NEXT_PUBLIC_` to "make it work" ships it to every visitor. Treat the prefix as a public-disclosure decision.

Validate env at startup so a missing or malformed var fails immediately, not at 2 a.m. in a code path nobody tested:

```ts
// src/lib/env.ts
import { z } from "zod";

const serverSchema = z.object({
  DATABASE_URL: z.string().url(),
  STRIPE_SECRET_KEY: z.string().min(1),
  NODE_ENV: z.enum(["development", "test", "production"]),
});

// Parse once at module load — crashes the build/boot on misconfig.
export const env = serverSchema.parse(process.env);
```

Keep a committed `.env.example` documenting every variable (with placeholder values) so onboarding is "copy and fill in," not "grep the code for `process.env`."

## `next.config` Essentials

Start lean, but these are worth knowing on day one:

```ts
// next.config.ts
import type { NextConfig } from "next";

const config: NextConfig = {
  reactStrictMode: true,
  // Fail builds on type/lint errors instead of silently ignoring them.
  typescript: { ignoreBuildErrors: false },
  eslint: { ignoreDuringBuilds: false },
  images: {
    // Whitelist remote image hosts explicitly — required by next/image.
    remotePatterns: [{ protocol: "https", hostname: "cdn.example.com" }],
  },
  experimental: {
    // Opt-in to Partial Prerendering (covered in Module 1).
    // ppr: "incremental",
  },
};

export default config;
```

Resist the urge to disable build-time type/lint checks "to ship faster" — that just moves the failure to production.

## Monorepo Awareness: When to Reach for Turborepo

A monorepo (multiple packages/apps in one repo, orchestrated by a tool like Turborepo) is powerful and overkill for most single-app projects.

### Trade-offs: monorepo vs single app

**Reach for a monorepo when:**

- You have **multiple deployable apps** sharing real code — a web app, an admin dashboard, a marketing site, a mobile companion — and want one source of truth for shared UI and types.
- You want **shared packages** (`@acme/ui`, `@acme/config`, `@acme/db`) versioned and built together, with Turborepo caching builds so CI doesn't rebuild unchanged packages.
- A team is large enough that **build caching and task orchestration** save meaningful time.

**Stay with a single app when:**

- You have one Next.js app. Folder structure (`features/`) gives you modularity without the tooling overhead.
- Your team is small and the extra config, workspace resolution, and CI complexity would cost more than it saves.

The honest default: **start single-app.** Splitting into a monorepo later is a known, mechanical refactor. Adopting one prematurely buys you complexity before you have the problem it solves.

## Pitfalls

- **Organizing by file type at scale.** `components/` with 300 files means every feature is scattered. Group by domain.
- **Skipping TypeScript strict mode early.** Retrofitting strictness onto a mature codebase is far more painful than starting strict.
- **Prefixing secrets with `NEXT_PUBLIC_`.** It inlines them into the client bundle for the world to see. Audit every public-prefixed var.
- **Not validating env vars.** A missing `DATABASE_URL` should crash the boot, not surface as a cryptic error in one untested branch.
- **Disabling `ignoreBuildErrors`/`ignoreDuringBuilds`.** It feels fast and silently lets broken code ship. Fix the errors instead.
- **Adopting a monorepo for one app.** You pay the orchestration tax without the multi-app benefit.

## Exercises

1. **Design a structure.** For a SaaS app with marketing pages, an authenticated dashboard, and a public API, sketch the `src/` tree using route groups, `_components`, `features/`, and `lib/`. Justify where each piece lives.
2. **Audit env vars.** Take a real or sample app and list every env var. Classify each as server-only or public, and flag any secret that's currently public-prefixed.
3. **Write the env schema.** Implement a `lib/env.ts` Zod schema for an app using a database, an auth provider, and a payment provider. Decide which vars are server vs public.
4. **Boundary enforcement.** Identify three modules that should be server-only and add `import "server-only"`. Then deliberately import one into a Client Component and observe the build failure.
5. **Monorepo decision memo.** Write a one-paragraph recommendation for whether a given project (you choose the scenario) should adopt Turborepo, citing the specific trade-offs that decide it.

## Key Takeaways

- Organize by domain (`features/`) and keep `app/` for routing only; colocate route-private code in `_components`.
- Turn on TypeScript strict mode and absolute imports from day one — retrofitting is expensive.
- Treat `NEXT_PUBLIC_` as a public-disclosure decision and validate all env vars at startup with Zod.
- Use `import "server-only"` / `"client-only"` to make boundary mistakes fail at build time.
- Default to a single app; adopt a monorepo only when multiple apps genuinely share code and build caching pays off.

---

[← Previous: Mental Models for Advanced Next.js](01-mental-models.md) | [Back to Course Index](../README.md) | [Next: The RSC Architecture in Depth →](../01-deep-rendering/01-rsc-architecture.md)
