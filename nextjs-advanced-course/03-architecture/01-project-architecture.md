# Project Architecture at Scale

A small Next.js app survives almost any folder layout. A large one does not. As headcount grows and the codebase passes a few hundred files, the cost of a bad structure shows up as merge conflicts, accidental coupling, slow onboarding, and "where does this go?" debates in every PR. Architecture is the set of decisions that keeps those costs flat as the app grows. This lesson is about making those decisions deliberately instead of letting them accrete.

## What You'll Learn

- Why type-based folder layouts collapse at scale and how feature/domain organization fixes it
- The modular monolith as a default, and the seams that make it splittable later
- Where the `app/` directory's responsibility ends and `features/` + `lib/` begin
- How to decide between colocation and shared placement for any given module
- Enforcing architectural boundaries with lint rules and import constraints
- When (and when not) to graduate to a Turborepo monorepo with shared packages
- A concrete directory tree for a large production app

## Type-Based vs Feature-Based Organization

Most tutorials teach type-based grouping: every component in `components/`, every hook in `hooks/`, every util in `utils/`. It reads cleanly with ten files. At three hundred files, `components/` is an undifferentiated bin, and a single feature's code is smeared across six top-level folders. Touching "checkout" means editing `components/`, `hooks/`, `lib/`, `types/`, and `services/` simultaneously.

Feature-based (domain-based) organization groups by *what the code is about*, not *what kind of artifact it is*:

```text
features/
  checkout/
    components/      // CheckoutForm, OrderSummary
    hooks/           // useCart, usePromoCode
    actions.ts       // server actions for this domain
    queries.ts       // data access for this domain
    schema.ts        // zod schemas + inferred types
    index.ts         // public surface of the feature
```

The win is *locality of change*. A feature is a unit you can reason about, review, test, and delete as a whole. The folder boundary doubles as a dependency boundary if you enforce it (more below).

### Trade-offs

Type-based isn't wrong everywhere. Genuinely cross-cutting primitives — a `Button`, a `formatCurrency`, a `db` client — have no single owning feature and belong in shared layers. The rule of thumb: **group by feature until a thing is used by three or more features, then promote it to shared.** Promoting too early creates a `shared/` folder that becomes its own dumping ground; promoting too late means copy-paste drift. Bias toward duplication first, abstraction second — premature shared abstractions are harder to undo than duplication.

## The Modular Monolith

For almost every team that isn't FAANG-scale, the right starting architecture is a *modular monolith*: one deployable Next.js app, internally divided into modules with explicit boundaries. You get the operational simplicity of a single deploy (one CI pipeline, one set of env vars, atomic refactors across modules) without the distributed-systems tax of microservices or premature package splitting.

The discipline that makes it "modular" rather than a "big ball of mud" is that modules talk through **defined interfaces**, not by reaching into each other's internals:

```ts
// features/checkout/index.ts
// The ONLY things other features may import from checkout.
export { CheckoutButton } from "./components/CheckoutButton";
export { getCartTotal } from "./queries";
export type { Cart } from "./schema";

// Internals (./queries.ts internals, ./components/OrderSummary) are private.
```

Other features import from `features/checkout` (the barrel), never from `features/checkout/queries`. That single rule is what lets you later extract `checkout` into its own package or service with a known, small surface area to rewire.

### Trade-offs

Barrel files (`index.ts`) have a real cost: they can defeat tree-shaking and balloon bundles if a client component imports a barrel that transitively pulls in server-only code. Keep server and client surfaces in separate barrels (or skip the barrel for client code and import the leaf directly). Don't barrel everything reflexively — barrel the *public* API, import internals by path.

## Where `app/` Ends and `features/` Begins

The single most clarifying rule for an App Router codebase: **`app/` is for routing, not for logic.** A route file's job is to map a URL to a composition of feature code, handle the request/response concerns of that URL (params, search params, metadata, caching directives), and nothing else.

```tsx
// app/(shop)/checkout/page.tsx
// Thin: parse the route, delegate to the feature.
import { CheckoutScreen } from "@/features/checkout";
import { getCart } from "@/features/checkout/queries";

export const metadata = { title: "Checkout" };

export default async function CheckoutPage() {
  const cart = await getCart(); // feature owns the data access
  return <CheckoutScreen cart={cart} />;
}
```

The page is ten lines. All substance — UI, validation, business rules, queries — lives in `features/checkout`. This separation means a feature can be tested without a router, reused across multiple routes (e.g. `/checkout` and a `/quick-buy` modal), and moved without rewriting URLs.

`lib/` sits below features: it's the shared, feature-agnostic foundation. The `db` client, the configured `fetch` wrapper, auth helpers, the logger, generic utilities. Nothing in `lib/` should import from `features/` — dependencies point downward only.

```text
app/        → routes, layouts, route handlers (thin, no business logic)
features/   → domain modules (the bulk of the app)
lib/        → shared infra: db, auth, logger, fetch, env (no feature imports)
components/ → shared UI primitives only (design-system level)
```

### Trade-offs

The discipline costs a little boilerplate: a one-line page delegating to a feature can feel like ceremony. It pays off the first time a designer asks to surface checkout in three places, or the first time you split the repo. If a route is genuinely a one-off with no reuse and no logic, inlining is fine — don't manufacture a feature for a static marketing page.

## Colocation vs Shared

For any module, ask: *who uses this?* If one feature uses it, colocate it inside that feature. If several do, it's a candidate for `lib/` or `components/`. Colocation is the default because it keeps the radius of change small and makes deletion safe — when you remove a feature, everything it owned goes with it, with no orphaned shared files left behind.

A useful heuristic chain:

1. Used in one file → keep it in that file.
2. Used across one feature → feature-local module.
3. Used across 3+ features → promote to shared (`lib/` or `components/`).
4. Used across 3+ *apps* → promote to a package (monorepo).

