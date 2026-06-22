# Server Actions and Mutations

So far we have only **read** data. But real apps also **change** data: adding a
todo, posting a comment, updating a profile. In old setups this meant writing a
separate API route and calling it with `fetch`. Next.js gives you a shortcut
called a **Server Action**: a function that runs on the server but that you can
call straight from a form or a button, no API route needed.

Think of a Server Action like a kitchen order slip. You (the form) fill it out
and hand it through the window. The kitchen (the server) does the work, then sends
back the result. You never had to learn the kitchen's internal phone number; you
just passed the slip.

## What You'll Learn

- What a Server Action is and how `"use server"` marks one
- Calling an action straight from a `<form action={...}>`
- Reading submitted fields with `FormData`
- Mutating data and then refreshing the page with `revalidatePath`
- Calling a Server Action from a Client Component
- Showing a pending state with `useFormStatus`
- Validating input and returning a friendly error message
- A complete worked "add todo" example

## What Is a Server Action?

A Server Action is an `async` function that **runs on the server**, marked with
the `"use server"` directive. Because it runs on the server, it can safely touch
your database and secrets, just like a Server Component.

You can mark a single function inline, or a whole file.

```tsx
// app/actions.ts
"use server"; // every exported function in this file is a Server Action

export async function sayHello() {
  console.log("This runs on the server!");
}
```

The magic part: you can hand this function directly to a form, and Next.js wires
up the network call for you.

## Using an Action in a Form

A `<form>` can take a Server Action as its `action`. When the form is submitted,
Next.js runs the action on the server and passes it the form's data.

```tsx
// app/page.tsx
import { addTodo } from "./actions";

export default function HomePage() {
  return (
    // On submit, Next.js calls addTodo on the server.
    <form action={addTodo}>
      <input name="title" placeholder="New todo" />
      <button type="submit">Add</button>
    </form>
  );
}
```

No `onSubmit`, no `fetch`, no API route. The form just points at the action.

## Reading `FormData`

When a form calls an action, the action receives a `FormData` object: a bag of
the form's fields. You read a field by its `name` using `.get()`.

```tsx
// app/actions.ts
"use server";

export async function addTodo(formData: FormData) {
  // Read the input whose name="title".
  const title = formData.get("title") as string;

  console.log("New todo:", title);
}
```

The `name` attribute on each input is the key you use here, so `name="title"` in
the form matches `formData.get("title")` in the action.

## Mutating Data, Then Revalidating

The whole point is to **change** data. After you write to the database, the cached
page might still show the old data, so you call `revalidatePath` to refresh it
(this is the on-demand revalidation from the
[Caching lesson](03-caching-and-revalidation.md)).

```tsx
// app/actions.ts
"use server";
import { revalidatePath } from "next/cache";
import { db } from "@/lib/db";

export async function addTodo(formData: FormData) {
  const title = formData.get("title") as string;

  // 1. Change the data.
  await db.todo.create({ data: { title } });

  // 2. Tell Next.js the "/" page's data is stale, so it shows the new todo.
  revalidatePath("/");
}
```

The flow is always: **mutate, then revalidate.** Without the revalidate step, the
user might submit a new todo but not see it appear.

## Calling an Action From a Client Component

Forms are the easy case. But you can also call a Server Action from inside a
Client Component, for example from a button's `onClick`.

```tsx
// app/components/DeleteButton.tsx
"use client";
import { deleteTodo } from "../actions";

export default function DeleteButton({ id }: { id: number }) {
  return (
    // Call the server action directly inside the click handler.
    <button onClick={() => deleteTodo(id)}>Delete</button>
  );
}
```

```tsx
// app/actions.ts
"use server";
import { revalidatePath } from "next/cache";
import { db } from "@/lib/db";

export async function deleteTodo(id: number) {
  await db.todo.delete({ where: { id } });
  revalidatePath("/");
}
```

Even though the button lives in the browser, calling `deleteTodo` runs the code
on the server. Next.js handles the network round-trip behind the scenes.

## Showing a Pending State

While the action is running, it is nice to disable the button and say "Saving...".
React gives you `useFormStatus` for this. It must be used in a Client Component
that is **inside** the form.

```tsx
// app/components/SubmitButton.tsx
"use client";
import { useFormStatus } from "react-dom";

export default function SubmitButton() {
  // pending is true while the form's action is running.
  const { pending } = useFormStatus();

  return (
    <button type="submit" disabled={pending}>
      {pending ? "Saving..." : "Add"}
    </button>
  );
}
```

