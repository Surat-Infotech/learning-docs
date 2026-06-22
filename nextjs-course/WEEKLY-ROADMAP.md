# Next.js Course — 8-Week Weekly Roadmap (15 hours/week)

This roadmap turns the [Next.js Course](README.md) into a clear weekly plan.

- **Time budget:** 15 hours per week
- **Total length:** 8 weeks (~120 hours)
- **Split:** About **40% reading** and **60% writing code** (so ~6 hours reading
  and ~9 hours coding each week).

> **Prerequisite:** Comfort with basic JavaScript (functions, arrays, objects,
> async/await). If you need it first, do the
> [JavaScript Course](../javascript-course/README.md).

> **How to read this plan.** Each week lists the lessons to study and a feature to
> add to one growing project — a **Blog + Dashboard** app. By Week 8 you will have
> built and deployed a complete full-stack Next.js app.

**Weekly rhythm (suggested):** 3 study days × ~5 hours, or 5 days × 3 hours.

---

## Week-at-a-Glance

| Week | Focus                       | Lessons                         | Project Milestone                       |
| ---- | --------------------------- | ------------------------------- | --------------------------------------- |
| 1    | Setup + React Essentials    | Module 0 + Module 1 (1–3)       | Static components + first page          |
| 2    | React Interactivity         | Module 1 (4–5)                  | Interactive UI (forms, effects)         |
| 3    | Routing                     | Module 2                        | Multi-page blog with layouts            |
| 4    | Rendering & Data            | Module 3 (1–3)                  | Server-rendered post list + detail      |
| 5    | Mutations & Streaming       | Module 3 (4–5)                  | Add/edit posts with Server Actions      |
| 6    | Building Features           | Module 4                        | Styling, images, API, auth              |
| 7    | Data & State                | Module 5                        | Database-backed posts + env config      |
| 8    | Production + Final          | Module 6                        | Polish, test, and deploy                |

---

## Week 1 — Setup and React Basics

**Goal:** Run a Next.js app and learn the React building blocks.

**Lessons (Module 0 + Module 1, lessons 1–3):**

- [What Is Next.js](00-getting-started/01-what-is-nextjs.md)
- [Setup and Project Structure](00-getting-started/02-setup-and-project-structure.md)
- [Components and JSX](01-react-essentials/01-components-and-jsx.md)
- [Props and State](01-react-essentials/02-props-and-state.md)
- [Lists and Conditional Rendering](01-react-essentials/03-lists-and-conditional-rendering.md)

**Project milestone — Start the Blog:**

- Create the app with `create-next-app` (TypeScript + Tailwind + App Router).
- Build a `PostCard` component that takes `title`, `excerpt`, and `date` as props.
- Render a list of hard-coded posts on the home page using `.map()`.

✅ **End-of-week check:** You can create a Next.js app and build components that
render data with props.

---

## Week 2 — React Interactivity

**Goal:** Make components respond to the user.

**Lessons (Module 1, lessons 4–5):**

- [Events and Forms](01-react-essentials/04-events-and-forms.md)
- [`useEffect` and the Component Lifecycle](01-react-essentials/05-useeffect-and-lifecycle.md)

**Project milestone — Add Interactivity:**

- Add a client-side **search box** that filters the post list as you type.
- Add a "like" button to each `PostCard` that tracks count in state.
- Use `useEffect` to save the search term to `localStorage` and restore it.

✅ **End-of-week check:** You can build interactive client components with state,
events, and effects.

---

## Week 3 — Routing

**Goal:** Turn the app into a real multi-page site.

**Lessons (Module 2):**

- [The App Router and File-Based Routing](02-routing/01-app-router-basics.md)
- [Pages, Layouts, and Templates](02-routing/02-pages-layouts-templates.md)
- [Linking and Navigation](02-routing/03-linking-and-navigation.md)
- [Dynamic Routes](02-routing/04-dynamic-routes.md)
- [Route Groups and Nested Layouts](02-routing/05-route-groups-nested-layouts.md)
- [Loading and Error UI](02-routing/06-loading-and-error-ui.md)

**Project milestone — Multi-Page Blog:**

- Add a root layout with a shared nav bar and footer.
- Create `/blog` (list) and `/blog/[slug]` (single post) pages.
- Link cards to their post pages with `<Link>`.
- Add `loading.tsx` and `not-found.tsx` for the blog section.

✅ **End-of-week check:** You can build nested routes, dynamic pages, layouts, and
loading/error states.

---

## Week 4 — Rendering and Data

