# Server Actions in Depth

Server Actions let you mutate data from a component without hand-writing an API route — but that convenience hides a sharp edge: an action is a **public HTTP endpoint**, even though it looks like a function call. Developers who treat it as "just a function" ship authorization holes and unvalidated inputs straight to production. This lesson covers how actions really work over the wire, the security you cannot skip, and the client patterns (`useActionState`, `useOptimistic`) that make them feel great.

## What You'll Learn

- How Server Actions work over the wire: a POST to the same route with an encrypted action ID.
- The security model — why every action is a public endpoint you must authenticate, authorize, and validate.
- Input validation with Zod and returning typed results versus throwing.
- `useActionState` and `useOptimistic` for progressive enhancement and instant UX.
- Revalidating caches after a mutation.
- The risk of closing over server data in inline actions.
- When a Route Handler is the better tool.

## How Server Actions Work Over the Wire

A Server Action is a function marked `"use server"` (either a whole file or an inline directive). When you reference one from a Client Component — say as a form's `action` or an event handler — Next.js doesn't ship the function body to the browser. It ships an **encrypted reference (an action ID)**.

```text
1. Client invokes the action (form submit or call).
2. The browser sends a POST to the CURRENT route's URL, carrying:
   - the encrypted action ID (which server function to run)
   - the serialized arguments
3. The server decrypts the ID, runs the corresponding function with those args.
4. The function returns a (serializable) result and/or triggers revalidation.
5. The response streams back: the return value PLUS any updated RSC payload
   from revalidated segments, so the UI updates without a manual refetch.
```

Two consequences that shape everything:

- **It's an HTTP POST endpoint.** Anyone who can find the action ID can POST to it with arbitrary arguments. The "function call" ergonomics are a client-side illusion over a network request.
- **Arguments and return values must be serializable** — same boundary rules as RSC props. You can't return a class instance with methods.

```ts
// features/posts/server/actions.ts
"use server"; // every export in this file is a Server Action

import { db } from "@/lib/db";

export async function createPost(formData: FormData) {
  // This runs on the server. But it is reachable by anyone. Read on.
}
```

## Security: Treat Every Action as a Public Endpoint

This is the section to internalize. Because an action is a public POST endpoint, the call site giving it "safe-looking" data means nothing — an attacker calls the endpoint directly. Three things are mandatory in **every** mutating action.

### 1. Authenticate — who is this?

Re-establish identity from the request inside the action. Never trust an `userId` passed as an argument.

```ts
"use server";
import { getCurrentUser } from "@/features/auth/server";

export async function deletePost(postId: string) {
  const user = await getCurrentUser(); // reads session from cookies, server-side
  if (!user) throw new Error("Unauthorized");
  // ...
}
```

### 2. Authorize — are they allowed to do *this*?

Authentication isn't authorization. Confirm this user may act on this specific resource.

```ts
  const post = await db.post.findUnique({ where: { id: postId } });
  if (!post) throw new Error("Not found");
  if (post.authorId !== user.id && user.role !== "admin") {
    throw new Error("Forbidden"); // a logged-in user is NOT entitled to every post
  }
```

A logged-in user deleting *someone else's* post by passing a different `postId` is the canonical Server Action vulnerability. The ownership check is what stops it.

### 3. Validate — is the input well-formed?

Parse and validate every argument with a schema. Arguments arrive over the wire; the TypeScript types are erased and offer zero runtime protection.

```ts
"use server";
import { z } from "zod";

const CreatePostSchema = z.object({
  title: z.string().min(1).max(200),
  body: z.string().min(1).max(50_000),
  published: z.boolean(),
});

export async function createPost(_prev: unknown, formData: FormData) {
  const user = await getCurrentUser();
  if (!user) return { ok: false, error: "Unauthorized" } as const;

  const parsed = CreatePostSchema.safeParse({
    title: formData.get("title"),
    body: formData.get("body"),
    published: formData.get("published") === "on",
  });
  if (!parsed.success) {
    return { ok: false, error: "Invalid input", issues: parsed.error.flatten() } as const;
  }

  const post = await db.post.create({ data: { ...parsed.data, authorId: user.id } });
  revalidatePath("/posts");
  return { ok: true, id: post.id } as const;
}
```

