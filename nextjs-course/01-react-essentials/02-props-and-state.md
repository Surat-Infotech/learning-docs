# Props and State

A component is more useful when you can feed it different data. Imagine a coffee
machine: it is one machine, but you give it a different setting each time
(espresso, latte, black coffee) and it produces a different result. That
"setting you pass in" is called **props**. And sometimes a component needs to
**remember** something that changes over time, like how many cups it has made
today. That memory is called **state**. This lesson covers both.

## What You'll Learn

- What **props** are and how to pass data into a component
- Why props are **read-only** (a component must not change them)
- How to **type props** with TypeScript so mistakes are caught early
- Why interactivity needs the **`"use client"`** directive
- What **state** is and how `useState` works
- How to **read** and **update** state with its setter
- Why you must **never mutate state directly**
- How updating state causes a **re-render**

## What Are Props?

**Props** (short for "properties") are the data you pass into a component. You
write them like HTML attributes when you use the component.

```tsx
// app/page.tsx
function Greeting(props) {
  // props is an object: { name: "Maya" }
  return <h1>Hello, {props.name}!</h1>;
}

export default function Home() {
  // we pass name="Maya" into Greeting
  return <Greeting name="Maya" />;
}
// => "Hello, Maya!"
```

The component receives all the attributes as a single object called `props`.
Here `props.name` is `"Maya"`.

### Destructuring Props

It is common to pull values out of the props object right in the function
parameter. This is called **destructuring** and makes the code shorter.

```tsx
// app/page.tsx
function Greeting({ name }) {
  // pull "name" straight out of props
  return <h1>Hello, {name}!</h1>;
}
```

You can pass any kind of value, not just text. Use `{}` for non-string values:

```tsx
// app/page.tsx
function Price({ amount, onSale }) {
  return (
    <p>
      Costs ${amount}. {onSale ? "On sale!" : "Full price."}
    </p>
  );
}

export default function Home() {
  // numbers and booleans go inside {}
  return <Price amount={19} onSale={true} />;
}
```

## Props Are Read-Only

A component must **never change its own props**. Props belong to the parent that
passed them in. Think of props like a gift wrapped by someone else: you can look
at it and use it, but you should not rewrite the label.

```tsx
// app/page.tsx
function Greeting({ name }) {
  name = "Someone Else"; // BAD: do not reassign props
  return <h1>Hello, {name}!</h1>;
}
```

If a component needs a value that changes, that is what **state** is for (coming
up below).

## Typing Props with TypeScript

Because Next.js uses TypeScript, you can describe the **shape** of your props.
This catches mistakes, like forgetting a prop or passing the wrong type, before
you even run the app.

```tsx
// app/components/Price.tsx
type PriceProps = {
  amount: number; // must be a number
  onSale: boolean; // must be true or false
};

function Price({ amount, onSale }: PriceProps) {
  return (
    <p>
      Costs ${amount}. {onSale ? "On sale!" : "Full price."}
    </p>
  );
}
```

Now if you write `<Price amount="cheap" />`, TypeScript warns you, because
`"cheap"` is text, not a number. Optional props get a `?`:

```tsx
// app/components/Badge.tsx
type BadgeProps = {
  label: string;
  color?: string; // optional: may be left out
};

function Badge({ label, color = "gray" }: BadgeProps) {
  // default to "gray" if color was not passed
  return <span className={`badge ${color}`}>{label}</span>;
}
```

## Adding Interactivity: `"use client"`

So far our components only display data. To make a component **interactive**
(respond to clicks, remember changing values), it must run in the browser. In
the App Router, components run on the server by default, so you must opt in to a
**Client Component** by putting the directive `"use client"` at the very top of
the file.

```tsx
// app/components/Counter.tsx
"use client"; // this line makes the whole file a Client Component
```

Anything that uses state or event handlers needs this line. Without it, you will
get an error.

## What Is State?

**State** is data a component remembers between renders, and that can change over
time. When a number on the screen needs to go up when you click a button, that
number lives in state.

