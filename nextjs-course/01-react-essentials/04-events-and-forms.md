# Events and Forms

Up to now our components mostly displayed data. But apps need to **react** to the
user: a click, a key press, typing into a box, submitting a form. Think of your
component as a receptionist: when someone rings the bell (an event), the
receptionist does something in response (runs a function). This lesson teaches
you how to handle those events and how to build forms that stay in sync with
your state.

## What You'll Learn

- What an **event** is and how to attach an **event handler**
- The common handlers: **`onClick`**, **`onChange`**, **`onSubmit`**
- The difference between **passing** a function and **calling** it
- How to stop a form from reloading the page with **`preventDefault`**
- What a **controlled input** is (value + onChange + state)
- How to build a small **form** that collects a name and email
- The idea of **lifting state up** to share data between components

## Events Need `"use client"`

Event handling is interactive, so any component with events must be a **Client
Component**. Add the `"use client"` directive at the very top of the file.

```tsx
// app/components/ClickMe.tsx
"use client"; // required for interactivity
```

## Handling a Click with `onClick`

An **event handler** is a function that runs when something happens. You attach
it to an element with a prop like `onClick`.

```tsx
// app/components/ClickMe.tsx
"use client";

export default function ClickMe() {
  // this function runs when the button is clicked
  function handleClick() {
    alert("You clicked the button!");
  }

  return <button onClick={handleClick}>Click me</button>;
}
```

The handler names start with `on` and use camelCase: `onClick`, `onChange`,
`onSubmit`, `onMouseEnter`, and so on.

## Passing a Function vs Calling It

This is the single most common beginner mistake, so look carefully:

```tsx
// PASS the function — React calls it later, when the click happens
<button onClick={handleClick}>Good</button>

// CALL the function — runs immediately during render, NOT what you want
<button onClick={handleClick()}>Bad</button>
```

`onClick={handleClick}` gives React the function so it can run it **later**.
`onClick={handleClick()}` runs the function **right now** and hands React
whatever it returned, which is almost never what you want.

If you need to pass arguments, wrap the call in an **arrow function**. The arrow
function is the thing being passed; the call inside happens only on click.

```tsx
// app/components/Greeter.tsx
"use client";

export default function Greeter() {
  function greet(name: string) {
    alert(`Hello, ${name}!`);
  }

  // wrap in () => ... so it is passed, not called immediately
  return <button onClick={() => greet("Maya")}>Greet Maya</button>;
}
```

## Reading Input with `onChange`

The `onChange` event fires every time the user types in an input. React gives
the handler an **event object**, and the typed text lives at
`event.target.value`.

```tsx
// app/components/NameEcho.tsx
"use client";

import { useState } from "react";

export default function NameEcho() {
  const [name, setName] = useState("");

  return (
    <div>
      {/* update state on every keystroke */}
      <input onChange={(event) => setName(event.target.value)} />
      <p>Hello, {name || "stranger"}!</p>
    </div>
  );
}
```

## Controlled Inputs

In the example above, state follows the input. A **controlled input** goes one
step further: the input's `value` is also driven by state. State becomes the
single source of truth, and the input always shows exactly what is in state.

A controlled input has three parts working together:

1. A piece of **state** holds the value.
2. The input's **`value`** is set from that state.
3. The input's **`onChange`** updates that state.

```tsx
// app/components/ControlledName.tsx
"use client";

import { useState } from "react";

export default function ControlledName() {
  const [name, setName] = useState(""); // 1. state

  return (
    <input
      value={name} // 2. value comes from state
      onChange={(event) => setName(event.target.value)} // 3. onChange updates state
    />
  );
}
```

Why bother? Because the value lives in state, you can read it, validate it,
clear it, or pre-fill it whenever you like. The input and your data can never
drift apart.

## Submitting a Form with `onSubmit` and `preventDefault`

By default, submitting an HTML form reloads the whole page, the old way websites
worked. In a React app we want to handle the submission ourselves, so we stop
that default behavior with **`event.preventDefault()`**.

