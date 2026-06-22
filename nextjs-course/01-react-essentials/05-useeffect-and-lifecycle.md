# useEffect and the Component Lifecycle

So far, your components have done one job: take some props and state, and return
UI. That is pure and predictable. But real apps sometimes need to reach **outside**
that neat world: start a timer, listen for the keyboard, or fetch data from the
internet. These "reach outside" tasks are called **side effects**, and React
gives us a hook called **`useEffect`** to run them safely. Think of `useEffect`
as a note you leave for React: "after you finish drawing the screen, please also
do this for me."

## What You'll Learn

- What a **side effect** is and why it needs special handling
- How **`useEffect(fn, deps)`** works
- The **dependency array** and its three forms (`[]`, none, `[x]`)
- What a **cleanup function** is and when it runs
- Common uses: **timers**, **subscriptions**, and **fetching on the client**
- Why in Next.js you often **fetch on the server** instead (a forward reference)
- How to avoid the dreaded **infinite loop**

## Effects Need `"use client"`

`useEffect` only runs in the browser, so any component using it must be a
**Client Component**. Add the directive at the top of the file.

```tsx
// app/components/Timer.tsx
"use client"; // required: useEffect runs in the browser
```

## What Is a Side Effect?

A **side effect** is anything your component does that affects the world outside
of returning UI. Examples:

- Setting up a timer with `setInterval`
- Listening for browser events (scroll, resize, key presses)
- Fetching data from an API
- Reading or writing to `localStorage`

These should **not** run while React is calculating the UI, because they can be
slow or unpredictable. So we put them inside `useEffect`, and React runs them
**after** it has drawn the screen.

## The Shape of `useEffect`

`useEffect` takes two arguments: a **function** to run (the effect), and a
**dependency array** that controls *when* it runs.

```tsx
// app/components/Logger.tsx
"use client";

import { useEffect } from "react";

export default function Logger() {
  useEffect(() => {
    // this runs AFTER the component is shown on screen
    console.log("Component appeared on screen");
  }, []); // the dependency array

  return <p>Check the console.</p>;
}
```

## The Dependency Array

The second argument decides how often the effect runs. There are three cases.

### 1. Empty array `[]` — run once

With an empty array, the effect runs **once**, right after the component first
appears. This is perfect for one-time setup, like starting a timer or fetching
initial data.

```tsx
// runs a single time, when the component first mounts
useEffect(() => {
  console.log("Hello, I just appeared!");
}, []);
```

### 2. No array — run after every render

If you leave the array off entirely, the effect runs after **every** render.
This is rarely what you want and is a common source of bugs.

```tsx
// runs after the first render AND after every update — be careful
useEffect(() => {
  console.log("Rendered again");
});
```

### 3. With values `[x]` — run when they change

List values in the array and the effect runs after the first render **and**
whenever any listed value changes since the last render.

```tsx
// app/components/Title.tsx
"use client";

import { useEffect, useState } from "react";

export default function Title() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    // runs whenever `count` changes
    document.title = `Count: ${count}`;
  }, [count]); // depend on count

  return <button onClick={() => setCount(count + 1)}>Add ({count})</button>;
}
```

The rule of thumb: **list every value from your component that the effect uses.**
Here the effect uses `count`, so `count` belongs in the array.

## Cleanup Functions

Some effects set something up that must later be **torn down**: a timer must be
cleared, an event listener must be removed. If you do not clean up, you get
leftover timers and listeners piling up, which causes bugs and wasted memory.

To clean up, **return a function** from your effect. React runs it before the
effect runs again, and when the component is removed from the screen.

```tsx
// app/components/Clock.tsx
"use client";

import { useEffect, useState } from "react";

export default function Clock() {
  const [seconds, setSeconds] = useState(0);

  useEffect(() => {
    // setup: start a timer that ticks every second
    const id = setInterval(() => {
      setSeconds((prev) => prev + 1);
    }, 1000);

    // cleanup: stop the timer when the component goes away
    return () => clearInterval(id);
  }, []); // set up once

  return <p>Seconds: {seconds}</p>;
}
```

The cleanup is like turning off the tap when you leave the room: you opened it in
setup, so you close it in cleanup.

### Subscriptions

The same setup/cleanup pattern handles browser event listeners.