React gives us a tool called **`useState`** to create state. Tools like this
that start with `use` are called **hooks**.

```tsx
// app/components/Counter.tsx
"use client";

import { useState } from "react";

function Counter() {
  // useState returns a pair: [currentValue, functionToUpdateIt]
  const [count, setCount] = useState(0); // start at 0

  return <p>The count is {count}.</p>;
}
```

Let's break down `const [count, setCount] = useState(0);`:

- `useState(0)` creates a piece of state starting at `0`.
- It returns an array of two things.
- `count` is the **current value** (read it to display).
- `setCount` is the **setter function** (call it to change the value).
- The names are up to you, but the pattern `[thing, setThing]` is standard.

## Updating State with the Setter

You change state by **calling the setter**. You never assign to the variable
directly.

```tsx
// app/components/Counter.tsx
"use client";

import { useState } from "react";

export default function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>The count is {count}.</p>
      {/* clicking calls setCount, which updates the state */}
      <button onClick={() => setCount(count + 1)}>Add one</button>
    </div>
  );
}
```

When you call `setCount(count + 1)`, React updates the value **and** re-runs the
component to show the new number. This re-running is called a **re-render**.

### Updating Based on the Previous Value

If the new value depends on the old value, pass a function to the setter. React
hands you the latest value, which avoids subtle bugs.

```tsx
// inside Counter
// safer when updates may happen quickly
<button onClick={() => setCount((prev) => prev + 1)}>Add one</button>
```

## Never Mutate State Directly

This will **not** work:

```tsx
// app/components/Counter.tsx
count = count + 1; // BAD: React does not notice this change
```

Changing the variable directly does not tell React to re-render, so the screen
will not update. **Always** go through the setter. The same rule applies to
objects and arrays in state: make a new copy instead of editing in place.

```tsx
// updating an object in state: build a NEW object
const [user, setUser] = useState({ name: "Maya", age: 25 });

// GOOD: copy the old fields with ..., then change one
setUser({ ...user, age: 26 });
```

## How Re-renders Work

Each time state changes, React runs your component function again from the top
and compares the new UI to the old one, updating only what changed on the
screen. Your component is like a recipe: when an ingredient (state) changes,
React re-cooks the dish and serves the updated version. You describe *what* the
UI should look like for a given state, and React handles the *how*.

## Common Mistakes

- **Forgetting `"use client"`.** Any component using `useState` or events must
  have this directive at the top of its file.
- **Trying to change props.** Props are read-only; use state for changing data.
- **Assigning to the state variable** (`count = 5`) instead of calling the
  setter (`setCount(5)`). The screen will not update.
- **Mutating objects/arrays in state** with `push` or by setting a field. Make a
  fresh copy with `...` first.
- **Calling the setter during render** (outside an event handler), which causes
  an infinite loop of re-renders.

## Exercises

1. Write a `Greeting` component that takes a `name` prop (typed as `string`) and
   shows "Welcome, NAME". Use it twice with different names.
2. Create a typed `UserCard` component with props `name: string`,
   `age: number`, and an optional `city?: string` that defaults to `"Unknown"`.
3. Build a `Counter` Client Component with two buttons: one that adds 1 and one
   that subtracts 1. Remember the `"use client"` directive.
4. Add a "Reset" button to your counter that sets the count back to 0.
5. Make a `Toggle` component that flips a boolean state between `true` and
   `false` and shows "ON" or "OFF" accordingly.

## Key Takeaways

- **Props** pass data into a component and are **read-only**.
- TypeScript lets you **type props** to catch mistakes early.
- Interactivity requires the **`"use client"`** directive at the top of the file.
- **`useState`** returns `[value, setter]`; read the value, call the setter to
  change it.
- **Never mutate state directly** — always use the setter, and copy objects and
  arrays.
- Updating state triggers a **re-render** that refreshes the screen.

---

[← Previous: Components and JSX](01-components-and-jsx.md) | [Back to Course Index](../README.md) | [Next: Lists and Conditional Rendering →](03-lists-and-conditional-rendering.md)
