# Pages, Layouts, and Templates

Most websites have parts that stay the same on every screen: a navigation bar at
the top, a footer at the bottom, maybe a sidebar. You would not want to copy and
paste that nav bar into every single page. Instead, Next.js lets you write it
once in a **layout**. Think of a layout as a picture frame: you swap the photo
inside (the page), but the frame around it stays put. In this lesson we learn
layouts, the special root layout, nesting layouts, and a cousin called a
**template**.

## What You'll Learn

- What `layout.tsx` is and how it wraps your pages
- That a layout **persists** (stays mounted) as you move between pages
- The root layout, and why it owns the `<html>` and `<body>` tags
- The `{children}` prop and how the page slots into the layout
- How to nest layouts so different sections get different shared UI
- The difference between `layout.tsx` and `template.tsx`
- Where to put shared UI so it appears in the right places

## What a Layout Is

A `layout.tsx` file defines UI that wraps the page in the same folder *and* every
page in folders below it. It receives the page as a special prop called
`children` and decides where to place it.

```tsx
// app/layout.tsx
export default function RootLayout({
  children,
}: {
  children: React.ReactNode; // the page (or nested layout) goes here
}) {
  return (
    <html lang="en">
      <body>
        <nav>My Site</nav> {/* shows on every page */}
        {children} {/* the active page renders here */}
        <footer>© 2026</footer> {/* shows on every page */}
      </body>
    </html>
  );
}
```

Whatever route you visit, the `<nav>` and `<footer>` appear, and the matching
`page.tsx` fills the `{children}` slot.

## Layouts Persist Across Navigation

Here is the key benefit. When you click from `/about` to `/blog`, the layout
does **not** reload. Only the `{children}` part swaps to the new page. The
layout stays mounted, keeping its state.

Why does that matter? Imagine a nav bar with an open dropdown menu, or a music
player that keeps playing. Because the layout persists, those keep working
across navigation. A normal full-page reload would reset all of that.

```text
Click /about -> /blog

  layout.tsx          <- stays the same, NOT re-rendered
    children:
      about page  ->  blog page   <- only this part changes
```

## The Root Layout

The top-level `app/layout.tsx` is the **root layout**, and it is required. Every
Next.js App Router project must have one. It is special because it is the only
layout that defines the `<html>` and `<body>` tags. Those tags wrap your entire
application.

```tsx
// app/layout.tsx
import "./globals.css"; // global styles, imported once here

export const metadata = {
  title: "My Site",
  description: "A site built with Next.js",
};

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body>{children}</body>
    </html>
  );
}
```

Rules for the root layout:

- It must include `<html>` and `<body>`. No other layout should.
- It cannot be removed; it is the base frame for the whole app.
- Global CSS is usually imported here so it applies everywhere.

## Nested Layouts

Layouts nest, just like folders. A `layout.tsx` inside a sub-folder wraps only
the pages in that folder and below, and it itself is wrapped by the layouts
above it. This lets a section of your site have its own shared UI.

Say the dashboard needs a sidebar that the rest of the site should not have:

```tsx
// app/dashboard/layout.tsx
export default function DashboardLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <div style={{ display: "flex" }}>
      <aside>Dashboard menu</aside> {/* only on /dashboard pages */}
      <section>{children}</section>
    </div>
  );
}
```

```tsx
// app/dashboard/page.tsx
export default function DashboardPage() {
  return <h1>Dashboard home</h1>;
}
```

Now the rendering nests like Russian dolls:

```text
RootLayout (html, body, nav)
  DashboardLayout (sidebar)
    DashboardPage (the content)
```

Visiting `/dashboard` shows the site nav *and* the dashboard sidebar. Visiting
`/about` shows only the site nav, because `/about` is not inside `dashboard`.

## Templates vs Layouts

A **template** (`template.tsx`) looks almost identical to a layout: it also wraps
pages and receives `children`. The one big difference is what happens on
navigation:

- A **layout** persists. It stays mounted; its state survives navigation.
- A **template** re-mounts. A fresh copy is created on every navigation, so its
  state resets each time.

```tsx
// app/template.tsx
export default function Template({
  children,
}: {
  children: React.ReactNode;
}) {
  // This component re-mounts on every page change.
  return <div className="fade-in">{children}</div>;
}
```

When would you want re-mounting? Cases like:

- A fade-in animation you want to play again on each page change.
- A `useEffect` that should run fresh every time you enter a new page.
- Per-page state that must reset rather than carry over.

If you do not need any of that, use a layout. Layouts are the default choice
because persisting is usually what you want, and it is more efficient.

## Where to Put Shared UI

A quick guide for deciding where shared UI lives:

- **Appears on every page?** Put it in the root `app/layout.tsx` (nav, footer).
- **Appears only in one section?** Put it in a nested layout for that folder
  (a dashboard sidebar in `app/dashboard/layout.tsx`).
- **Needs to reset/animate on each navigation?** Use a `template.tsx`.
- **Specific to one page only?** Just put it inside that `page.tsx`. No layout
  needed.

## Common Mistakes

- **Adding `<html>`/`<body>` to a nested layout.** Only the root layout has
  them. Putting them in a nested layout creates invalid, duplicated HTML.
- **Forgetting to render `{children}`.** If a layout does not include
  `{children}`, the page simply never shows. The frame appears with no photo.
- **Deleting the root layout.** It is required. Without `app/layout.tsx` the app
  will not build.
- **Reaching for a template when a layout works.** Templates re-mount and lose
  state every navigation. Use them only when you actually need that behavior.
- **Expecting a layout to re-run its effects on navigation.** It persists, so a
  `useEffect` inside it runs once, not on every page change.

## Exercises

1. Create a root `app/layout.tsx` with a `<nav>` containing the site name and a
   `<footer>`. Confirm both appear on `/` and on `/about`.
2. Add a `app/dashboard/layout.tsx` with a sidebar, plus `app/dashboard/page.tsx`
   and `app/dashboard/settings/page.tsx`. Verify the sidebar shows on both
   dashboard URLs but not on `/`.
3. Add a counter button to your root layout (make it a client component). Click
   it a few times, then navigate between pages. Confirm the count survives,
   proving the layout persists.
4. Move that same counter into a `template.tsx` instead. Navigate again and
   confirm the count resets each time, proving the template re-mounts.
5. Draw the nesting order (which layout wraps which) for a visit to
   `/dashboard/settings`.

## Key Takeaways

- A `layout.tsx` wraps pages via `{children}` and **persists** across
  navigation, keeping shared UI and state alive.
- The **root layout** (`app/layout.tsx`) is required and owns the `<html>` and
  `<body>` tags; no other layout should include them.
- Layouts **nest** like folders, letting sections of the site have their own
  shared UI such as a dashboard sidebar.
- A `template.tsx` is like a layout but **re-mounts** on every navigation,
  resetting its state. Use it only when you need fresh mounting or animations.
- Put global UI in the root layout, section UI in nested layouts, and page-only
  UI inside the page itself.

---

[← Previous: The App Router and File-Based Routing](01-app-router-basics.md) | [Back to Course Index](../README.md) | [Next: Linking and Navigation →](03-linking-and-navigation.md)
