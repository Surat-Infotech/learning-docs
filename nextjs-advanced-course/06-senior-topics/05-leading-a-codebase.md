# Leading a Next.js Codebase

Writing great Next.js code is table stakes for senior. Leading a Next.js codebase is a different job: you're now responsible for the decisions other people will make at 4pm on a Friday, the conventions that keep a fifteen-person team from building fifteen different apps, and the upgrade that could break production for everyone. Your leverage shifts from what you build to what you make *easy to build correctly*. This final lesson is about that shift — and it ends with a capstone that pulls the entire course together.

## What You'll Learn

- Establishing conventions and a written, living style guide
- Enforcing architecture with lint rules and CI so standards don't depend on vigilance
- Code-review focus areas specific to Next.js: client/server boundaries, caching correctness, security, bundle size
- Documenting decisions with ADRs so the *why* survives team turnover
- Onboarding and mentoring engineers into the RSC mental model
- Managing Next.js version upgrades and breaking changes safely
- A tech-debt strategy that pays down without stalling delivery

## Conventions and a written style guide

Undocumented conventions live in the heads of two senior engineers and die when they leave. Write them down. A Next.js style guide should be opinionated and specific, not generic React advice.

Cover at minimum:

- **File and folder structure.** Where do route segments, shared components, server-only code, and `lib` utilities live? Colocation rules. How features are organized.
- **Server vs client default.** State the default explicitly: "components are Server Components unless they need interactivity; push `"use client"` to leaves." This single rule prevents the most common architectural drift.
- **Data fetching.** Where fetching happens (Server Components / Route Handlers), caching defaults, when client fetching is allowed.
- **Server Actions vs Route Handlers.** When to use each, validation requirements, auth requirements.
- **Naming, error handling, loading states, metadata conventions.**

Keep it in-repo (`/docs` or `CONTRIBUTING.md`) so it versions with the code, and treat it as living — every recurring review comment is a candidate for a new rule. A convention you have to re-explain in review is a convention that belongs in the guide.

## Enforcing architecture in CI

Anything enforced by human attention will eventually be forgotten. Encode standards as automation so the machine catches drift and reviewers focus on judgement.

```js
// eslint.config.mjs (flat config) — illustrative
import nextPlugin from "@next/eslint-plugin-next";
import boundaries from "eslint-plugin-boundaries";

export default [
  {
    plugins: { "@next/next": nextPlugin, boundaries },
    rules: {
      // Catch Next-specific mistakes.
      "@next/next/no-img-element": "error",
      "@next/next/no-html-link-for-pages": "error",
      // Enforce architectural layering: UI cannot import from the data layer
      // directly, server-only code cannot be imported into client code, etc.
      "boundaries/element-types": ["error", { default: "disallow", rules: [/* ... */] }],
    },
  },
];
```

A CI pipeline that protects a Next codebase typically runs:

```text
ci:
  - typecheck          (tsc --noEmit; strict mode on)
  - lint               (next lint + a11y + boundary rules)
  - test               (unit + integration)
  - build              (next build — catches RSC/serialization errors)
  - bundle-size check   (fail PRs that exceed a budget)
  - import guards       ("server-only" / "client-only" packages enforce the boundary)
```

Two underused enforcers worth highlighting:

- **The `server-only` and `client-only` packages.** Importing `server-only` into a module makes the build fail if that module ever ends up in a client bundle — a hard guarantee that secrets and server logic never leak to the browser. Use them on your data layer.
- **A bundle-size budget in CI.** Client JS creeps up one `"use client"` at a time. A per-route budget that fails the PR turns an invisible slow regression into a visible, blocking one.

## Code review focus areas (Next-specific)

Generic review advice applies, but a Next.js reviewer must specifically guard four things that juniors get wrong and that don't show up in tests:

1. **Client/server boundary.** Is `"use client"` on the smallest necessary unit, or did it land on a whole subtree? Did a secret or a heavy server-only dependency get imported into a client component? Is `children` used to keep server content inside client shells?
2. **Caching correctness.** Is the rendering mode (static/dynamic/ISR) intentional, or did a stray `cookies()`/`searchParams` read flip a page to dynamic? Are `revalidate`/`revalidateTag`/`cache` settings explicit and correct? Could a personalized response end up in a shared cache?
3. **Security.** Are Server Actions and Route Handlers validating input and checking authorization *on every call* (not relying on UI gating)? Are environment secrets server-only? Is user input sanitized? Auth checks belong in the data/action layer, never only in the component.
4. **Bundle size.** Did this PR pull a large client dependency into the bundle? Is a date/icon/util library being imported wholesale? Did an interactive feature get built server-first where possible?