The mental checklist for every action: **authenticate → authorize → validate → act → revalidate.** Skipping any of the first three is a security bug.

## Returning Typed Results vs Throwing

You have two ways to signal outcomes, and they serve different purposes.

- **Return a typed result** (`{ ok: true, ... } | { ok: false, error }`) for *expected* outcomes the UI must handle: validation errors, "username taken," "insufficient funds." These are part of your data flow, fully typed, and easy to render inline.
- **Throw** for *exceptional* failures: a database is down, an invariant is violated, the user is unauthorized in a way that should never happen through the UI. Thrown errors are caught by the nearest `error.tsx` boundary.

### Trade-offs: results vs exceptions

Returning typed results keeps the happy and unhappy paths visible in the type system and lets the form show field-level messages without an error boundary tearing down the page. Throwing is cleaner for genuinely unexpected states and centralizes handling, but a thrown error in an action triggers the error boundary — usually too heavy for "the title is too short." **Rule of thumb: expected, recoverable, field-level → return a result; truly exceptional → throw.**

## Client Patterns: `useActionState` and `useOptimistic`

### `useActionState` — pending state and results

`useActionState` wires an action to a form, exposes the latest returned state, and gives you a `pending` flag — all with progressive enhancement (the form works before JS hydrates).

```tsx
// features/posts/components/create-post-form.tsx
"use client";
import { useActionState } from "react";
import { createPost } from "../server/actions";

export function CreatePostForm() {
  const [state, action, pending] = useActionState(createPost, null);

  return (
    <form action={action}>
      <input name="title" required />
      <textarea name="body" required />
      <label><input type="checkbox" name="published" /> Publish</label>

      {state && !state.ok && <p role="alert">{state.error}</p>}
      <button disabled={pending}>{pending ? "Saving…" : "Create"}</button>
    </form>
  );
}
```

The `_prev` first argument in the action signature is the previous state that `useActionState` passes in — that's why the action above takes `(_prev, formData)`.

### `useOptimistic` — instant feedback

For mutations where you can predict the result, render the optimistic state immediately and reconcile when the server responds.

```tsx
"use client";
import { useOptimistic } from "react";
import { toggleLike } from "../server/actions";

export function LikeButton({ postId, likes }: { postId: string; likes: number }) {
  const [optimisticLikes, addOptimistic] = useOptimistic(likes, (n) => n + 1);

  return (
    <form action={async () => {
      addOptimistic(undefined);   // bump the count instantly
      await toggleLike(postId);   // server confirms; revalidation corrects if wrong
    }}>
      <button>♥ {optimisticLikes}</button>
    </form>
  );
}
```

The optimistic value shows immediately; when the action's revalidation streams back the real data, React reconciles. If the server rejected the action, the count snaps back to truth.

## Revalidation After Mutation

After a successful mutation you almost always need to invalidate cached data so the UI reflects the change.

```ts
import { revalidatePath, revalidateTag } from "next/cache";

revalidatePath("/posts");        // re-render anything rendered for this path
revalidateTag("posts");          // invalidate all fetches tagged "posts"
```

`revalidateTag` is the more scalable tool: tag your fetches by entity, and a single mutation can invalidate every page that depends on that entity without you enumerating paths. We go deep on tags in the caching module. Sometimes you instead want `redirect()` after a mutation (e.g., to the new resource), which navigates and naturally fetches fresh data.

## Closures Over Server Data — A Quiet Risk

Inline Server Actions can close over variables from the enclosing server scope. This is convenient but carries a real hazard: **closed-over values are encrypted and embedded in the payload sent to the client**, then sent back when the action runs.

