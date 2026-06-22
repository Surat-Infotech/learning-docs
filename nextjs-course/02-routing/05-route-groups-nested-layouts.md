# Route Groups and Nested Layouts

As a project grows, the `app/` folder fills up with many route folders. You will
often want to *organize* those folders, or give one section of the site a
different layout, without changing the URLs. Next.js has a neat tool for this
called a **route group**: a folder whose name is wrapped in parentheses. Think
of it like a labelled drawer in a desk: the drawer keeps things grouped, but the
label is invisible to the URL. In this lesson we learn route groups, private
folders, and a brief look at two advanced features.

## What You'll Learn

- What a **route group** `(name)` is and that it does not affect the URL
- How to use route groups to keep the `app/` folder tidy
- How to apply different layouts to different sections with route groups
- What **private folders** `_components` are and why they are ignored by routing
- A quick awareness of **parallel routes** and **intercepting routes**
- How to read a folder tree that mixes groups and normal routes

## Route Groups Do Not Change the URL

A folder named with parentheses, like `(marketing)`, is a **route group**. Next.js
uses it only to organize files. The group name is *not* part of the URL.

```text
app/
  (marketing)/
    about/
      page.tsx     ->  /about        (NOT /marketing/about)
    contact/
      page.tsx     ->  /contact
  (shop)/
    products/
      page.tsx     ->  /products     (NOT /shop/products)
```

Notice `(marketing)` and `(shop)` vanish from the URLs. `/about` works even
though the file lives inside `(marketing)`. The parentheses are the signal: "this
folder is for organizing, ignore it in the URL."

This is handy for grouping pages that belong together conceptually, marketing
pages versus shop pages, without forcing `/marketing/...` or `/shop/...` into
the addresses.

## Applying Different Layouts per Section

The most powerful use of route groups is giving each section its own layout. You
put a `layout.tsx` inside the group, and it wraps only the pages in that group.

```text
app/
  layout.tsx               <- root layout (whole site)
  (marketing)/
    layout.tsx             <- wraps only marketing pages
    page.tsx               ->  /
    about/
      page.tsx             ->  /about
  (shop)/
    layout.tsx             <- wraps only shop pages
    products/
      page.tsx             ->  /products
    cart/
      page.tsx             ->  /cart
```

Now the marketing section can have a big hero header, while the shop section has
a cart bar, even though neither prefix shows in the URL.

```tsx
// app/(marketing)/layout.tsx
export default function MarketingLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <div>
      <header>Big marketing banner</header>
      {children}
    </div>
  );
}
```

```tsx
// app/(shop)/layout.tsx
export default function ShopLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <div>
      <nav>Cart (0 items)</nav>
      {children}
    </div>
  );
}
```

Both layouts still sit inside the root `app/layout.tsx`, so the nesting is:
root layout, then the group's layout, then the page.

```text
RootLayout
  ShopLayout (cart bar)
    ProductsPage
```

A note: you can even have two different `page.tsx` files map to `/` from two
different groups, but only one group may define the `/` route at a time, or you
get a conflict. Keep one home page.

## Private Folders

Sometimes you want a folder inside `app/` that holds components or helpers but
should **never** become a route, even by accident. Prefix the folder name with
an underscore to make it a **private folder**. Next.js completely ignores it for
routing.

```text
app/
  _components/         <- private: never a route
    Button.tsx
    Card.tsx
  _lib/                <- private: helpers, never a route
    format.ts
  blog/
    page.tsx           ->  /blog
```

You import from private folders like any other file:

```tsx
// app/blog/page.tsx
import { Card } from "../_components/Card"; // from a private folder

export default function BlogPage() {
  return (
    <main>
      <Card>Hello</Card>
    </main>
  );
}
```

The difference from colocation: a private folder makes the *intent* explicit and
guarantees the folder is never treated as a route, which is useful for shared
component or utility folders that live inside `app/`.

## Route Groups vs Private Folders

A quick contrast so they do not blur together:

- **Route group** `(name)`: still contains *routes*, just hides the group name
  from the URL. Used to organize routes and scope layouts.
- **Private folder** `_name`: contains *non-route* files (components, helpers)
  and is fully excluded from routing.

## A Brief Look: Parallel and Intercepting Routes

Next.js has two advanced routing features you do not need yet, but should
recognize the names of:

- **Parallel routes** (folders named with an `@`, like `@team` and `@analytics`)
  let you render multiple independent sub-pages in the *same* layout at once,
  such as two dashboard panels that update separately.
- **Intercepting routes** (folders using `(.)`, `(..)`, etc.) let you
  "intercept" a navigation and show it differently, for example opening a photo
  in a modal over the current page instead of a full new page.

These are powerful for complex apps but rare for beginners. For now, just know
they exist so the syntax does not surprise you later. We focus on the everyday
routing you have already learned.

## Common Mistakes

- **Thinking the group name appears in the URL.** It does not. `(shop)/products`
  is `/products`, not `/shop/products`.
- **Two groups both defining `/`.** Only one route can own a given path. Two home
  pages from two groups cause a conflict error.
- **Using a route group when you meant a private folder.** A group still holds
  routes; if a folder holds only components, prefix it with `_` instead.
- **Forgetting parentheses must wrap the whole name.** It is `(marketing)`, not
  `marketing()` or `(marketing`.
- **Expecting a private folder's files to be reachable by URL.** They never are;
  that is the whole point.

## Exercises

1. Reorganize a small site into `(marketing)` and `(shop)` route groups. Confirm
   the URLs (`/`, `/about`, `/products`) stay the same with no group prefix.
2. Give `(marketing)` a layout with a banner and `(shop)` a layout with a cart
   bar. Verify each section shows only its own layout.
3. Create an `app/_components/` private folder with a `Button` component, import
   it into a page, and confirm `/( _components)` or `/_components` is not a route.
4. Draw the layout nesting (root, group layout, page) for a visit to `/cart`
   inside the `(shop)` group.
5. Look up one real-world use of parallel routes (for example a dashboard) and
   write one sentence on why it helps.

## Key Takeaways

- A **route group** `(name)` organizes folders and scopes layouts **without**
  adding anything to the URL.
- Put a `layout.tsx` inside a group to give that whole section its own shared UI
  while the root layout still wraps everything.
- A **private folder** `_name` is fully ignored by routing; use it for shared
  components and helpers that live inside `app/`.
- Groups still contain routes; private folders contain non-route files. Pick by
  whether the folder holds pages or just helpers.
- **Parallel routes** (`@slot`) and **intercepting routes** (`(.)`) are advanced
  features worth recognizing but not needed for everyday routing yet.

---

[← Previous: Dynamic Routes](04-dynamic-routes.md) | [Back to Course Index](../README.md) | [Next: Loading and Error UI →](06-loading-and-error-ui.md)
