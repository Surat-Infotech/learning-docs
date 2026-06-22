# Advanced Next.js Course — For Junior and Senior Developers

This course picks up where the [Next.js Course](../nextjs-course/README.md) ends.
It is written for developers who can already build a Next.js app and now want to
**architect, optimize, secure, and scale** production systems — and lead the teams
that build them.

Whether you are a junior leveling up or a senior sharpening your judgement, each
lesson goes past "how" into **why, trade-offs, and what breaks at scale**. You
will find deep dives on the React Server Components architecture, the four caching
layers, data-access design, performance budgets, security, observability, and the
soft skills of leading a Next.js codebase.

> **Prerequisites:** You should be comfortable with the App Router, Server and
> Client Components, routing, basic data fetching, and Server Actions. If any of
> that is shaky, complete the [Next.js Course](../nextjs-course/README.md) first.

> **Want a schedule?** Follow the
> [10-Week Weekly Roadmap (12 hours/week)](WEEKLY-ROADMAP.md).

---

## How to Use This Course

1. **Read with a real codebase open.** Apply each idea to a project you maintain.
2. **Focus on trade-offs.** Every lesson has a *Trade-offs* discussion — that
   judgement is the real skill.
3. **Do the design exercises.** They are architectural, not drills — sketch
   solutions and defend your choices.
4. **Measure.** For performance and caching lessons, profile before and after.
5. **Build the capstone** (final lesson) to tie everything together.

---

## Course Map

### Module 0 — Orientation
Set the mental models and a production-grade foundation.

- [Mental Models for Advanced Next.js](00-orientation/01-mental-models.md)
- [Production-Grade Project Setup](00-orientation/02-project-setup.md)

### Module 1 — Deep Rendering & React Server Components
How rendering really works, and how to control it.

- [The RSC Architecture in Depth](01-deep-rendering/01-rsc-architecture.md)
- [Rendering Strategies and Partial Prerendering](01-deep-rendering/02-rendering-strategies.md)
- [Streaming and Suspense Orchestration](01-deep-rendering/03-streaming-orchestration.md)
- [Server Actions in Depth](01-deep-rendering/04-server-actions-in-depth.md)

### Module 2 — Caching & Data
The hardest part of Next.js, made clear.

- [The Next.js Caching Layers](02-caching-and-data/01-caching-layers.md)
- [Revalidation and Invalidation Strategies](02-caching-and-data/02-revalidation-strategies.md)
- [Advanced Data Fetching Patterns](02-caching-and-data/03-data-fetching-patterns.md)
- [Designing a Data Access Layer (DAL)](02-caching-and-data/04-data-access-layer.md)

### Module 3 — Architecture & Patterns
Structure that survives growth.

- [Project Architecture at Scale](03-architecture/01-project-architecture.md)
- [Component Architecture and Composition](03-architecture/02-component-architecture.md)
- [State Management at Scale](03-architecture/03-state-management.md)
- [End-to-End Type Safety](03-architecture/04-type-safety.md)
- [Error Handling and Resilience Patterns](03-architecture/05-error-handling-resilience.md)

### Module 4 — Performance & Scaling
Make it fast, and keep it fast as you grow.

- [Core Web Vitals and Performance Budgets](04-performance-and-scaling/01-web-vitals.md)
- [Bundle Optimization and Reducing Client JS](04-performance-and-scaling/02-bundle-optimization.md)
- [Asset Optimization at Scale](04-performance-and-scaling/03-asset-optimization.md)
- [Edge Runtime, Middleware, and Personalization](04-performance-and-scaling/04-edge-and-middleware.md)
- [Scaling and Infrastructure Considerations](04-performance-and-scaling/05-scaling-infrastructure.md)

### Module 5 — Production Engineering
Ship safely and operate with confidence.

- [Authentication and Authorization Patterns](05-production-engineering/01-auth-patterns.md)
- [Security Hardening](05-production-engineering/02-security.md)
- [Observability: Logging, Tracing, and Monitoring](05-production-engineering/03-observability.md)
- [Testing Strategy for Next.js](05-production-engineering/04-testing-strategy.md)
- [CI/CD and Deployment Strategies](05-production-engineering/05-cicd-deployment.md)

### Module 6 — Senior Topics
The breadth and leadership expected of a lead.

- [Internationalization (i18n)](06-senior-topics/01-internationalization.md)
- [Caching, CDN, and Multi-Region Delivery](06-senior-topics/02-cdn-multiregion.md)
- [Migration and Incremental Adoption](06-senior-topics/03-migration.md)
- [Design Systems and Component Libraries in Next.js](06-senior-topics/04-design-systems.md)
- [Leading a Next.js Codebase](06-senior-topics/05-leading-a-codebase.md)

---

## What You Will Be Able to Do at the End

- Reason precisely about the server/client boundary, rendering, and caching.
- Design a data access layer that is secure, cached, and testable by default.
- Architect a large Next.js codebase with clear boundaries and end-to-end types.
- Diagnose and fix performance problems, and set budgets that hold.
- Harden auth, security, and observability for production.
- Build a robust CI/CD pipeline and choose the right deployment model.
- Lead a Next.js team: set conventions, review effectively, and manage upgrades.

Finish with the **capstone** in the final lesson: architect and ship a
production-grade, multi-region, internationalized, auth-protected, observable
Next.js application that applies everything here.