Make these a literal checklist in your PR template so they're reviewed every time, not just when someone remembers.

## Documenting decisions with ADRs

The most expensive thing a team loses is the *reasoning* behind a decision. Six months later someone "fixes" a deliberate choice and reintroduces the bug it prevented. Architecture Decision Records capture the why, cheaply.

```text
docs/adr/0007-use-server-actions-for-mutations.md

# 7. Use Server Actions for mutations

## Status
Accepted — 2026-03-10

## Context
We had a mix of Route Handlers and client fetch for mutations, with
inconsistent validation and error handling.

## Decision
All mutations go through Server Actions with Zod validation and a shared
auth wrapper. Route Handlers are reserved for public/webhook/third-party APIs.

## Consequences
+ Consistent validation and auth; less client JS; progressive enhancement.
- Team must learn Server Action patterns; some edge cases need Route Handlers.
```

Keep ADRs short, numbered, in-repo, and immutable (supersede rather than edit). They make onboarding faster and stop relitigating settled decisions.

## Onboarding and mentoring

The single biggest onboarding hurdle for a Next.js team is the RSC mental model. Engineers arriving from a Pages Router or pure-SPA background reflexively reach for `useEffect` fetching and `"use client"`. Structure onboarding around unlearning that.

- **Pair on the first real feature** so the server-first default is taught in context, not just documented.
- **Give a "paved path" example feature** in the repo — one route that demonstrates the canonical data fetching, Server Action, caching, and component-boundary patterns. New engineers copy the shape.
- **Mentor toward judgement, not rules.** The goal isn't "memorize when to use `"use client"`"; it's understanding the cost model (bundle, hydration, data locality) so they make the call themselves.
- **Review generously early.** Use the first weeks of PRs to teach the boundaries and caching model; that investment compounds across everything they ship after.

## Managing version upgrades

Next.js moves fast, and major versions carry breaking changes (async `params`/`searchParams`, caching default changes, minimum React/Node versions). An upgrade is a project, not a `npm update`.

A disciplined upgrade process:

```text
1. Read the upgrade guide and changelog; list breaking changes that touch you.
2. Run the official codemod (`npx @next/codemod@latest upgrade`) on a branch.
3. Typecheck + build; fix what the codemod missed (often caching defaults).
4. Run the full test suite and exercise critical paths manually / in preview.
5. Deploy to a staging/preview environment; watch error rates and Web Vitals.
6. Roll out behind canary/percentage traffic if the platform allows.
7. Pin the version; only bump deliberately, never automatically in production.
```

Senior practices around this: stay one minor behind bleeding edge for production unless a fix demands otherwise; keep dependencies (especially React and any RSC-aware library) compatible *before* the major bump; and budget time for the upgrade explicitly rather than smuggling it into a feature branch where regressions can't be attributed.

## Tech-debt strategy

Debt isn't inherently bad — it's a loan. Leadership is managing the interest, not refusing to ever borrow.

- **Make it visible.** Track debt in the same backlog as features, with the cost-of-not-fixing articulated. Invisible debt never gets prioritized.
- **Pay it down incrementally, in the path of work.** The "boy scout rule": when you touch an area, leave it slightly better. This beats stop-the-world refactors that stall delivery and rarely finish.
- **Reserve a steady fraction of capacity** (a common figure is ~20%) for debt and maintenance so it's continuous, not a heroic quarterly purge.
- **Distinguish debt from architecture mistakes.** A messy component is debt; a wrong server/client boundary across the whole app is an architecture problem that needs a deliberate plan and probably an ADR.
- **Don't gold-plate.** Not every imperfection is worth fixing. Spend the budget where the interest — bug rate, slowed delivery, onboarding pain — is actually high.

### Trade-offs: when to use what

- **Lint/CI enforcement vs review convention** — Automate anything mechanical and objective (boundaries, bundle size, a11y). Reserve human review for judgement that a rule can't express. Don't burn reviewer attention on what a linter could catch.
- **Strict conventions vs team autonomy** — Be strict on the few things that cause systemic damage (boundaries, caching, security); be permissive on style preferences a formatter already handles. Over-constraining demotivates; under-constraining fragments the codebase.
- **Aggressive upgrades vs stability** — Track current for security and DX, but lag production slightly and upgrade deliberately. The cost of a bad major in production outweighs the cost of being a version behind.
- **Big refactor vs incremental cleanup** — Default to incremental. Reserve big refactors for genuine architecture errors with a clear, time-boxed plan.

## Pitfalls

