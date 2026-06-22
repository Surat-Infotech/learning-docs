# Lists and Conditional Rendering

Real apps rarely show a fixed set of things. A to-do app shows however many
tasks you have. A shop shows however many products are in stock. And sometimes
the screen should change depending on a situation: show a "Welcome" message if
you are logged in, or a "Sign in" button if you are not. This lesson teaches you
two everyday skills: turning a list of data into UI, and showing UI only when a
condition is true.

## What You'll Learn

- How to turn an array into UI with **`.map()`**
- Why each list item needs a **`key`** prop
- How to choose a good key (and why the array index is a poor choice)
- Conditional rendering with the **`&&`** operator
- Conditional rendering with the **ternary** (`? :`) operator
- Using an **early return** to show a different UI
- How to render **nothing** with `null`
- Building a small **list of items** that combines these ideas

## Rendering a List with `.map()`

In JavaScript, `.map()` takes an array and builds a **new** array by transforming
each item. In React, we use it to turn an array of data into an array of JSX
elements.

```tsx
// app/page.tsx
export default function Page() {
  const fruits = ["Apple", "Banana", "Cherry"];

  return (
    <ul>
      {/* map each string into an <li> */}
      {fruits.map((fruit) => (
        <li>{fruit}</li>
      ))}
    </ul>
  );
}
// => a list with Apple, Banana, Cherry
```

The `{}` lets us drop JavaScript into JSX, and `.map()` returns an array of
`<li>` elements that React renders one after another.

## The `key` Prop

If you run the code above, React will warn you: **"Each child in a list should
have a unique key prop."** A **key** is a special prop that gives each item a
stable identity. React uses keys to track which items changed, were added, or
were removed, so it can update the screen efficiently.

```tsx
// app/page.tsx
export default function Page() {
  const fruits = ["Apple", "Banana", "Cherry"];

  return (
    <ul>
      {fruits.map((fruit) => (
        // the key must be unique among siblings
        <li key={fruit}>{fruit}</li>
      ))}
    </ul>
  );
}
```

### Choosing a Good Key

A key should be **unique** and **stable** (it should not change between renders).
The best key is usually an ID that comes with your data.

```tsx
// app/page.tsx
type Product = { id: number; name: string };

export default function Page() {
  const products: Product[] = [
    { id: 101, name: "Keyboard" },
    { id: 102, name: "Mouse" },
  ];

  return (
    <ul>
      {products.map((product) => (
        // use the stable id as the key
        <li key={product.id}>{product.name}</li>
      ))}
    </ul>
  );
}
```

Avoid using the array index (`.map((item, index) => ... key={index})`) as the
key when the list can change order, have items added, or removed. The index of
an item shifts around, which can confuse React and cause display bugs. An index
key is only safe for a fixed list that never changes.

## Conditional Rendering with `&&`

Often you want to show something **only if** a condition is true. The `&&`
operator is perfect for this. In JavaScript, `A && B` evaluates to `B` only when
`A` is true; otherwise it stops at `A`.

```tsx
// app/page.tsx
export default function Page() {
  const unreadCount = 3;

  return (
    <header>
      <h1>Inbox</h1>
      {/* show the badge ONLY when there are unread messages */}
      {unreadCount > 0 && <span className="badge">{unreadCount}</span>}
    </header>
  );
}
```

If `unreadCount > 0` is true, the `<span>` shows. If it is false, nothing shows.

**One gotcha:** do not use a plain number on the left of `&&`. If the value is
`0`, React will actually print "0" on the screen. Always compare to make a real
boolean, like `unreadCount > 0 &&`.

## Conditional Rendering with a Ternary

When you want to show **one thing or another** (an either/or choice), use the
**ternary** operator: `condition ? whenTrue : whenFalse`.

