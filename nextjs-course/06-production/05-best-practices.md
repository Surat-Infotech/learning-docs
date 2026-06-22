# Best Practices and Project Structure

You now know all the pieces of Next.js. This final lesson is about putting them
together with good habits, so your project stays tidy and easy to grow as it gets
bigger.

Think of it like organizing a kitchen. You *could* throw every pot, spice, and
plate into one giant drawer — and for a tiny meal it would not matter. But the
day you cook something real, a well-organized kitchen makes everything faster and
calmer. These habits are how you keep a Next.js project from turning into one
giant messy drawer.

## What You'll Learn

- How to organize folders with colocation, `_components`, `lib/`, and route groups
- The "server-first" mindset and how to keep client components minimal
- Fetching data right where it is used
- Building reusable components
- Using TypeScript everywhere for safety
- Accessibility and semantic HTML basics
- Keeping environment variables and secrets safe
- A complete example project structure

## Organizing Your Folders

There is no single "correct" structure, but a few conventions make projects feel
familiar to anyone who opens them.

### Colocation: keep related files together

Next.js only treats `page.tsx`, `layout.tsx`, and a handful of special files as
routes. Everything else in a route folder is **ignored** by the router, so you
can safely keep helper files right next to the page that uses them.

```text
app/
  dashboard/
    page.tsx          <- the route
    DashboardChart.tsx <- a component used only here, kept nearby
```

### `_components`: private folders

Prefixing a folder with an underscore (like `_components`) tells Next.js, "this is
private — never make it a route." It is a clear signal that a folder holds support
files, not pages.

```text
app/
  dashboard/
    _components/      <- never a route; just helpers for this section
      Chart.tsx
      StatCard.tsx
    page.tsx
```

### `lib/`: shared logic

Put plain, reusable functions — data fetching, formatting, validation — in a
top-level `lib/` folder. This is also exactly what made testing easier in an
earlier lesson.

```ts
// lib/products.ts
export async function getProducts() {
  const res = await fetch("https://api.example.com/products");
  return res.json();
}
```

### Route groups: organize without changing URLs

Wrapping a folder in parentheses, like `(marketing)`, groups routes together
**without** adding to the URL. Great for giving different sections their own
layout.

```text
app/
  (marketing)/        <- the "(marketing)" part does NOT appear in the URL
    layout.tsx        <- a layout just for marketing pages
    page.tsx          <- the URL is still "/"
    about/page.tsx    <- the URL is "/about"
  (app)/
    layout.tsx        <- a different layout for the logged-in app
    dashboard/page.tsx
```

## The Server-First Mindset

The single most important habit: **start everything as a Server Component, and
only reach for `"use client"` when you truly need interactivity.** This keeps
JavaScript small and pages fast.

```tsx
// Default: a Server Component. Most of your app should look like this.
// app/posts/page.tsx
export default async function PostsPage() {
  const posts = await getPosts();
  return <ul>{posts.map((p) => <li key={p.id}>{p.title}</li>)}</ul>;
}
```

When you do need a client component, make it small and push it to the edges (a
single button, a search box), not the whole page.

## Fetch Data Where It Is Used

In older React apps, people fetched all data at the top and drilled it down
through many props. With Server Components you can do something nicer: **fetch
data in the very component that needs it.** Next.js automatically avoids fetching
the same thing twice in one request, so this is safe.

```tsx
// app/components/UserBadge.tsx  (Server Component)
import { getUser } from "@/lib/users";

export default async function UserBadge() {
  const user = await getUser(); // fetched right here, no prop-drilling
  return <span>Hi, {user.name}</span>;
}
```

## Build Reusable Components

If you copy and paste the same markup three times, turn it into a component. A
good reusable component takes props and does not hard-code its content.

```tsx
// app/components/Button.tsx
type Props = {
  children: React.ReactNode;
  onClick?: () => void;
};

export default function Button({ children, onClick }: Props) {
  return (
    <button className="btn" onClick={onClick}>
      {children}
    </button>
  );
}
```

Now every button in your app looks consistent, and you change the style in one
place.

## TypeScript Everywhere

TypeScript catches mistakes before your users do. Type your props, your function
arguments, and your data shapes. It feels like extra typing at first but saves you
from a whole category of bugs.

```ts
// lib/types.ts
export type Product = {
  id: string;
  name: string;
  priceCents: number;
};
```

```tsx
// Using the type makes wrong usage an instant error in your editor.
function ProductCard({ product }: { product: Product }) {
  return <h3>{product.name}</h3>;
}
```