```tsx
// app/components/SearchForm.tsx
"use client";

import { useState } from "react";

export default function SearchForm() {
  const [query, setQuery] = useState("");

  function handleSubmit(event: React.FormEvent) {
    event.preventDefault(); // stop the page from reloading
    alert(`Searching for: ${query}`);
  }

  return (
    <form onSubmit={handleSubmit}>
      <input value={query} onChange={(e) => setQuery(e.target.value)} />
      <button type="submit">Search</button>
    </form>
  );
}
```

Put `onSubmit` on the `<form>`, not on the button. Pressing Enter in a field
also submits the form, and `onSubmit` catches both cases.

## A Full Form Example: Name and Email

Let's collect two fields and show the result when the form is submitted.

```tsx
// app/components/SignupForm.tsx
"use client";

import { useState } from "react";

export default function SignupForm() {
  const [name, setName] = useState("");
  const [email, setEmail] = useState("");
  const [submitted, setSubmitted] = useState(false);

  function handleSubmit(event: React.FormEvent) {
    event.preventDefault();
    setSubmitted(true); // remember that we submitted
  }

  return (
    <form onSubmit={handleSubmit}>
      <label>
        Name:
        <input value={name} onChange={(e) => setName(e.target.value)} />
      </label>

      <label>
        Email:
        <input
          type="email"
          value={email}
          onChange={(e) => setEmail(e.target.value)}
        />
      </label>

      <button type="submit">Sign up</button>

      {/* show a confirmation only after submitting */}
      {submitted && (
        <p>
          Thanks, {name}! We will email you at {email}.
        </p>
      )}
    </form>
  );
}
```

Each input is controlled by its own state. On submit we prevent the reload and
flip `submitted` to `true`, which causes the confirmation message to appear.

## Lifting State Up (Brief)

Sometimes two components need to share the same data. For example, an input and a
live preview. The rule is: **move the state up to their closest common parent**,
then pass the value down as props and pass a setter down so the child can update
it. This is called **lifting state up**.

```tsx
// app/components/NameTool.tsx
"use client";

import { useState } from "react";

function NameInput({ value, onChange }: {
  value: string;
  onChange: (next: string) => void;
}) {
  return <input value={value} onChange={(e) => onChange(e.target.value)} />;
}

function Preview({ value }: { value: string }) {
  return <p>Preview: {value}</p>;
}

export default function NameTool() {
  // state lives in the PARENT and is shared with both children
  const [name, setName] = useState("");

  return (
    <div>
      <NameInput value={name} onChange={setName} />
      <Preview value={name} />
    </div>
  );
}
```

The parent owns the state. The input changes it through the `onChange` prop, and
the preview reads it through the `value` prop. Both children stay in sync because
they share one source of truth. You will use this pattern constantly.

## Common Mistakes

- **Calling instead of passing** the handler: `onClick={fn()}` runs immediately.
  Use `onClick={fn}` or `onClick={() => fn(arg)}`.
- **Forgetting `"use client"`**, which makes event handlers fail in the App
  Router.
- **Forgetting `preventDefault`** in a submit handler, so the page reloads and
  your code seems to do nothing.
- **A controlled input with `value` but no `onChange`**, which makes the field
  read-only and impossible to type in.
- **Putting `onSubmit` on the button** instead of the `<form>`, missing the
  Enter-key submit.

## Exercises

1. Make a button that, when clicked, increases a counter shown next to it.
2. Build a controlled text input that shows the number of characters typed below
   it, live.
3. Create a form with a single "message" field. On submit, prevent the reload
   and display the message inside a `<blockquote>`.
4. Extend the signup form to include a "phone" field, all controlled.
5. Use lifting state up: a parent holds a `color` string; one child input changes
   it, and another child shows a box whose `backgroundColor` is that color.

## Key Takeaways

- **Event handlers** are functions attached with props like `onClick`,
  `onChange`, `onSubmit` — and the component must be a Client Component.
- **Pass** a handler (`onClick={fn}`); wrap in an arrow to pass arguments.
- Call **`event.preventDefault()`** in submit handlers to stop a page reload.
- A **controlled input** ties `value` and `onChange` to state, making state the
  single source of truth.
- **Lifting state up** puts shared state in the common parent and passes it down
  as props.

---

[← Previous: Lists and Conditional Rendering](03-lists-and-conditional-rendering.md) | [Back to Course Index](../README.md) | [Next: useEffect and Lifecycle →](05-useeffect-and-lifecycle.md)
