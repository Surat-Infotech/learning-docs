# The App Router and File-Based Routing

A **route** is a URL path in your app, like `/about` or `/blog`. In Next.js you
do not write a big list of routes by hand. Instead, the folders you create
inside a special `app` folder *become* your URLs. Think of it like rooms in a
house: the hallway leads to rooms, and each room has a door with a name. The
folder structure is the hallway, and `page.tsx` is the door you walk through to
see a room. This idea is called **file-based routing** because the files and
folders on disk decide the routes.

The **App Router** is the modern routing system in Next.js (introduced in
version 13 and the default in 14 and 15). It lives in the `app` folder and is
what we use throughout this course.

## What You'll Learn

- How folders inside `app/` turn into URL routes automatically
- That `page.tsx` is the file that defines what a route's page looks like
- The root route: `app/page.tsx` maps to `/` (your home page)
- How nested folders create nested URL paths
- The special files Next.js looks for (page, layout, loading, error, not-found, route)
- The difference between a folder that *is* a route and a file you just keep nearby
- How to read a folder tree and predict the URLs it creates

## Folders Become URLs

In the App Router, every folder inside `app/` represents a **path segment** (one
piece of a URL between slashes). To make that folder reachable in the browser,
you add a `page.tsx` file inside it. That file's job is to return the UI for
that route.

Here is the smallest possible example. This file defines your home page:

```tsx
// app/page.tsx
export default function HomePage() {
  // This is what shows when someone visits "/"
  return <h1>Welcome to my site</h1>;
}
```

When you run `npm run dev` and open `http://localhost:3000/`, you see
"Welcome to my site". The folder is `app` (the root), and `page.tsx` makes the
`/` route real.

## The Root Route

The `app/page.tsx` file is special: it maps to `/`, the very top of your site.
There is no extra folder name, because `app` itself is the starting point.

```text
app/
  page.tsx        ->  /
```

Every other route lives in a folder *inside* `app`.

## Nested Folders, Nested Paths

To create `/about`, make a folder called `about` inside `app`, then put a
`page.tsx` inside it.

```tsx
// app/about/page.tsx
export default function AboutPage() {
  return <h1>About us</h1>; // shows at "/about"
}
```

The folder name (`about`) becomes the URL segment (`/about`). The file name
(`page.tsx`) is always the same word, "page". You do **not** name the file
`about.tsx`; the folder carries the name.

You can keep nesting. Put a folder inside a folder to build deeper paths:

```tsx
// app/blog/page.tsx
export default function BlogPage() {
  return <h1>The Blog</h1>; // shows at "/blog"
}
```

```tsx
// app/blog/first-post/page.tsx
export default function FirstPost() {
  return <h1>My first post</h1>; // shows at "/blog/first-post"
}
```

Each folder adds one segment to the URL. The path is just the chain of folder
names from `app` down to the `page.tsx`.

## A Folder Tree and Its URLs

Reading a tree is a skill. Here is a small site with its URL for each `page.tsx`:

```text
app/
  page.tsx                 ->  /
  about/
    page.tsx               ->  /about
  blog/
    page.tsx               ->  /blog
    first-post/
      page.tsx             ->  /blog/first-post
  contact/
    page.tsx               ->  /contact
```

Notice the rule: only folders that contain a `page.tsx` become visitable URLs.
A folder with no `page.tsx` (and no other special file) is not a route on its
own; it is just a container.

## The Special Files

Next.js gives a few file names a built-in meaning. When a file with one of these
names sits inside a route folder, Next.js wires it up automatically. You do not
have to import them anywhere. Here is the overview; we cover each in detail
later in this module.

```text
page.tsx        The UI for a route (makes the route publicly reachable)
layout.tsx      Shared UI that wraps the page and persists between pages
loading.tsx     A loading state shown while the route is fetching/rendering
error.tsx       UI shown when something throws an error in this segment
not-found.tsx   UI shown when a route or resource is missing (404)
route.ts        An API endpoint instead of a page (returns data, not UI)
```

A quick note on `route.ts`: a folder can have *either* a `page.tsx` (for a
visual page) *or* a `route.ts` (for an API that returns data like JSON), but not
both at the same path. We will use `page.tsx` for now.

## Route Folders vs Colocated Files

A common source of confusion: not every file in `app/` is a route. You can keep
helper files right next to your pages without creating new URLs. This is called
**colocation** (keeping related files close together).

```text
app/
  blog/
    page.tsx          ->  /blog        (a real route)
    PostCard.tsx      ->  (no route, just a component file)
    posts.ts          ->  (no route, just data/helpers)
```

Only `page.tsx` (and other special files) create routes. `PostCard.tsx` and
`posts.ts` are simply files you imported into your page. Next.js ignores them
for routing.

```tsx
// app/blog/page.tsx
import { PostCard } from "./PostCard"; // colocated component, not a route

export default function BlogPage() {
  return (
    <main>
      <h1>The Blog</h1>
      <PostCard /> {/* reusing the nearby component */}
    </main>
  );
}
```

So the simple rule is: **a folder is a route only when it has a `page.tsx`
(or another special file). Everything else nearby is just colocated code.**

## Common Mistakes

- **Naming the file after the folder.** It is always `page.tsx`, never
  `about.tsx`. The folder name decides the URL, not the file name.
- **Forgetting `export default`.** A `page.tsx` must `export default` a
  component. Without it, Next.js cannot find the page and the route breaks.
- **Expecting a folder with no `page.tsx` to be visitable.** A bare folder is
  just a container. Visiting it gives a 404 until you add a `page.tsx`.
- **Putting routes outside `app/`.** Only files inside the `app` folder become
  routes. A `page.tsx` in your project root does nothing.
- **Capitalizing folder names by accident.** URLs are case-sensitive on many
  servers. Stick to lowercase folder names like `about`, not `About`.

## Exercises

1. Create `app/page.tsx` that shows "Home" and `app/contact/page.tsx` that shows
   "Contact". Run the dev server and visit `/` and `/contact` to confirm both
   work.
2. Build this tree on paper, then write the URL each `page.tsx` maps to:
   `app/page.tsx`, `app/shop/page.tsx`, `app/shop/cart/page.tsx`,
   `app/help/faq/page.tsx`.
3. Add a `app/blog/page.tsx` and, next to it, a colocated `Header.tsx` component.
   Import and render `Header` inside the blog page. Confirm that visiting
   `/blog/Header` gives a 404 (proving the component is not a route).
4. Create a folder `app/services/` with no `page.tsx`. Visit `/services` and
   note the 404. Then add a `page.tsx` and watch the route start working.

## Key Takeaways

- The App Router lives in the `app/` folder and uses **file-based routing**:
  folders become URL segments.
- A folder becomes a real route only when it contains a `page.tsx` that
  `export default`s a component.
- `app/page.tsx` is the home page (`/`); nested folders create nested paths.
- Special file names (`page`, `layout`, `loading`, `error`, `not-found`,
  `route`) are recognized automatically by Next.js.
- Files that are not special (components, data, helpers) can be **colocated**
  inside route folders without creating new URLs.

---

[← Previous: useEffect and Lifecycle](../01-react-essentials/05-useeffect-and-lifecycle.md) | [Back to Course Index](../README.md) | [Next: Pages, Layouts, and Templates →](02-pages-layouts-templates.md)