Drop `<SubmitButton />` inside your `<form>` and the button automatically shows
the pending state. No manual loading flag needed.

## Validation and Returning Errors

You should never trust input blindly. Validate it in the action, and if something
is wrong, **return** an error object that the page can show. To display the
returned value in a form, use the `useActionState` hook (in a Client Component).

```tsx
// app/actions.ts
"use server";
import { revalidatePath } from "next/cache";
import { db } from "@/lib/db";

// prevState is whatever this action returned last time; we can ignore it here.
export async function addTodo(prevState: unknown, formData: FormData) {
  const title = (formData.get("title") as string)?.trim();

  // Validate: reject empty titles with a friendly message.
  if (!title) {
    return { error: "Please enter a todo." };
  }

  await db.todo.create({ data: { title } });
  revalidatePath("/");

  return { error: null }; // success
}
```

```tsx
// app/components/TodoForm.tsx
"use client";
import { useActionState } from "react";
import { addTodo } from "../actions";
import SubmitButton from "./SubmitButton";

export default function TodoForm() {
  // useActionState wires the action to the form and tracks its return value.
  const [state, formAction] = useActionState(addTodo, { error: null });

  return (
    <form action={formAction}>
      <input name="title" placeholder="New todo" />
      <SubmitButton />
      {/* Show the error message the action returned, if any. */}
      {state.error && <p style={{ color: "red" }}>{state.error}</p>}
    </form>
  );
}
```

## Worked Example: A Tiny Todo App

Putting it all together: a list of todos (Server Component) plus a form that adds
one (using a Server Action).

```tsx
// app/actions.ts
"use server";
import { revalidatePath } from "next/cache";
import { db } from "@/lib/db";

export async function addTodo(prevState: unknown, formData: FormData) {
  const title = (formData.get("title") as string)?.trim();
  if (!title) return { error: "Please enter a todo." };

  await db.todo.create({ data: { title } });
  revalidatePath("/");           // refresh the list so the new todo shows
  return { error: null };
}
```

```tsx
// app/page.tsx  (Server Component)
import { db } from "@/lib/db";
import TodoForm from "./components/TodoForm";

export default async function HomePage() {
  // Read the current todos on the server.
  const todos = await db.todo.findMany();

  return (
    <main>
      <h1>My Todos</h1>

      {/* The client form that calls the server action */}
      <TodoForm />

      <ul>
        {todos.map((t) => (
          <li key={t.id}>{t.title}</li>
        ))}
      </ul>
    </main>
  );
}
```

When the user submits, `addTodo` runs on the server, writes to the database,
revalidates `/`, and the list re-renders with the new todo, all without you
writing a single API route or `fetch` call.

## Common Mistakes

- **Forgetting `"use server"`.** Without it, the function is not a Server Action
  and the form wiring will not work.
- **Skipping `revalidatePath` after a mutation.** The data changes but the cached
  page keeps showing the old version, so nothing seems to happen.
- **Mismatching `name` and `formData.get(...)`.** If the input is `name="title"`
  but you read `formData.get("name")`, you get `null`.
- **Trusting input without validation.** Always check and clean the data on the
  server; the browser can send anything.
- **Calling `useFormStatus` outside the form.** It only works in a Client
  Component rendered inside the `<form>` it reports on.

## Exercises

1. Write a Server Action `addTodo` that reads a `title` from `FormData` and logs
   it. Wire it to a form with `<form action={addTodo}>`.
2. Connect a real or fake database call inside the action, then add
   `revalidatePath("/")` and confirm the list updates after submitting.
3. Add a `SubmitButton` Client Component that uses `useFormStatus` to show
   "Saving..." while the action runs.
4. Add validation that rejects an empty title and returns `{ error: "..." }`,
   then display that error using `useActionState`.
5. Add a `deleteTodo(id)` action and call it from a `"use client"` delete button's
   `onClick`, revalidating the list afterward.

## Key Takeaways

- A **Server Action** is an `async` function marked `"use server"` that runs on
  the server, with no API route to write.
- Hand an action to a form with `<form action={myAction}>`; read fields with
  `formData.get("name")`.
- After changing data, call `revalidatePath` so the page shows the new state:
  **mutate, then revalidate.**
- You can call actions from Client Components too (for example from `onClick`).
- Use `useFormStatus` for a pending state and `useActionState` to validate input
  and show returned errors.

---

[← Previous: Streaming and Suspense](04-streaming-and-suspense.md) | [Back to Course Index](../README.md) | [Next: Styling →](../04-building-features/01-styling.md)
