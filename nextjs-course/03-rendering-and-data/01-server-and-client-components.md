# Server Components vs Client Components

In the Next.js App Router there are two kinds of components: **Server
Components** and **Client Components**. Knowing which is which is the single most
important idea in this whole course, so we will take it slowly.

Think of a restaurant. The **kitchen** (the server) does heavy work behind the
scenes: it reads recipes, opens the fridge (the database), and cooks the food.
The **dining table** (the browser) is where you actually interact: you click,
you type, you react. Server Components are the kitchen. Client Components are the
table. Most of your app is kitchen work, and only a little of it needs to happen
at the table.

## What You'll Learn

- That Server Components are the **default** in the App Router
- What a Server Component can do (run async code, read a database, use secrets)
- Why Server Components ship **no JavaScript** to the browser
- When you need a Client Component and how to opt in with `"use client"`
- The four signs you need a Client Component (state, effects, events, browser APIs)
- The one-way rule: you can put Client Components inside Server Components, but not the reverse
- How to pass server data down into Client Components using props and `children`

## Server Components Are the Default

In the App Router, **every component is a Server Component unless you say
otherwise**. You do not type anything special to make one. This file is a Server
Component just by living in the `app` folder.

```tsx
// app/page.tsx
// No "use client" at the top, so this runs on the SERVER.

export default function HomePage() {
  return <h1>Welcome to my shop</h1>;
}
```

"Runs on the server" means the code executes on the computer in the data center,
**before** anything reaches the user's browser. The browser receives finished
HTML, not the code that made it.

### Server Components can be async

Because they run on the server, Server Components are allowed to be `async`
functions. This lets you `await` things like a database call right inside the
component.

```tsx
// app/page.tsx
// Notice the word "async" in front of the function.

export default async function HomePage() {
  // We can await directly here. More on fetching in the next lesson.
  const res = await fetch("https://api.example.com/products");
  const products = await res.json();

  return <p>We have {products.length} products.</p>;
}
```

A regular React component in the browser can **not** be `async` like this. This
ability is special to Server Components.

### Server Components can touch secrets and databases

Since the code never goes to the browser, it is safe to use secret keys and talk
straight to your database.

```tsx
// app/dashboard/page.tsx
import { db } from "@/lib/db"; // an imaginary database helper

export default async function Dashboard() {
  // Safe: this query and any secret keys stay on the server.
  const users = await db.user.findMany();

  return <p>{users.length} users signed up.</p>;
}
```

If you tried this in the browser, anyone could open the developer tools and steal
your database password. On the server, nobody outside ever sees it.

### Server Components ship no JavaScript

This is the big performance win. The browser gets HTML for a Server Component,
not its code. Less JavaScript means a faster page and a quicker first load.

```text
Server Component  ->  runs in the data center  ->  sends HTML to browser
                                                    (browser ships 0 JS for it)
```

## When You Need a Client Component

Server Components are great, but they run once on the server and then they are
"done". They can not respond to clicks or remember things as the user pokes at
the page. For that, you need a **Client Component**.

You opt in by writing `"use client"` as the very first line of the file.

```tsx
// app/components/Counter.tsx
"use client"; // <- this line makes it a Client Component

import { useState } from "react";

export default function Counter() {
  const [count, setCount] = useState(0); // state needs a Client Component

  return (
    <button onClick={() => setCount(count + 1)}>
      Clicked {count} times
    </button>
  );
}
```

### The four signs you need `"use client"`

Reach for a Client Component when you need any of these:

- **State** with `useState` or `useReducer` (remembering values over time)
- **Effects** with `useEffect` (running code after the page shows)
- **Event handlers** like `onClick`, `onChange`, `onSubmit`
- **Browser-only APIs** like `window`, `localStorage`, or `navigator`

If your component just shows data and has none of these, leave it as a Server
Component.

```tsx
// app/components/Greeting.tsx
// No state, no clicks -> keep it a Server Component (the default).

export default function Greeting({ name }: { name: string }) {
  return <p>Hello, {name}!</p>;
}
```

## The One-Way Rule

You can **import a Client Component into a Server Component**. That is normal and
common.