```tsx
// app/components/WindowWidth.tsx
"use client";

import { useEffect, useState } from "react";

export default function WindowWidth() {
  const [width, setWidth] = useState(0);

  useEffect(() => {
    function handleResize() {
      setWidth(window.innerWidth);
    }

    handleResize(); // set the initial value
    window.addEventListener("resize", handleResize); // subscribe

    // unsubscribe on cleanup
    return () => window.removeEventListener("resize", handleResize);
  }, []);

  return <p>Window width: {width}px</p>;
}
```

## Fetching Data on the Client

You can fetch data inside an effect. Because the effect's function cannot itself
be `async`, define an `async` function inside and call it.

```tsx
// app/components/UserName.tsx
"use client";

import { useEffect, useState } from "react";

export default function UserName({ id }: { id: number }) {
  const [name, setName] = useState<string | null>(null);

  useEffect(() => {
    async function load() {
      const res = await fetch(`/api/users/${id}`);
      const data = await res.json();
      setName(data.name);
    }
    load();
  }, [id]); // re-fetch when the id changes

  return <p>{name ?? "Loading..."}</p>;
}
```

## In Next.js, Prefer Fetching on the Server

Here is an important point for the App Router: even though you *can* fetch in
`useEffect`, you often **should not**. Because Server Components run on the
server, they can fetch data there directly, before the page is sent to the
browser. That is faster, more secure (secrets stay on the server), and needs no
loading spinner.

```tsx
// app/users/page.tsx
// a Server Component — no "use client", no useEffect needed
export default async function UsersPage() {
  // fetch directly on the server
  const res = await fetch("https://api.example.com/users");
  const users = await res.json();

  return (
    <ul>
      {users.map((u: { id: number; name: string }) => (
        <li key={u.id}>{u.name}</li>
      ))}
    </ul>
  );
}
```

You will learn this server-fetching approach in detail in a later module. The
takeaway for now: use `useEffect` for effects that **must** happen in the browser
(timers, listeners, reading `localStorage`), and lean on the server for plain
data fetching.

## Avoiding Infinite Loops

The classic `useEffect` bug is an **infinite loop**: an effect updates state,
which triggers a re-render, which runs the effect again, which updates state
again, forever.

```tsx
// app/components/Broken.tsx
"use client";

import { useEffect, useState } from "react";

export default function Broken() {
  const [count, setCount] = useState(0);

  // BAD: no dependency array, and the effect changes state every time
  useEffect(() => {
    setCount(count + 1); // triggers a re-render -> effect runs again -> loop!
  });

  return <p>{count}</p>;
}
```

The fixes: add a correct dependency array so the effect does not run on every
render, and make sure the effect does not endlessly change a value it depends
on. Most "run once" setup should use `[]`.

## Common Mistakes

- **Forgetting `"use client"`**, so `useEffect` never runs.
- **Leaving out the dependency array**, causing the effect to run on every render
  (and sometimes infinite loops).
- **Missing dependencies** — if the effect uses a value, list it, or it may use a
  stale value.
- **Forgetting cleanup** for timers and listeners, leaving leftovers behind.
- **Reaching for `useEffect` to fetch data** when a Server Component could fetch
  it more simply.
- **Making the effect function itself `async`** — define an inner async function
  and call it instead.

## Exercises

1. Build a component that logs "mounted" once when it appears (empty deps).
2. Make a counter that updates `document.title` to show the count whenever it
   changes (deps `[count]`).
3. Create a stopwatch using `setInterval`, and clear the interval in a cleanup
   function.
4. Add a window-size display that listens to the `resize` event and removes the
   listener on cleanup.
5. Take the broken infinite-loop example and fix it so the count starts at 1 and
   stops there.

## Key Takeaways

- A **side effect** is work outside returning UI (timers, listeners, fetching);
  put it in **`useEffect`**.
- The **dependency array** controls timing: `[]` runs once, no array runs every
  render, `[x]` runs when `x` changes.
- Return a **cleanup function** to tear down timers and subscriptions.
- Use `useEffect` for browser-only effects; in Next.js, **prefer fetching data on
  the server**.
- Watch out for **infinite loops** caused by effects that update their own
  dependencies without a proper array.

---

[← Previous: Events and Forms](04-events-and-forms.md) | [Back to Course Index](../README.md) | [Next: The App Router and File-Based Routing →](../02-routing/01-app-router-basics.md)
