# Linking and Navigation

A website is a web of pages connected by links. On a plain HTML site you connect
pages with an `<a>` tag, but that makes the browser throw away the whole page and
download a fresh one every time, which feels slow. Next.js gives you a smarter
link that swaps only the part of the screen that changed. Think of it like
flipping to a new page in a flipbook rather than printing a whole new book each
time. In this lesson we learn how to link, how to highlight the current page, and
how to navigate from code.

## What You'll Learn

- Why `<Link>` beats a plain `<a>` for internal navigation
- What client-side navigation and prefetching mean (and why they feel fast)
- How to pass the destination with the `href` prop
- How to highlight the active link using `usePathname`
- How to navigate from code with `useRouter` (`push`, `replace`, `back`)
- How to redirect from the server with `redirect()`
- When to use each approach

## `<Link>` vs `<a>`

Next.js ships a `<Link>` component. You import it from `next/link` and use it
much like an `<a>`, but it enables **client-side navigation**: instead of
reloading the entire page, Next.js fetches just the new page's content and swaps
it in. The shared layout (nav, footer) stays put, and the change feels instant.

```tsx
// app/page.tsx
import Link from "next/link";

export default function HomePage() {
  return (
    <nav>
      <Link href="/">Home</Link>
      <Link href="/about">About</Link>
      <Link href="/blog">Blog</Link>
    </nav>
  );
}
```

A plain `<a href="/about">` would still work, but it triggers a full page reload:
the browser discards everything and starts over. With `<Link>`, only the page
content changes, so it is faster and your layout state survives.

`<Link>` also does **prefetching**: when a link scrolls into view, Next.js
quietly downloads that page's code in the background. By the time you click, it
is often already loaded, so the page appears immediately.

Rule of thumb: **use `<Link>` for routes inside your app, and `<a>` only for
external sites** (like `<a href="https://google.com">`).

## The `href` Prop

`href` tells the link where to go. It accepts a string path:

```tsx
// app/components/Menu.tsx
import Link from "next/link";

export function Menu() {
  return (
    <ul>
      <li><Link href="/">Home</Link></li>
      <li><Link href="/blog/first-post">First post</Link></li>
      <li><Link href="/about">About</Link></li>
    </ul>
  );
}
```

You can also build dynamic hrefs with a template string, for example linking to
a specific blog post by its slug:

```tsx
// app/blog/page.tsx
import Link from "next/link";

const slugs = ["hello-world", "next-tips"];

export default function BlogPage() {
  return (
    <ul>
      {slugs.map((slug) => (
        // becomes /blog/hello-world, /blog/next-tips, etc.
        <li key={slug}>
          <Link href={`/blog/${slug}`}>{slug}</Link>
        </li>
      ))}
    </ul>
  );
}
```

## Active Link Styling with `usePathname`

Often you want to highlight the link for the page you are currently on. To know
the current URL, use the `usePathname` hook from `next/navigation`. Because
hooks run in the browser, this must be a **client component** (note the
`"use client"` line at the top).

```tsx
// app/components/NavLinks.tsx
"use client"; // required: hooks run on the client

import Link from "next/link";
import { usePathname } from "next/navigation";

const links = [
  { href: "/", label: "Home" },
  { href: "/about", label: "About" },
  { href: "/blog", label: "Blog" },
];

export function NavLinks() {
  const pathname = usePathname(); // e.g. "/about"

  return (
    <nav>
      {links.map((link) => {
        const isActive = pathname === link.href; // is this the current page?
        return (
          <Link
            key={link.href}
            href={link.href}
            // give the active link a special class
            className={isActive ? "active" : ""}
          >
            {link.label}
          </Link>
        );
      })}
    </nav>
  );
}
```

`usePathname` returns the current path as a string. Comparing it to each link's
`href` tells you which link is active so you can style it differently.

## Programmatic Navigation with `useRouter`

