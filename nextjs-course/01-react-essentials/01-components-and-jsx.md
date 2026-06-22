# Components and JSX

Every website you see is made of small visual pieces: a button, a search bar,
a navigation menu, a card showing a photo. React lets you build your app out of
these small reusable pieces called **components**. Think of components like
**LEGO bricks**: each brick is simple on its own, but you snap many of them
together to build something big. And once you make a brick, you can reuse it
anywhere.

## What You'll Learn

- What a **component** is and why React uses them
- What **JSX** is and why it looks like HTML but is really JavaScript
- Why we write `className` instead of `class`
- How to embed JavaScript inside JSX using curly braces `{}`
- Why a component must return **one root element** (and how fragments help)
- How to **nest** components inside other components
- How to **export** a component so other files can use it
- A gentle note about **Server Components** in the App Router

## What Is a Component?

A **component** is just a JavaScript function that returns some UI (user
interface). "UI" means the visual stuff on the screen.

```tsx
// app/page.tsx
function Welcome() {
  // this function returns what should appear on the screen
  return <h1>Hello, world!</h1>;
}
```

That is a complete component. Its name is `Welcome`, and it returns a heading.

Two important rules for components:

- The name **must start with a capital letter** (`Welcome`, not `welcome`).
  React uses the capital letter to tell your components apart from plain HTML
  tags like `<h1>`.
- It must **return** something to show (or `null` to show nothing).

## What Is JSX?

That `<h1>Hello, world!</h1>` inside the function looks like HTML, but it is
not. It is **JSX** (JavaScript XML). JSX is a special syntax that lets you write
HTML-like code directly inside JavaScript. A tool built into Next.js translates
it into regular JavaScript for you.

So JSX *looks* like HTML, but it *is* JavaScript. That mix is what makes React
powerful: you can use real JavaScript logic right next to your markup.

```tsx
// app/page.tsx
function Tag() {
  // looks like HTML, but lives inside a .tsx (JavaScript) file
  return <span>I am JSX</span>;
}
```

### `className` Instead of `class`

In normal HTML you add a CSS class with the `class` attribute. But `class` is
already a reserved word in JavaScript (it means something else). So in JSX you
write `className` instead.

```tsx
// app/page.tsx
function Box() {
  // use className, NOT class
  return <div className="card">A styled box</div>;
}
```

A few other attributes change name too: `for` becomes `htmlFor`. Most others
stay the same.

### Embedding JavaScript with `{}`

The real magic of JSX is that you can drop JavaScript into your markup using
**curly braces** `{}`. Anything inside `{}` is treated as a JavaScript
expression, and its result is shown on screen.

```tsx
// app/page.tsx
function Sum() {
  const name = "Maya";
  const year = 2026;

  return (
    <p>
      {name} says the year is {year}. Next year is {year + 1}.
    </p>
  );
}
// => "Maya says the year is 2026. Next year is 2027."
```

You can put any expression inside `{}`: a variable, math, a function call, or
even another piece of JSX. You **cannot** put a statement like an `if` or a
`for` loop there (those are not expressions). We will learn how to handle that
in a later lesson.

## One Root Element

A component can only return **one** top-level element. This is wrong:

```tsx
// app/page.tsx
function Broken() {
  // ERROR: two elements side by side at the top
  return (
    <h1>Title</h1>
    <p>Some text</p>
  );
}
```

The fix is to wrap them in a single parent. Often a `<div>` works:

```tsx
// app/page.tsx
function Fixed() {
  return (
    <div>
      <h1>Title</h1>
      <p>Some text</p>
    </div>
  );
}
```

### Fragments `<>`

Sometimes you do not want an extra `<div>` in your page. A **fragment** lets you
group elements without adding any real tag to the page. You write it as an empty
pair of tags: `<>` and `</>`.

```tsx
// app/page.tsx
function Clean() {
  // the fragment groups the elements but adds nothing to the HTML
  return (
    <>
      <h1>Title</h1>
      <p>Some text</p>
    </>
  );
}
```

## Nesting Components

The whole point of components is to combine them. You use a component by writing
its name as if it were an HTML tag.

```tsx
// app/page.tsx
function Header() {
  return <h1>My Site</h1>;
}

function Footer() {
  return <footer>© 2026</footer>;
}

function Page() {
  // we nest Header and Footer inside Page
  return (
    <div>
      <Header />
      <p>Welcome to my site.</p>
      <Footer />
    </div>
  );
}
```

Notice the self-closing form `<Header />`. If a component has no children
between its tags, you close it with `/>`.

## Exporting a Component

In the App Router, the file `app/page.tsx` becomes a webpage. The component you
want Next.js to show on that page must be the **default export**.

```tsx
// app/page.tsx
export default function Home() {
  // this is the page Next.js will render at "/"
  return <h1>Home Page</h1>;
}
```

`export default` means "this is the main thing this file gives to other files."
Each file can have only one default export.

## A Simple Greeting Example

Let's put the ideas together: a `Greeting` component used inside the page.

```tsx
// app/page.tsx
function Greeting() {
  const hour = 9; // pretend it is 9 AM
  const message = hour < 12 ? "Good morning" : "Hello";

  return (
    <section className="greeting">
      <h2>{message}!</h2>
      <p>Welcome back to the dashboard.</p>
    </section>
  );
}

export default function Home() {
  return (
    <main>
      <Greeting />
    </main>
  );
}
```

## A Note on Server Components

In the Next.js App Router, the components you write are **Server Components by
default**. That means React runs them on the server (not in the visitor's
browser) and sends ready-made HTML to the page. You do not need to do anything
special for this; it just happens.

For now, do not worry about it. Just know that all the examples above are
Server Components. In the next lesson, you will meet a special case where you
need to opt in to a **Client Component** to add interactivity.

## Common Mistakes

- **Lowercase component names.** `<greeting />` will be treated as an unknown
  HTML tag. Always capitalize: `<Greeting />`.
- **Using `class` instead of `className`.** This is the most common JSX typo.
- **Returning two elements at the top level.** Wrap them in a `<div>` or a
  fragment `<>`.
- **Forgetting `{}` around JavaScript.** Writing `<p>year + 1</p>` shows the
  literal text "year + 1". You need `<p>{year + 1}</p>`.
- **Forgetting the default export** on a page file, which causes a Next.js
  error that nothing is being exported.

## Exercises

1. Create a component called `ProfileCard` that returns a `<div>` containing
   your name in an `<h2>` and a short bio in a `<p>`. Give the `<div>` a
   `className` of `"profile"`.
2. Inside `ProfileCard`, declare a variable `age` and show the sentence
   "Next year I will be X" where X is `age + 1`, using `{}`.
3. Make a `Page` component that nests `ProfileCard` twice. Use a fragment `<>`
   instead of a `<div>` as the wrapper.
4. Create a `Greeting` component that picks "Good morning" or "Good evening"
   based on a `const hour` value, and display it inside `Page`.

## Key Takeaways

- A **component** is a function that returns UI; its name must be capitalized.
- **JSX** looks like HTML but is JavaScript that Next.js translates for you.
- Use `className` instead of `class`, and `{}` to embed JavaScript.
- A component returns **one root element**; use a fragment `<>` to avoid extra
  tags.
- You **nest** components by writing them like tags, and a page needs a
  **default export**.
- App Router components are **Server Components by default** — no setup needed.

---

[← Previous: Setup and Project Structure](../00-getting-started/02-setup-and-project-structure.md) | [Back to Course Index](../README.md) | [Next: Props and State →](02-props-and-state.md)
