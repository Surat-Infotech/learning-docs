# Advanced Next.js Course — 10-Week Weekly Roadmap (12 hours/week)

This roadmap turns the [Advanced Next.js Course](README.md) into a focused weekly
plan for working developers.

- **Time budget:** 12 hours per week
- **Total length:** 10 weeks (~120 hours)
- **Split:** About **30% reading**, **50% applying to a real codebase**, and
  **20% measuring/profiling and writing up trade-offs**.

> **Prerequisite:** Comfort with the App Router, Server/Client Components, routing,
> data fetching, and Server Actions (the [Next.js Course](../nextjs-course/README.md)).

> **How to read this plan.** This course is best done against a **real, evolving
> project**. Each week you study lessons, apply them to your app, and write a short
> "decision log" entry explaining the trade-offs you chose. The capstone in Week 10
> ties it all together.

**Weekly rhythm (suggested):** 2–3 deep sessions of ~4 hours. Protect time to
profile and measure — judgement comes from seeing real numbers.

---

## Week-at-a-Glance

| Week | Focus                          | Lessons                          |
| ---- | ------------------------------ | -------------------------------- |
| 1    | Mental Models + Foundation     | Module 0                         |
| 2    | Deep Rendering & RSC           | Module 1                         |
| 3    | Caching Layers & Revalidation  | Module 2 (1–2)                   |
| 4    | Data Fetching & the DAL        | Module 2 (3–4)                   |
| 5    | Architecture & Composition     | Module 3 (1–3)                   |
| 6    | Types & Resilience             | Module 3 (4–5)                   |
| 7    | Performance                    | Module 4 (1–3)                   |
| 8    | Edge, Scaling & Auth           | Module 4 (4–5) + Module 5 (1)    |
| 9    | Security, Observability, Tests | Module 5 (2–5)                   |
| 10   | Senior Topics + Capstone       | Module 6                         |

---

## Week 1 — Mental Models and Foundation

**Lessons (Module 0):**

- [Mental Models for Advanced Next.js](00-orientation/01-mental-models.md)
- [Production-Grade Project Setup](00-orientation/02-project-setup.md)

**Apply:** Audit a current project's structure and tooling. Reorganize toward
feature/domain folders, enable TS strict mode, and set path aliases. Write a one-page
"architecture intent" note.

---

## Week 2 — Deep Rendering and React Server Components

**Lessons (Module 1):**

- [The RSC Architecture in Depth](01-deep-rendering/01-rsc-architecture.md)
- [Rendering Strategies and Partial Prerendering](01-deep-rendering/02-rendering-strategies.md)
- [Streaming and Suspense Orchestration](01-deep-rendering/03-streaming-orchestration.md)
- [Server Actions in Depth](01-deep-rendering/04-server-actions-in-depth.md)

**Apply:** Map each route in your app to a rendering strategy and justify it. Add a
deliberate Suspense boundary and harden one Server Action (auth + Zod validation).

---

## Week 3 — Caching Layers and Revalidation

**Lessons (Module 2, lessons 1–2):**

- [The Next.js Caching Layers](02-caching-and-data/01-caching-layers.md)
- [Revalidation and Invalidation Strategies](02-caching-and-data/02-revalidation-strategies.md)

**Apply:** Diagram how a key request flows through the four caches. Design a cache
**tagging scheme** for one data model and wire `revalidateTag` into its mutations.

---

## Week 4 — Data Fetching and the Data Access Layer

**Lessons (Module 2, lessons 3–4):**

- [Advanced Data Fetching Patterns](02-caching-and-data/03-data-fetching-patterns.md)
- [Designing a Data Access Layer (DAL)](02-caching-and-data/04-data-access-layer.md)

**Apply:** Find and fix a request waterfall (measure the before/after). Extract data
access into a `lib/dal/` with `server-only` and in-DAL authorization checks.

---

## Week 5 — Architecture and Composition

**Lessons (Module 3, lessons 1–3):**

- [Project Architecture at Scale](03-architecture/01-project-architecture.md)
- [Component Architecture and Composition](03-architecture/02-component-architecture.md)
- [State Management at Scale](03-architecture/03-state-management.md)