```tsx
export default async function Page() {
  const secret = await getApiSecret(); // server-only secret

  async function leak() {
    "use server";
    useSecret(secret); // ⚠️ `secret` is bound into the action's encrypted closure
  }
  // ...
}
```

Next.js encrypts these bound values so they aren't readable by the client, but you're still shipping them over the network round-trip, and the encryption key is per-deployment. The safer habit: **don't close over secrets or large data in inline actions.** Re-fetch what you need inside the action from a trusted source (session, IDs passed and re-validated). Treat closed-over data as data that leaves your server and comes back.

## When to Prefer a Route Handler Instead

Server Actions aren't the answer to every server interaction. Reach for a **Route Handler** (`route.ts`) when:

- You need a **public or third-party-facing API** (webhooks, mobile clients, partner integrations) — actions are coupled to your app's form/RSC flow.
- You need **fine HTTP control**: custom status codes, headers, streaming responses, content negotiation, CORS.
- The operation is a **read/query**, not a mutation. Actions are designed for mutations; for data fetching, fetch in a Server Component or expose a handler.
- You want a **stable, versioned contract** that other systems depend on. Action IDs are an internal detail, not an API.

### Trade-offs

Server Actions win on developer ergonomics for mutations driven by your own UI: colocated, typed, progressively enhanced, with built-in revalidation. Route Handlers win when you need an explicit, addressable, protocol-level HTTP endpoint with a contract. Use actions for "the form on this page saves a thing"; use handlers for "other systems talk to us."

## Pitfalls

- **Trusting arguments.** TypeScript types vanish at runtime; an attacker POSTs whatever they like. Validate every input with a schema.
- **Authenticating but not authorizing.** Checking that *a* user is logged in without checking they own *this* resource is the classic action vulnerability.
- **Throwing for expected validation errors.** It triggers `error.tsx` and tears down the page; return a typed result for recoverable, field-level problems instead.
- **Forgetting to revalidate.** The mutation succeeds but the UI shows stale data because no cache was invalidated. Pair every mutation with `revalidatePath`/`revalidateTag` or a `redirect`.
- **Closing over secrets/large data in inline actions.** They're embedded (encrypted) in the payload and round-tripped. Re-fetch inside the action instead.
- **Using an action where you need a real API.** Webhooks and external consumers need a Route Handler with a stable contract, not an internal action ID.

## Exercises

1. **Harden an action.** Take a `deleteComment(commentId)` action that only checks login and add proper authorization plus Zod validation. Describe the exact attack the authorization check prevents.
2. **Result vs throw.** Refactor an action that throws on validation errors to return a typed discriminated-union result, and update the form to render field-level messages via `useActionState`.
3. **Optimistic toggle.** Implement a follow/unfollow button with `useOptimistic` that updates instantly and self-corrects if the server rejects the change.
4. **Revalidation by tag.** Design a tagging scheme for a blog where editing a post invalidates the post page, the author's page, and the homepage feed — with a single `revalidateTag` call per concern.
5. **Action or handler?** For each of these, decide action vs Route Handler and justify: a contact form, a Stripe webhook, a "load more comments" fetch, a partner-facing data export API.

## Key Takeaways

- A Server Action is an encrypted-ID POST endpoint, not a local function call — convenient ergonomics over a real network request with serialization rules.
- Every mutating action must authenticate, authorize, and validate; skipping any of these ships a security hole. The classic bug is authenticating without an ownership check.
- Return typed results for expected/recoverable outcomes and throw only for exceptional failures.
- Use `useActionState` for pending/result state with progressive enhancement and `useOptimistic` for instant feedback; revalidate (path/tag) after every mutation.
- Avoid closing over secrets/large data in inline actions, and reach for a Route Handler when you need a public, protocol-level, contract-stable HTTP endpoint.

---

[← Previous: Streaming and Suspense Orchestration](03-streaming-orchestration.md) | [Back to Course Index](../README.md) | [Next: The Next.js Caching Layers →](../02-caching-and-data/01-caching-layers.md)