- **Conventions that live only in people's heads.** They evaporate with turnover. Write them down, in-repo.
- **Relying on review vigilance for mechanical rules.** Humans forget; CI doesn't. Encode boundaries, bundle budgets, and `server-only` guards.
- **Reviewing only logic, not the Next-specific four** (boundaries, caching, security, bundle). Those are exactly what slips through tests.
- **Auto-upgrading Next in production.** Major versions break. Treat upgrades as projects with staging and canary rollout.
- **Letting tech debt stay invisible.** Untracked debt is never prioritized. Surface it with its cost.
- **Stop-the-world refactors.** They stall delivery and often don't finish. Pay down in the path of work.
- **Mentoring with rules instead of the cost model.** Engineers who understand *why* make better autonomous decisions than those who memorized a list.

## Exercises

1. Draft a one-page Next.js style guide for a team, with explicit defaults for server vs client, data fetching, and Server Actions vs Route Handlers. Add it to a repo as `CONTRIBUTING.md`.
2. Configure CI to enforce three things: a typecheck, a bundle-size budget that fails oversized PRs, and `server-only` guards on your data layer. Verify each fails on a deliberately bad PR.
3. Write a PR-review checklist covering the four Next-specific focus areas and apply it to a real or sample PR, documenting what it caught.
4. Write an ADR for a real architectural decision in your codebase (e.g. caching strategy, auth approach) using the Status/Context/Decision/Consequences format.
5. Write an upgrade runbook for moving your app one major Next.js version forward, listing the breaking changes that apply to you, the codemod step, and your rollout/rollback plan.

## Wrap-up: from senior to lead

You started this course knowing Next.js basics. You now understand rendering strategies and caching layers as a system, can reason about data architecture across regions, can migrate a legacy app without a rewrite, can build a design system that respects the RSC boundary, and can lead the people and process around a codebase. The throughline of senior engineering is judgement under trade-offs: there is rarely one right answer, only the right answer *for this constraint set*. Keep asking "what does this cost, and who pays it" — in latency, in bundle bytes, in complexity, in the next engineer's onboarding — and you'll make decisions the rest of the team can trust and build on.

Now prove it.

## Capstone Project

**Architect and build a production-grade Next.js application** that exercises everything in this course. The point is not feature count; it's defensible decisions.

**Build:** a multi-tenant SaaS dashboard (pick a domain — analytics, project management, a storefront admin) that is:

- **Internationalized** — `[locale]` routing with middleware detection, at least two locales including one RTL, localized metadata with `hreflang`, and `Intl`-based formatting. Most pages statically rendered per locale.
- **Multi-region aware** — a documented caching strategy across CDN and Next caches, a deliberate compute-vs-data placement decision, ISR for shared content, and a cached-shell-plus-dynamic-hole pattern for personalized views.
- **Auth-protected** — real authentication, authorization enforced in the data/action layer (not the UI), Server Actions with input validation, and `server-only` guards on secrets.
- **Observable** — structured logging, error tracking, and Core Web Vitals / performance monitoring wired up, with at least one meaningful alert.
- **Built on a design system** — a `packages/ui` library in a monorepo with server-compatible and client components, build-time styling, CSS-variable theming, and a11y verified in CI.
- **Led like a real codebase** — an in-repo style guide, CI enforcing typecheck + lint + boundaries + a bundle budget, at least three ADRs, and a written upgrade runbook.

**Deliverables:**

1. The running application, deployed.
2. An architecture document explaining your rendering, caching, region, and data-fetching decisions and their trade-offs.
3. The ADRs, style guide, and CI configuration.
4. A short retrospective: what you'd do differently, and where you deliberately took on debt and why.

Treat the architecture document and retrospective as the most important deliverables. Anyone can wire up features; a senior engineer can explain *why* each decision was the right one for the constraints — and that is exactly what this course was for. Go build it.

## Key Takeaways

- Your leverage as a lead is making correct decisions *easy*: written conventions, automated enforcement, and a paved path beat individual vigilance.
- Encode mechanical standards (boundaries, bundle budgets, `server-only` guards) in CI; reserve human review for the Next-specific judgement calls — boundaries, caching, security, bundle size.
- ADRs preserve the *why* and stop teams from relitigating or accidentally undoing deliberate decisions.
- Onboard around the RSC cost model, not a rule list; engineers who understand the trade-offs make better autonomous calls.
- Treat version upgrades as projects with codemods, staging, and canary rollout — never auto-upgrade production.
- Manage tech debt as visible, incrementally-paid interest, distinguishing routine debt from genuine architecture errors.

---

[← Previous: Design Systems and Component Libraries in Next.js](04-design-systems.md) | [Back to Course Index](../README.md)