**Apply:** Refactor a subtree that is needlessly client-side using the "server
children into client wrapper" pattern. Classify your app's state into the four
categories and move some to the URL.

---

## Week 6 — Types and Resilience

**Lessons (Module 3, lessons 4–5):**

- [End-to-End Type Safety](03-architecture/04-type-safety.md)
- [Error Handling and Resilience Patterns](03-architecture/05-error-handling-resilience.md)

**Apply:** Add Zod parsing at every external boundary (forms, params, env, API
responses). Convert one throwing Server Action to a typed Result return and add an
appropriately scoped error boundary.

---

## Week 7 — Performance

**Lessons (Module 4, lessons 1–3):**

- [Core Web Vitals and Performance Budgets](04-performance-and-scaling/01-web-vitals.md)
- [Bundle Optimization and Reducing Client JS](04-performance-and-scaling/02-bundle-optimization.md)
- [Asset Optimization at Scale](04-performance-and-scaling/03-asset-optimization.md)

**Apply:** Run the bundle analyzer, set a budget, and cut client JS (lazy-load or
move-to-server one heavy dependency). Audit images/fonts and fix any layout shift.
Record LCP/CLS/INP before and after.

---

## Week 8 — Edge, Scaling, and Auth

**Lessons (Module 4, lessons 4–5 + Module 5, lesson 1):**

- [Edge Runtime, Middleware, and Personalization](04-performance-and-scaling/04-edge-and-middleware.md)
- [Scaling and Infrastructure Considerations](04-performance-and-scaling/05-scaling-infrastructure.md)
- [Authentication and Authorization Patterns](05-production-engineering/01-auth-patterns.md)

**Apply:** Review what your middleware does and move any real authorization into the
DAL ("verify close to the data"). Document your scaling/infra assumptions (DB
pooling, regions, cache storage).

---

## Week 9 — Security, Observability, and Testing

**Lessons (Module 5, lessons 2–5):**

- [Security Hardening](05-production-engineering/02-security.md)
- [Observability: Logging, Tracing, and Monitoring](05-production-engineering/03-observability.md)
- [Testing Strategy for Next.js](05-production-engineering/04-testing-strategy.md)
- [CI/CD and Deployment Strategies](05-production-engineering/05-cicd-deployment.md)

**Apply:** Run a security pass (secrets, headers/CSP, validate Server Actions). Wire
up error monitoring and structured logging. Add a CI pipeline (typecheck, lint,
test, build) with preview deployments.

---

## Week 10 — Senior Topics and Capstone

**Lessons (Module 6):**

- [Internationalization (i18n)](06-senior-topics/01-internationalization.md)
- [Caching, CDN, and Multi-Region Delivery](06-senior-topics/02-cdn-multiregion.md)
- [Migration and Incremental Adoption](06-senior-topics/03-migration.md)
- [Design Systems and Component Libraries in Next.js](06-senior-topics/04-design-systems.md)
- [Leading a Next.js Codebase](06-senior-topics/05-leading-a-codebase.md)

**Capstone (the rest of the week and beyond):**

Architect and build a production-grade Next.js application that applies the whole
course: a secure DAL with authorization, a deliberate caching/revalidation scheme,
end-to-end types, performance budgets that pass in CI, observability, i18n, and a
documented deployment with rollback. Write an ADR for each major decision.

✅ **Course complete:** You can architect, optimize, secure, scale, and lead a
production Next.js codebase.

---

## Readiness Checkpoints

- [ ] **End of Week 2:** Explain the RSC payload and choose a rendering strategy per route.
- [ ] **End of Week 4:** Design caching/tagging and a secure data access layer.
- [ ] **End of Week 6:** Enforce end-to-end types and resilient error handling.
- [ ] **End of Week 7:** Set and meet a performance budget with real measurements.
- [ ] **End of Week 9:** Harden security, add observability, and ship via CI/CD.
- [ ] **End of Week 10:** Deliver the capstone with documented architectural decisions.

---

[← Back to Course Index](README.md)