**Goal:** Fetch real data on the server.

**Lessons (Module 3, lessons 1–3):**

- [Server Components vs Client Components](03-rendering-and-data/01-server-and-client-components.md)
- [Data Fetching in Server Components](03-rendering-and-data/02-data-fetching.md)
- [Caching and Revalidation](03-rendering-and-data/03-caching-and-revalidation.md)

**Project milestone — Server-Rendered Posts:**

- Move posts to a JSON API or a local file/route and fetch them in an async
  Server Component.
- Build the post detail page by fetching a single post by `slug`; call
  `notFound()` when missing.
- Add time-based revalidation to the list.

✅ **End-of-week check:** You understand Server vs Client Components and can fetch
and cache data on the server.

---

## Week 5 — Mutations and Streaming

**Goal:** Change data and improve perceived speed.

**Lessons (Module 3, lessons 4–5):**

- [Streaming and Suspense](03-rendering-and-data/04-streaming-and-suspense.md)
- [Server Actions and Mutations](03-rendering-and-data/05-server-actions.md)

**Project milestone — Create and Edit Posts:**

- Add a "New Post" form powered by a **Server Action**; `revalidatePath` after.
- Wrap a slow section (e.g. comments) in `<Suspense>` with a fallback.
- Show a pending state on the submit button with `useFormStatus`.

✅ **End-of-week check:** You can mutate data with Server Actions and stream slow
content with Suspense.

---

## Week 6 — Building Features

**Goal:** Add the pieces every real app needs.

**Lessons (Module 4):**

- [Styling Your App](04-building-features/01-styling.md)
- [Images, Fonts, and Metadata](04-building-features/02-images-fonts-metadata.md)
- [Route Handlers (API Routes)](04-building-features/03-route-handlers.md)
- [Forms and Data Mutations](04-building-features/04-forms-and-mutations.md)
- [Authentication Basics](04-building-features/05-authentication-basics.md)

**Project milestone — Polish + Protect:**

- Style the app with Tailwind; add cover images with `<Image>` and a custom font.
- Add per-page SEO metadata.
- Build a `GET /api/posts` Route Handler.
- Add validation to the new-post form, and protect the "New Post" page behind a
  simple session check.

✅ **End-of-week check:** You can style, optimize media, expose an API, validate
forms, and gate pages behind auth.

---

## Week 7 — Data and State

**Goal:** Back the app with a real database.

**Lessons (Module 5):**

- [Working with a Database](05-data-and-state/01-databases-overview.md)
- [Environment Variables](05-data-and-state/02-environment-variables.md)
- [Client State Management](05-data-and-state/03-client-state-management.md)

**Project milestone — Database-Backed Blog:**

- Set up a hosted Postgres (e.g. Neon/Supabase) with Prisma or Drizzle.
- Move posts into the database; read in Server Components, write in Server Actions.
- Store the connection string in `.env.local` (never committed).
- Move the search filter to use the URL (`searchParams`) instead of local state.

✅ **End-of-week check:** You can connect to a database, manage secrets, and choose
the right place for state.

---

## Week 8 — Production and Final

**Goal:** Make it robust and ship it.

**Lessons (Module 6):**

- [Error Handling in Production](06-production/01-error-handling.md)
- [Performance and Optimization](06-production/02-performance-optimization.md)
- [Testing Your Next.js App](06-production/03-testing.md)
- [Deployment](06-production/04-deployment.md)
- [Best Practices and Project Structure](06-production/05-best-practices.md)

**Final project — Ship the Blog + Dashboard:**

- Add friendly error and not-found pages.
- Audit `"use client"` boundaries and lazy-load any heavy component.
- Write a couple of component tests and one Playwright end-to-end test.
- Deploy to **Vercel** with environment variables and preview deployments.
- Clean up the folder structure and write a project README.

✅ **End-of-week check:** You can plan, build, test, and deploy a full-stack
Next.js app on your own.

---

## Readiness Checkpoints

- [ ] **End of Week 2:** Build interactive client components.
- [ ] **End of Week 3:** Build a multi-page app with routing and layouts.
- [ ] **End of Week 5:** Fetch data on the server and mutate it with Server Actions.
- [ ] **End of Week 7:** Back the app with a real database.
- [ ] **End of Week 8:** Test and deploy a full-stack app to production.

When you can build and ship an app like this without a tutorial, you are ready for
the [Advanced Next.js Course](../nextjs-advanced-course/README.md).

---

[← Back to Course Index](README.md)