Sometimes you need to navigate from code rather than a click, for example after a
form submits successfully. The `useRouter` hook (from `next/navigation`, again
client-only) gives you a `router` object with navigation methods.

```tsx
// app/login/LoginForm.tsx
"use client";

import { useRouter } from "next/navigation";

export function LoginForm() {
  const router = useRouter();

  function handleSubmit() {
    // ...pretend we logged the user in successfully...
    router.push("/dashboard"); // go to the dashboard
  }

  return <button onClick={handleSubmit}>Log in</button>;
}
```

The three methods you will use most:

- `router.push("/path")` — go to a new page and **add** it to history (the back
  button returns to the current page).
- `router.replace("/path")` — go to a new page but **replace** the current entry
  in history (the back button skips it). Good after login so the user cannot go
  "back" to the login form.
- `router.back()` — go to the previous page, like clicking the browser's back
  button.

```tsx
// app/components/BackButton.tsx
"use client";

import { useRouter } from "next/navigation";

export function BackButton() {
  const router = useRouter();
  return <button onClick={() => router.back()}>Go back</button>;
}
```

## Redirecting on the Server with `redirect()`

The hooks above run in the browser. But sometimes you want to redirect on the
**server**, before any UI is sent, for example "if the user is not logged in,
send them to the login page". For that, use the `redirect()` function from
`next/navigation` inside a server component.

```tsx
// app/dashboard/page.tsx
import { redirect } from "next/navigation";

async function getUser() {
  // pretend this checks a session; returns null if not logged in
  return null;
}

export default async function DashboardPage() {
  const user = await getUser();

  if (!user) {
    redirect("/login"); // stops rendering and sends the user to /login
  }

  return <h1>Welcome back</h1>;
}
```

`redirect()` immediately stops rendering the current page and sends the visitor
elsewhere. It does not need `"use client"`, because it runs on the server. Note
that you do not `return` after it; it throws internally to halt execution.

## Common Mistakes

- **Using `<a>` for internal links.** This forces a full reload and loses
  client-side speed. Use `<Link>` for routes inside your app.
- **Calling `usePathname` or `useRouter` in a server component.** They are
  client hooks. Add `"use client"` at the top of the file, or move them into a
  small client component.
- **Importing `useRouter` from the wrong place.** In the App Router it comes from
  `next/navigation`, not the old `next/router` (that was the Pages Router).
- **Using `push` when `replace` is right.** After login or a wizard step, use
  `replace` so the user cannot navigate "back" into a stale form.
- **Calling `redirect()` and then writing more code expecting it to run.**
  Execution stops at `redirect()`; nothing after it in that path runs.

## Exercises

1. Build a `NavLinks` client component with links to `/`, `/about`, and `/blog`.
   Add CSS so the active link is bold using `usePathname`.
2. Create a blog list page that maps over an array of slugs and renders a
   `<Link>` to `/blog/{slug}` for each one.
3. Make a fake login form that calls `router.replace("/dashboard")` on submit.
   Confirm the browser back button does not return you to the form.
4. Add a "Go back" button using `router.back()` on a detail page and test it.
5. In a server component for `/dashboard`, use `redirect("/login")` when there is
   no user. Confirm you land on `/login`.

## Key Takeaways

- Use `<Link href="...">` from `next/link` for internal navigation: it does
  fast **client-side navigation** and **prefetching**. Use `<a>` only for
  external links.
- `usePathname` (client-only) returns the current path so you can style the
  active link.
- `useRouter` (client-only) navigates from code: `push` adds history, `replace`
  swaps the current entry, `back` goes to the previous page.
- `redirect()` (from `next/navigation`) redirects on the **server** and stops
  rendering immediately.
- Hooks (`usePathname`, `useRouter`) require `"use client"`; `redirect()` runs
  on the server and does not.

---

[← Previous: Pages, Layouts, and Templates](02-pages-layouts-templates.md) | [Back to Course Index](../README.md) | [Next: Dynamic Routes →](04-dynamic-routes.md)
