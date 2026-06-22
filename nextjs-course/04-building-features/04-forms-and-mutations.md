# Forms and Data Mutations

Forms are how users *give* your app information — a login, a comment, a new blog post.
In this lesson we'll build a real form that saves data, shows a loading spinner while
it works, and displays helpful error messages. The best part: with Next.js, forms can
work **even before JavaScript loads**, like a paper form that still mails fine if the
fancy online version is down. 

## What You'll Learn

- How to build a form powered by a **Server Action**
- What **progressive enhancement** is (forms that work without JavaScript)
- How to show a "saving..." state with **`useFormStatus`**
- How to return validation errors and messages with **`useActionState`**
- Why you need **both** client-side and server-side validation
- How to make the UI feel instant with **`useOptimistic`** (briefly)

## A Form with a Server Action

A **Server Action** is a function that runs on the server, marked with the
`"use server"` directive. You can hand it straight to a form's `action` attribute, and
Next.js wires up the rest.

```tsx
// app/posts/new/page.tsx
export default function NewPostPage() {
  // this function runs on the SERVER when the form submits
  async function createPost(formData: FormData) {
    "use server"; // marks it as a Server Action

    // read the fields by their "name" attribute
    const title = formData.get("title");
    console.log("Saving post:", title);
    // ...save to a database here...
  }

  return (
    <form action={createPost}>
      <input name="title" placeholder="Post title" />
      <button type="submit">Create</button>
    </form>
  );
}
```

Notice there's no `fetch`, no `onSubmit`, no API endpoint. You pass the function
directly to `action={...}` and Next.js handles sending the form data to the server.

## Progressive Enhancement

**Progressive enhancement** means the form works at a basic level for *everyone*, then
gets nicer when extras (like JavaScript) are available. Because a Server Action form
uses the real HTML `<form>` `action`, the browser can submit it the old-fashioned way
— so it works **even if JavaScript hasn't loaded yet**. When JS *is* ready, Next.js
upgrades it to a smooth, no-refresh submission.

Analogy: it's like a staircase next to an escalator. If the escalator (JavaScript) is
off, you can still walk up the stairs and reach the same floor.

## Showing a Pending State with `useFormStatus`

While the server is working, you want to disable the button and show "Saving...". The
`useFormStatus` hook tells you whether the parent form is currently submitting.
Important: it must be used in a **child** component inside the form, and that component
must be a Client Component.

```tsx
// app/components/submit-button.tsx
"use client"; // hooks need a client component

import { useFormStatus } from "react-dom";

export default function SubmitButton() {
  // pending is true while the form is submitting
  const { pending } = useFormStatus();
  return (
    <button type="submit" disabled={pending}>
      {pending ? "Saving..." : "Create"}
    </button>
  );
}
```

Drop `<SubmitButton />` inside your `<form>` and it automatically reflects the form's
status. No props needed.

## Validation and Messages with `useActionState`

A form needs to tell the user when something's wrong ("Title is required"). The
`useActionState` hook (formerly called `useFormState`) lets your Server Action
*return* a value — like an error message — which React feeds back into your component.

> Heads up: this hook used to be named `useFormState` and lived in `react-dom`. In
> current React/Next it's **`useActionState`** from `react`.

The action now takes two arguments: the **previous state**, and the `formData`.

```tsx
// app/posts/actions.ts
"use server";

// the shape of what we return to the form
type State = { error?: string; success?: boolean };

export async function createPost(
  prevState: State, // the previous returned state
  formData: FormData,
): Promise<State> {
  const title = formData.get("title")?.toString().trim();

  // SERVER-SIDE validation
  if (!title) {
    return { error: "Title is required." };
  }
  if (title.length < 3) {
    return { error: "Title must be at least 3 characters." };
  }

  // ...save to the database...
  return { success: true };
}
```

Now wire it up in a Client Component with `useActionState`:

```tsx
// app/posts/new/page.tsx
"use client";

import { useActionState } from "react";
import { createPost } from "../actions";
import SubmitButton from "../../components/submit-button";

export default function NewPostPage() {
  // [current state, the action to pass to the form, isPending]
  const [state, formAction] = useActionState(createPost, {});

  return (
    <form action={formAction}>
      <input name="title" placeholder="Post title" />
      {/* show the error returned by the server */}
      {state.error && <p style={{ color: "red" }}>{state.error}</p>}
      {state.success && <p style={{ color: "green" }}>Saved! 🎉</p>}
      <SubmitButton />
    </form>
  );
}
```

When the action returns `{ error: "..." }`, that value lands in `state.error` and the
message appears — no page refresh.

## Client-Side vs Server-Side Validation

You should validate in **two** places, and they serve different purposes:

- **Client-side validation** runs in the browser. It's fast and friendly — it catches
  mistakes *before* sending anything (for example, the HTML `required` attribute, or
  checking a field as the user types). But it can be bypassed.
- **Server-side validation** runs in your Server Action. It's the **real** guard —
  never trust data from the browser, because a clever user (or a script) can skip the
  client checks.

```tsx
// quick client-side hints with plain HTML attributes
<input name="title" required minLength={3} />
```

Rule to remember: client-side validation is for *convenience*; server-side validation
is for *safety*. Always do the server side.

## Optimistic UI with `useOptimistic` (Brief)

Sometimes you want the screen to update *instantly*, before the server even replies —
that's an **optimistic update**. The `useOptimistic` hook shows a temporary, expected
result while the real action runs in the background.

```tsx
// app/components/like-button.tsx
"use client";

import { useOptimistic } from "react";

export default function Likes({ count }: { count: number }) {
  // show an instantly-updated count while the server catches up
  const [optimisticCount, addOptimistic] = useOptimistic(
    count,
    (current) => current + 1, // how to compute the temporary value
  );

  async function like() {
    addOptimistic(null); // bump the number on screen right away
    await saveLikeOnServer(); // real work happens after
  }

  return <button onClick={like}>👍 {optimisticCount}</button>;
}
```

The user sees the like count jump immediately; if the server fails, React rolls it
back. Use this for snappy interactions like likes, todos, and reactions.

## Common Mistakes

- **Forgetting `"use server"`.** Without it, your function isn't a Server Action and
  the form won't work.
- **Reading the wrong field name.** `formData.get("title")` must match the input's
  `name="title"` exactly.
- **Using `useFormStatus` in the same component as the form.** It must be in a *child*
  component *inside* the form.
- **Skipping server-side validation.** Client checks can be bypassed; always re-check
  on the server.
- **Mixing up the argument order in `useActionState` actions.** The action receives
  `(prevState, formData)` — previous state first.

## Exercises

1. Build a "create post" form whose Server Action logs the submitted title.
2. Add a `SubmitButton` child component using `useFormStatus` that shows "Saving..."
   while submitting.
3. Convert the action to use `useActionState` and return an error when the title is
   empty. Display it.
4. Add server-side validation requiring at least 3 characters, plus a client-side
   `minLength` for instant feedback.
5. (Stretch) Add a like button that uses `useOptimistic` to bump the count instantly.

## Key Takeaways

- A **Server Action** (`"use server"`) can be passed straight to a form's `action`.
- Server Action forms get **progressive enhancement** — they work even without JS.
- **`useFormStatus`** gives a `pending` flag for loading UI (use it in a child
  component).
- **`useActionState`** lets the action return validation errors/messages back to the
  form.
- Validate on the **client** for convenience and on the **server** for safety —
  always do both.
- **`useOptimistic`** updates the UI instantly while the real action finishes.

---

[← Previous: Route Handlers (API Routes)](03-route-handlers.md) | [Back to Course Index](../README.md) | [Next: Authentication Basics →](05-authentication-basics.md)