Each promotion is a one-way-ish door; resist climbing the ladder speculatively.

## Enforcing Boundaries

Conventions that aren't enforced rot. Lint rules turn architecture into something CI checks. Use `eslint-plugin-boundaries` or `eslint-plugin-import`'s `no-restricted-paths` to encode the dependency direction:

```js
// eslint.config.js (flat config excerpt)
// Encode: lib must not import features; features import each other
// only via their public barrel.
import boundaries from "eslint-plugin-boundaries";

export default [
  {
    plugins: { boundaries },
    settings: {
      "boundaries/elements": [
        { type: "app", pattern: "app/*" },
        { type: "feature", pattern: "features/*" },
        { type: "lib", pattern: "lib/*" },
        { type: "shared-ui", pattern: "components/*" },
      ],
    },
    rules: {
      "boundaries/element-types": [
        "error",
        {
          default: "disallow",
          rules: [
            // app can use anything
            { from: "app", allow: ["feature", "lib", "shared-ui"] },
            // features use lib/ui freely, and each other (barrel-checked separately)
            { from: "feature", allow: ["lib", "shared-ui", "feature"] },
            // lib is the floor: it depends on nothing internal
            { from: "lib", allow: ["lib"] },
            { from: "shared-ui", allow: ["shared-ui", "lib"] },
          ],
        },
      ],
    },
  },
];
```

Pair this with `no-restricted-imports` to ban deep imports into a feature's internals (`@/features/*/queries`), forcing everyone through the barrel. Add `import/no-cycle` to catch circular dependencies, which are the canary for a missing boundary.

### Trade-offs

Strict boundary linting generates friction and the occasional false positive; teams sometimes disable it under deadline pressure and never re-enable it. Make the rules *errors*, not warnings (warnings are ignored), but keep the rule set small enough that violations are real signals. The goal is to catch the cross-module reach that you'd otherwise only find in code review months later.

## When to Split into a Monorepo

A monorepo (Turborepo) is the right move when you have **multiple deployable apps sharing real code** — a customer web app, an admin dashboard, a marketing site, plus a shared design system and shared API client. The signal is not "the repo is big"; it's "we are copy-pasting between repos" or "we want to deploy these independently but share a UI library."

```text
apps/
  web/                 // the main Next.js app
  admin/               // separate Next.js app, own deploy
  marketing/
packages/
  ui/                  // shared design system
  db/                  // prisma client + schema, shared
  config/              // eslint/tsconfig/tailwind presets
  api-client/          // typed client shared by apps
turbo.json             // task pipeline + caching
```

Turborepo's value is the *task graph and remote caching*: it builds only what changed and reuses cached outputs across machines and CI, which is what keeps build times sane once you have several apps. The shared `packages/config` deduplicates the dozen config files that otherwise drift between apps.

### Trade-offs

A monorepo adds genuine overhead: workspace tooling, package versioning decisions, more complex CI, and a learning curve for contributors. **Do not start here.** A single Next.js app with internal `features/` modules carries you remarkably far, and the modular-monolith discipline above is exactly what makes the eventual split cheap — well-bounded features become packages with minimal rewiring. Split when the second deployable app appears, not in anticipation of it.

## Pitfalls

- **Business logic in `page.tsx`/route handlers.** Routes become un-reusable and un-testable. Keep them thin; push logic into features.
- **A `shared/` or `utils/` junk drawer.** Everything ends up "shared." Force promotion to require 3+ consumers and a real name.
- **Barrels that mix server and client exports.** A client component importing the barrel drags server-only code (or `server-only`-poisoned modules) into the client bundle. Split barrels by environment.
- **Deep imports bypassing the barrel.** Without a lint rule, people import `features/x/internal/thing` and your boundary evaporates. Ban it explicitly.
- **Circular dependencies between features.** A sign two features are really one, or that a shared concept belongs in `lib/`. Treat `import/no-cycle` errors as architecture bugs.
- **Premature monorepo.** You inherit all the complexity and none of the benefit until you have a second app.

## Exercises

1. Take an existing type-based app (or sketch one with `components/`, `hooks/`, `utils/`) and design the feature-based equivalent. Identify which current `utils/` belong in `lib/` vs a specific feature.
2. Define the public barrel (`index.ts`) for a `billing` feature with 15 internal files. Decide exactly which exports are public and justify each. Write the `no-restricted-imports` rule that protects the rest.
3. Write the `eslint-plugin-boundaries` config for an app with `app/`, `features/`, `lib/`, and `components/`, encoding the downward-only dependency direction. Add a rule that forbids `features` from importing `app`.
4. You're asked to add an internal admin tool. Decide: new route group in the same app, or new app in a monorepo? Write the one-paragraph decision with the trigger conditions that would flip your answer.
5. Design the `packages/` layout for a 3-app monorepo and name the Turborepo pipeline tasks (`build`, `lint`, `test`, `typecheck`) with their dependency edges in `turbo.json`.

## Key Takeaways

- Organize by feature/domain, not by artifact type; promote to shared only at 3+ consumers.
- The modular monolith is the right default: one deploy, explicit module boundaries, splittable later.
- `app/` is routing only; `features/` holds logic; `lib/` is shared infra and depends on nothing internal.
- Colocate by default; the smaller the change radius, the safer deletion and refactoring become.
- Enforce dependency direction with lint errors, not conventions — boundaries that aren't checked rot.
- Reach for a Turborepo monorepo only when a second deployable app shares real code; well-bounded features make that split cheap.

---

[← Previous: Designing a Data Access Layer](../02-caching-and-data/04-data-access-layer.md) | [Back to Course Index](../README.md) | [Next: Component Architecture and Composition →](02-component-architecture.md)