```tsx
// app/page.tsx
export default function Page({ isLoggedIn }: { isLoggedIn: boolean }) {
  return (
    <nav>
      {/* either a greeting OR a sign-in button */}
      {isLoggedIn ? <span>Welcome back!</span> : <button>Sign in</button>}
    </nav>
  );
}
```

You can read it as: "Is the user logged in? If yes, show the greeting;
otherwise, show the sign-in button."

## Early Return

When a whole component should look completely different based on a condition, an
**early return** is the cleanest approach. You return one UI and stop before
reaching the rest of the function.

```tsx
// app/components/UserPanel.tsx
type UserPanelProps = { user: { name: string } | null };

export default function UserPanel({ user }: UserPanelProps) {
  // if there is no user, return early with a different UI
  if (!user) {
    return <p>Please sign in to continue.</p>;
  }

  // this part only runs when a user exists
  return <h2>Hello, {user.name}!</h2>;
}
```

## Rendering Nothing with `null`

A component must always return something, but "something" can be **`null`**,
which tells React to render nothing at all.

```tsx
// app/components/Warning.tsx
type WarningProps = { show: boolean };

export default function Warning({ show }: WarningProps) {
  // render nothing when there is no warning to show
  if (!show) {
    return null;
  }

  return <p className="warning">Be careful!</p>;
}
```

## Putting It Together: A List of Items

Here is a small example that combines lists, keys, and conditional rendering.

```tsx
// app/page.tsx
type Task = { id: number; title: string; done: boolean };

export default function TaskList() {
  const tasks: Task[] = [
    { id: 1, title: "Write lesson", done: true },
    { id: 2, title: "Review code", done: false },
    { id: 3, title: "Ship feature", done: false },
  ];

  // handle the empty case with an early return
  if (tasks.length === 0) {
    return <p>No tasks yet. Enjoy your day!</p>;
  }

  return (
    <ul>
      {tasks.map((task) => (
        <li key={task.id}>
          {task.title}
          {/* show a checkmark only for finished tasks */}
          {task.done && <span> ✅</span>}
          {/* either "Done" or "Pending" */}
          <em> ({task.done ? "Done" : "Pending"})</em>
        </li>
      ))}
    </ul>
  );
}
```

This example shows the three core patterns side by side: `.map()` with a `key`
for the list, an early return for the empty case, `&&` for the optional
checkmark, and a ternary for the status label.

## Common Mistakes

- **Forgetting the `key` prop** on mapped items, which triggers a React warning
  and can cause display bugs.
- **Using the array index as a key** for lists that change. Prefer a stable ID.
- **Using a number with `&&`.** `{count && <X />}` prints `0` when `count` is
  `0`. Write `{count > 0 && <X />}` instead.
- **Putting an `if` statement inside JSX `{}`.** Only expressions work there; use
  a ternary, or an early return outside the JSX.
- **Returning nothing by accident.** To render nothing on purpose, return `null`.

## Exercises

1. Given an array of color strings, render them as a `<ul>` with a proper `key`
   on each `<li>`.
2. Change the array to objects with `{ id, name }` and use `id` as the key.
3. Render a "Cart" component that shows the item count with a badge **only when**
   the count is greater than zero (use `&&` correctly).
4. Write a `Status` component that takes an `online: boolean` prop and shows
   "Online" in one style or "Offline" in another, using a ternary.
5. Write a `NotificationDot` component that returns `null` when there are no
   notifications and a red dot otherwise.

## Key Takeaways

- Use **`.map()`** to turn an array of data into an array of JSX elements.
- Every mapped item needs a **unique, stable `key`** — prefer an ID over the
  index.
- Use **`&&`** to show something only when a condition is true (compare to make a
  real boolean).
- Use the **ternary** `? :` to choose between two UIs.
- Use an **early return** when the whole component should differ, and return
  **`null`** to render nothing.

---

[← Previous: Props and State](02-props-and-state.md) | [Back to Course Index](../README.md) | [Next: Events and Forms →](04-events-and-forms.md)
