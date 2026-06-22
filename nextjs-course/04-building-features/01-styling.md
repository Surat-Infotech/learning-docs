# Styling Your App

Your app *works*, but right now it probably looks plain. Time to make it pretty! 🎨
Styling is like choosing paint, furniture, and decorations for a room you've already
built. Next.js gives you several ways to add styles, and they all play nicely
together. In this lesson we'll meet each one and learn when to reach for which.

## What You'll Learn

- What **global CSS** is and where to put it (`app/globals.css`)
- How **CSS Modules** give you *scoped* styles that don't clash
- What **Tailwind CSS** is and how `create-next-app` sets it up for you
- How to use **inline styles** for quick one-off cases
- How to switch classes on and off with **conditional classes** (and the `clsx` helper)
- How to **choose an approach** for your project

## Global CSS

**Global CSS** means styles that apply to your *entire* app — every page, every
component. Think of it like the house rules: they affect everyone.

In a Next.js App Router project, there's already a file for this:

```css
/* app/globals.css */

/* These rules affect the whole app */
body {
  margin: 0;
  font-family: sans-serif;
  background-color: #fafafa;
  color: #111;
}

/* A class anyone can use */
.headline {
  font-size: 2rem;
  font-weight: bold;
}
```

You import this file **once**, in your root layout, and it covers everything:

```tsx
// app/layout.tsx
import "./globals.css"; // load the global styles for the whole app

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

Global CSS is perfect for things like fonts, body background, and reset rules. But be
careful: because it's *global*, a class named `.button` here will affect **every**
button in your app. That can cause surprises in big projects.

## CSS Modules (Scoped Styles)

A **CSS Module** is a CSS file whose class names only apply to the *one component*
that imports it. The styles are **scoped** — locked to that component, like a label
on your own lunchbox so nobody else's food gets mixed up.

The trick: name the file `*.module.css`.

```css
/* app/components/card.module.css */
.card {
  border: 1px solid #ddd;
  border-radius: 8px;
  padding: 16px;
}

.title {
  font-size: 1.25rem;
  color: navy;
}
```

Now import it as an object and use the keys:

```tsx
// app/components/card.tsx
import styles from "./card.module.css"; // import as an object

export default function Card() {
  // styles.card becomes a unique name like "card_a1b2c3"
  return (
    <div className={styles.card}>
      <h2 className={styles.title}>Hello!</h2>
    </div>
  );
}
```

Behind the scenes, Next.js turns `.card` into a unique name so it can never clash
with a `.card` somewhere else. You get the safety of scoped styles with plain CSS
syntax. 

> A class is **scoped** when it only affects the component that uses it, not the
> whole app.

## Tailwind CSS

**Tailwind CSS** is a popular styling tool built around **utility classes** — tiny,
single-purpose classes you stack together right in your markup. Instead of writing a
`.card` rule in a separate file, you describe the look inline with classes.

```tsx
// app/page.tsx
export default function Home() {
  // p-4 = padding, rounded = rounded corners, bg-white = white background
  return (
    <div className="p-4 rounded border border-gray-300 bg-white">
      <h2 className="text-xl text-blue-700 font-bold">Hello!</h2>
    </div>
  );
}
```

Each class does one thing:

- `p-4` -> padding on all sides
- `text-xl` -> larger text
- `bg-white` -> white background
- `rounded` -> rounded corners

### How create-next-app sets it up

When you run `create-next-app` and say "yes" to Tailwind, it does the setup for you:

```bash
# during project creation, you'll be asked:
# Would you like to use Tailwind CSS? > Yes
npx create-next-app@latest my-app
```

It adds Tailwind's config and a special import line at the top of `globals.css`:

```css
/* app/globals.css (Tailwind v4 style) */
@import "tailwindcss"; /* this line turns on all the utility classes */
```

After that, the utility classes just work everywhere. No extra imports per file.

Tailwind feels strange at first ("why so many classes?!") but it's fast to write and
keeps styles close to your markup. Many Next.js projects use it by default.

## Inline Styles

**Inline styles** are styles written directly on an element using the `style`
attribute. In React, you pass an **object** (not a string), and property names are
written in camelCase.

```tsx
// app/components/badge.tsx
export default function Badge() {
  // note the double braces: one for JSX, one for the object
  return (
    <span style={{ backgroundColor: "gold", padding: "4px 8px" }}>
      New!
    </span>
  );
}
```

Inline styles are handy for quick, one-off, or **dynamic** values (like a width that
comes from data). But avoid using them for everything — they can't do hover effects
or media queries, and they clutter your markup.

## Conditional Classes

Often you want a class only *sometimes* — for example, highlight a button when it's
active. You can do this with plain JavaScript:

```tsx
// app/components/tab.tsx
export default function Tab({ active }: { active: boolean }) {
  // pick a class based on the "active" prop
  const className = active ? "tab tab-active" : "tab";
  return <button className={className}>Profile</button>;
}
```

This works, but gets messy with many conditions. The tiny library **`clsx`** makes it
cleaner. It joins class names and skips any that are falsy (false, undefined, etc.).

```bash
npm install clsx
```

```tsx
// app/components/tab.tsx
import clsx from "clsx";