```tsx
// app/page.tsx  (Server Component)
import Counter from "./components/Counter"; // Counter is a Client Component

export default function HomePage() {
  return (
    <main>
      <h1>My page</h1>
      <Counter /> {/* allowed: client inside server */}
    </main>
  );
}
```

But you can **not** import a Server Component into a Client Component. Once a file
says `"use client"`, everything it imports is treated as client code too, and
client code can not run server-only things like database queries.

### How to share server data with a Client Component

Since the Client Component can not fetch the data itself in a server-safe way,
the Server Component fetches it and **passes it down as props**.

```tsx
// app/page.tsx  (Server Component)
import LikeButton from "./components/LikeButton";

export default async function HomePage() {
  const post = await getPost(); // server fetch

  // Hand the data to the client component through a prop.
  return <LikeButton initialLikes={post.likes} />;
}
```

```tsx
// app/components/LikeButton.tsx  (Client Component)
"use client";
import { useState } from "react";

export default function LikeButton({ initialLikes }: { initialLikes: number }) {
  const [likes, setLikes] = useState(initialLikes); // starts from server value

  return <button onClick={() => setLikes(likes + 1)}>{likes} likes</button>;
}
```

### Passing Server Components as `children`

There is a neat trick: a Client Component can render a Server Component if it
receives it as `children`. The Server Component is rendered on the server first,
then "slotted into" the client wrapper.

```tsx
// app/page.tsx  (Server Component)
import Panel from "./components/Panel";        // Client Component
import ServerInfo from "./components/ServerInfo"; // Server Component

export default function HomePage() {
  return (
    <Panel>
      {/* ServerInfo is rendered on the server, then placed inside Panel */}
      <ServerInfo />
    </Panel>
  );
}
```

## A Mental Model

```text
              THE PAGE
   +-----------------------------+
   |   Server Component (kitchen)|   <- default, async OK, DB/secrets OK,
   |                             |      ships HTML only, no JS to browser
   |   passes data down as props |
   |          |                  |
   |          v                  |
   |   +---------------------+   |
   |   | Client Component    |   |   <- "use client", has state/clicks,
   |   | (the dining table)  |   |      ships JS to the browser
   |   +---------------------+   |
   +-----------------------------+

   Arrows only point DOWN: server -> client. Never client -> server import.
```

## Common Mistakes

- **Adding `"use client"` everywhere "just in case".** This drags more code into
  the browser and loses the performance benefit. Add it only when you truly need
  state, effects, events, or browser APIs.
- **Trying to `await fetch` inside a Client Component as if it were a Server
  Component.** Client Components can not be `async`; use the patterns from later
  lessons instead.
- **Using `useState` or `onClick` in a file with no `"use client"`.** You will
  get an error. The fix is to add the directive or move that logic into a small
  Client Component.
- **Putting a database call in a Client Component.** Your secrets would leak to
  the browser. Keep data access on the server and pass results down.
- **Forgetting that `"use client"` is contagious.** Everything imported by a
  client file becomes client code too.

## Exercises

1. Create `app/page.tsx` as a Server Component that shows a heading. Confirm it
   works with no `"use client"` line.
2. Build a `Counter` Client Component with a button that increases a number.
   Import it into your Server Component page.
3. In a Server Component, define a value (for example `const name = "Sam"`) and
   pass it as a prop into a Client Component that displays it.
4. Take a small component you wrote and decide out loud: does it need to be a
   Client Component? List which of the four signs (if any) apply.
5. Try importing a Server Component directly into a `"use client"` file and read
   the error message. Then fix it by passing the Server Component as `children`.

## Key Takeaways

- Server Components are the **default**; you opt into Client Components with
  `"use client"`.
- Server Components can be `async`, can read databases and secrets, and ship no
  JavaScript to the browser.
- Use a Client Component when you need state, effects, event handlers, or
  browser APIs.
- You can put Client Components inside Server Components, but not the other way
  around.
- Share server data with Client Components by passing it down as props (or render
  Server Components through `children`).

---

[← Previous: Loading and Error UI](../02-routing/06-loading-and-error-ui.md) | [Back to Course Index](../README.md) | [Next: Data Fetching →](02-data-fetching.md)