## Accessibility and Semantic HTML

Accessibility means your app works for everyone, including people using screen
readers or a keyboard instead of a mouse. The easiest way to get it mostly right
is to use **semantic HTML** — tags that describe what they are.

```tsx
// Good: meaningful tags that screen readers understand.
<header>
  <nav>...</nav>
</header>
<main>
  <h1>Welcome</h1>
  <button onClick={save}>Save</button>
</main>
<footer>...</footer>
```

```tsx
// Avoid: a pile of divs that mean nothing to assistive tech.
<div onClick={save}>Save</div> // not focusable, not announced as a button
```

A few quick rules: use `<button>` for actions, `<a>`/`<Link>` for navigation,
always give `<img>`/`<Image>` an `alt`, label your form inputs, and use one
`<h1>` per page with headings in order.

## Environment and Secret Hygiene

Treat secrets with care so they never leak.

- Keep secrets in `.env.local`, and make sure it is in `.gitignore`.
- Only prefix a variable with `NEXT_PUBLIC_` if it is genuinely safe for the
  whole world to see.
- Read secrets on the server (in Server Components, Server Actions, or `lib/`),
  never in client code.
- Set the same variables in your host's dashboard when you deploy.

```ts
// Server-side only: safe.
const key = process.env.STRIPE_SECRET_KEY;

// Client-safe (because of the prefix): never put secrets here.
const analyticsId = process.env.NEXT_PUBLIC_ANALYTICS_ID;
```

## An Example Project Structure

Putting it all together, a healthy small-to-medium app might look like this:

```text
my-app/
  app/
    (marketing)/
      layout.tsx
      page.tsx
      about/page.tsx
    (app)/
      layout.tsx
      dashboard/
        _components/
          Chart.tsx
        page.tsx
        loading.tsx
        error.tsx
    api/
      webhook/route.ts
    layout.tsx          <- root layout
    not-found.tsx
  components/            <- shared UI used across the whole app
    Button.tsx
    Card.tsx
  lib/                  <- data fetching, helpers, types
    products.ts
    users.ts
    types.ts
  public/               <- static images and files
  .env.local            <- secrets (gitignored)
  next.config.ts
  package.json
```

This is a starting point, not a law. The goal is that anyone — including
future-you — can open the project and quickly find their way around.

## Common Mistakes

- **Reaching for `"use client"` by default.** Start on the server and add the
  directive only where you need interactivity.
- **Prop-drilling data from the top.** With Server Components, fetch data in the
  component that uses it.
- **Copy-pasting markup.** Extract repeated UI into a small reusable component.
- **Skipping TypeScript types.** Typing your props and data catches bugs in the
  editor instead of in production.
- **Using `<div onClick>` for buttons.** Use real semantic tags so the app is
  accessible and keyboard-friendly.

## Exercises

1. Reorganize a small project to use a `lib/` folder for data functions and a
   shared `components/` folder for reusable UI.
2. Add a route group like `(marketing)` and give it its own layout, confirming the
   URLs do not change.
3. Take a component that drills a prop several levels deep and refactor it so the
   data is fetched where it is used.
4. Add TypeScript types for one of your data shapes and use them in a component.
5. Audit one page for accessibility: are you using `<button>`, `<nav>`, `<main>`,
   and `alt` text correctly? Fix one thing.

## Key Takeaways

- Organize with colocation, `_components` for private files, `lib/` for shared
  logic, and route groups for layouts without URL changes.
- Adopt a server-first mindset and keep client components small.
- Fetch data where it is used, build reusable components, and type everything with
  TypeScript.
- Use semantic HTML for accessibility, and keep secrets out of client code and out
  of Git.
- A clear structure makes a growing app easy to navigate.

## You Did It

Take a moment — this is the end of the course. You started not knowing what
Next.js was, and now you can build pages and layouts, mix Server and Client
Components, fetch and mutate data, handle forms and state, recover from errors,
optimize for speed, test your work, and deploy a real app to the internet. That
is a complete, professional skill set. Whatever you wanted to build when you
started, you can build and ship it now.

The best next step is to **build something real** — a personal site, a small tool,
a project you actually care about. You will learn more from one real project than
from ten more tutorials. When you are ready to go deeper into advanced patterns,
the advanced course is waiting for you. Go make something.

---

[← Previous: Deployment](04-deployment.md) | [Back to Course Index](../README.md)