export default function Tab({ active }: { active: boolean }) {
  // each key is a class; it's added only when its value is true
  const className = clsx("tab", { "tab-active": active });
  return <button className={className}>Profile</button>;
}
```

If `active` is `true`, you get `"tab tab-active"`. If `false`, just `"tab"`. Clean and
readable.

## Choosing an Approach

You don't have to pick only one — but here's a simple guide:

| Approach        | Reach for it when...                                  |
|-----------------|-------------------------------------------------------|
| **Global CSS**  | Body styles, fonts, resets — true app-wide rules      |
| **CSS Modules** | You want plain CSS but safely scoped to one component |
| **Tailwind**    | You want to style fast right in your markup           |
| **Inline**      | A quick one-off or a *dynamic* value from data        |

Most teams pick **one main approach** (often Tailwind *or* CSS Modules) and use global
CSS for the basics. Pick what feels comfortable — consistency matters more than the
choice itself.

## Common Mistakes

- **Putting component-specific styles in `globals.css`.** A `.button` there affects
  *every* button. Use a CSS Module or Tailwind for component styles.
- **Writing inline styles as a string.** In React it must be an *object*:
  `style={{ color: "red" }}`, not `style="color: red"`.
- **Forgetting the `.module.css` ending.** Without it, the file is treated as global
  CSS and won't be scoped.
- **Mixing camelCase up in inline styles.** Use `backgroundColor`, not
  `background-color`.
- **Trying hover/media queries in inline styles.** They can't do that — use a CSS
  file, a module, or Tailwind instead.

## Exercises

1. Add a global rule in `app/globals.css` that sets the page background and a default
   font. Confirm it applies everywhere.
2. Create `card.module.css` with a `.card` class, then import and use it in a `Card`
   component. Inspect the element in your browser and notice the unique class name.
3. Rebuild that same card using only Tailwind utility classes. Compare which you
   prefer.
4. Make a `Button` component that takes a `primary` prop and uses `clsx` to add a
   `btn-primary` class only when `primary` is `true`.
5. Add a `Badge` with an inline style whose background color is passed in as a prop.

## Key Takeaways

- **Global CSS** (`app/globals.css`) applies everywhere; import it once in the layout.
- **CSS Modules** (`*.module.css`) give scoped class names that never clash.
- **Tailwind CSS** uses small **utility classes** in your markup; `create-next-app`
  can set it up automatically.
- **Inline styles** take an object and suit quick or dynamic one-offs.
- **Conditional classes** turn styles on and off; **`clsx`** makes that tidy.
- Pick one main approach and stay consistent.

---

[← Previous: Server Actions and Mutations](../03-rendering-and-data/05-server-actions.md) | [Back to Course Index](../README.md) | [Next: Images, Fonts, and Metadata →](02-images-fonts-metadata.md)
