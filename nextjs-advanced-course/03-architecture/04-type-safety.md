# End-to-End Type Safety

TypeScript gives you compile-time guarantees about code you control. It tells you nothing about data crossing a boundary — a database row, a form submission, an API response, an environment variable. Those arrive as `any`-shaped reality, and the moment you cast them to a type, you've lied to the compiler. End-to-end type safety means the types you trust are *earned* at each boundary by runtime validation, then flow unbroken from the database through the server to the client. This lesson is about building that unbroken chain and validating where it enters.

## What You'll Learn

- How types flow DB → DAL → Server Component → props → client, and where the chain breaks
- Why "parse, don't validate" (and don't trust) is the rule at every boundary
- Runtime validation with Zod: parsing inputs and inferring static types from schemas
- Typed Server Actions and robust form parsing
- Typed, validated environment variables
- Where tRPC earns its place — and where RSC + Server Actions make it unnecessary
- Eliminating `any` at boundaries without scattering casts

## The Type Chain and Where It Breaks

In a well-typed App Router app, types flow in one direction with no manual casts:

```text
DB schema  →  DAL query (typed return)  →  Server Component (awaits typed data)
           →  component props (typed DTO)  →  client component (typed props)
```

If you use Prisma or Drizzle, the DB layer generates types from the schema, so a query's return type is known. The DAL re-exposes those (or narrowed DTOs). Server Components await typed data; props carry typed DTOs; client leaves receive typed props. As long as nobody casts, the compiler tracks a field rename from the schema all the way to the JSX that renders it.

The chain breaks at exactly two kinds of boundary: **untyped external inputs** (request bodies, form data, third-party API responses, env vars) and **`any` casts** used to silence the compiler. Both turn a guarantee into a hope. The fix is the same in both cases: validate at the boundary and let inference carry a real type forward.

## Parse, Don't Trust

The principle: at every boundary where data enters from outside your type system, **parse it through a schema that throws (or returns an error) on mismatch**, and use the *parsed result*. Never cast (`as Foo`) raw input — a cast asserts a shape the compiler can't check, so a malformed payload sails through to corrupt your logic.

Zod is the standard tool. A schema is simultaneously a runtime validator and a source of static types, so you write the shape once and `z.infer` the type — they can never drift:

```ts
// features/users/schema.ts
import { z } from "zod";

export const createUserSchema = z.object({
  email: z.string().email(),
  name: z.string().min(1).max(100),
  role: z.enum(["member", "admin"]),
  age: z.coerce.number().int().min(0).optional(), // coerce string→number from forms
});

// One source of truth: the static type is derived from the runtime schema.
export type CreateUserInput = z.infer<typeof createUserSchema>;
```

At the boundary, parse:

```ts
// Boundary: an untyped body becomes a trusted, typed value — or it throws.
const result = createUserSchema.safeParse(rawBody);
if (!result.success) {
  // result.error has structured, per-field issues
  return { ok: false, errors: result.error.flatten().fieldErrors };
}
const input = result.data; // typed as CreateUserInput, guaranteed valid
```

The same applies to third-party API responses: don't `await res.json() as ApiShape`. Parse the JSON through a Zod schema so a silently changed upstream field surfaces as a validation error you log, not a runtime crash three layers deep.

## Typed Server Actions and Form Parsing

Server Actions receive `FormData` — an untyped key/value bag where every value is a string or `File`. This is a classic boundary, and it's where most type-safety bugs hide. Validate the `FormData` through your schema before touching it:

```ts
// features/users/actions.ts
"use server";

import { revalidatePath } from "next/cache";
import { createUserSchema } from "./schema";
import { db } from "@/lib/db";

// A typed result, not a thrown error, for expected validation failures.
type ActionResult =
  | { ok: true; userId: string }
  | { ok: false; errors: Record<string, string[]> };

export async function createUser(
  _prev: ActionResult | null,
  formData: FormData,
): Promise<ActionResult> {
  // Parse the untyped FormData boundary.
  const parsed = createUserSchema.safeParse({
    email: formData.get("email"),
    name: formData.get("name"),
    role: formData.get("role"),
    age: formData.get("age"),
  });

  if (!parsed.success) {
    return { ok: false, errors: parsed.error.flatten().fieldErrors };
  }

  // parsed.data is fully typed and validated from here on.
  const user = await db.user.create({ data: parsed.data });
  revalidatePath("/users");
  return { ok: true, userId: user.id };
}
```

On the client, wire it with `useActionState` so the typed `ActionResult` flows back into the UI for field-level error display:

```tsx
// features/users/CreateUserForm.tsx
"use client";
import { useActionState } from "react";
import { createUser } from "./actions";

export function CreateUserForm() {
  const [state, action, pending] = useActionState(createUser, null);
  return (
    <form action={action}>
      <input name="email" />
      {state?.ok === false && state.errors.email && (
        <p role="alert">{state.errors.email[0]}</p>
      )}
      <button disabled={pending}>Create</button>
    </form>
  );
}
```

The type `ActionResult` is shared between the action and the form, so the discriminated union (`ok: true | false`) gives the client compile-time knowledge of which fields exist in each branch. This is end-to-end type safety made concrete: one schema validates the input, one result type carries the outcome, and both ends agree at compile time.

### Trade-offs

You can wrap this boilerplate in a small helper (or use a library like `next-safe-action` / `zsa`) that takes a Zod schema and a handler and produces a typed, validated action with consistent error shapes. Worth it once you have more than a handful of actions; for one or two, the explicit version above is clearer and dependency-free.

## Typed Environment Variables

`process.env.WHATEVER` is typed `string | undefined` and validated by nobody. A missing `DATABASE_URL` becomes a confusing runtime error far from the cause. Validate env once at startup with a schema, export a typed object, and import *that* everywhere — never touch `process.env` directly in app code:

```ts
// lib/env.ts
import { z } from "zod";

const envSchema = z.object({
  DATABASE_URL: z.string().url(),
  STRIPE_SECRET_KEY: z.string().min(1),
  NODE_ENV: z.enum(["development", "test", "production"]),
  // NEXT_PUBLIC_* are inlined at build time; validate them too
  NEXT_PUBLIC_APP_URL: z.string().url(),
});

// Throws at boot if anything is missing/malformed — fail fast, with a clear message.
export const env = envSchema.parse(process.env);
```

Now `env.STRIPE_SECRET_KEY` is a guaranteed non-empty string, autocompleted, and a typo is a compile error. For stricter server/client separation, `@t3-oss/env-nextjs` enforces that server-only vars can't be imported into client code.

## tRPC: When It Helps, When It's Redundant

tRPC gives you end-to-end-typed RPC: define procedures on the server, call them from the client with full inference and no codegen, no manual API types. Before RSC, this was a huge win — it eliminated the hand-written, drift-prone types between your API routes and your React fetch calls.

In an App Router app, **much of tRPC's value is already provided by the framework**:

- **Server Components** call your DAL functions *directly* with full type inference — there's no client/server boundary to type because the code runs on the server. No RPC layer needed.
- **Server Actions** are typed function references callable from the client; arguments and return values are inferred end-to-end, validated with Zod at the boundary. That's typed RPC, built in.

So for an app that's mostly server-rendered with mutations via Server Actions, tRPC is largely redundant — you'd be adding a layer to re-solve a problem the framework already solves.

tRPC still earns its place when: you have a **separate client** (a mobile app, a third-party consumer, a heavily client-fetched SPA-like surface) that needs a typed API contract; you want a unified, organized procedure layer with middleware (auth, rate limiting, logging) across many endpoints; or you need React Query integration with typed query keys across a large client-fetching surface.

### Trade-offs

Don't bolt tRPC onto a server-first Next app reflexively — it adds a dependency, a router setup, and a conceptual layer that competes with Server Actions for the same job. Reach for it when a non-RSC consumer needs the contract, or when your client-fetching surface is large enough that a structured, typed procedure layer pays for itself. For everything else, DAL + Server Actions + Zod is the lighter, more idiomatic path.

## Avoiding `any` at Boundaries

`any` is the chain's weakest link — it disables checking and spreads silently to everything it touches. At boundaries:

- Replace `as SomeType` casts on external data with a Zod `parse`. The parsed value carries a real, checked type.
- Type intentionally-unknown inputs as `unknown`, not `any`, forcing a parse/narrow before use.
- Enable `@typescript-eslint/no-explicit-any` and `no-unsafe-*` rules so casts and `any` leaks fail CI.
- For third-party libs with bad types, write a thin typed wrapper in `lib/` that parses/narrows once, so the `any` is contained to one audited file.

## Pitfalls

- **Casting request/form/API data with `as`.** A lie to the compiler; malformed data flows straight through. Parse instead.
- **Duplicating a Zod schema and a TS interface.** They drift. Derive the type with `z.infer` from the single schema.
- **Throwing on expected validation failures in Server Actions.** Return a typed result so the UI can show field errors; reserve throws for the unexpected.
- **Reading `process.env` directly across the app.** Untyped and unvalidated. Centralize in a validated `env` module.
- **Trusting upstream API responses.** Schemas change without notice. Parse responses at the fetch boundary.
- **Adding tRPC to a server-first app out of habit.** Redundant with RSC + Server Actions; adds a layer that solves an already-solved problem.

## Exercises

1. Take a Server Action that currently does `const data = Object.fromEntries(formData) as CreateInput`. Replace it with a Zod `safeParse`, a typed `ActionResult` union, and field-level error display via `useActionState`.
2. Define one Zod schema for a `Product` and derive the static type with `z.infer`. Show that renaming a field in the schema produces a compile error at the consuming component — proving the chain holds.
3. Build a validated `lib/env.ts` for an app with a DB URL, a public app URL, and a secret key. Demonstrate the boot-time failure when a var is missing, and contrast with the unvalidated `process.env` failure mode.
4. Wrap a third-party API fetch so its response is parsed through a Zod schema in one `lib/` wrapper. Show how an upstream field rename surfaces as a logged validation error instead of a deep runtime crash.
5. Write the decision memo for whether your app needs tRPC. List the specific conditions (separate client, large client-fetch surface, cross-cutting middleware) that would justify it, and conclude for a given app description.

## Key Takeaways

- Types flow DB → DAL → Server Component → props → client; the chain breaks only at untyped external inputs and `any` casts.
- Parse, don't trust: validate every boundary with Zod and use the parsed result, never an `as` cast.
- Derive static types from schemas with `z.infer` so runtime validation and compile-time types can't drift.
- Type Server Actions with a shared discriminated-union result; parse `FormData` before use; return errors rather than throwing for expected failures.
- Validate environment variables once at boot and import a typed `env` object everywhere instead of touching `process.env`.
- RSC + Server Actions + Zod already deliver typed, validated end-to-end calls; add tRPC only when a non-RSC consumer or a large client-fetch surface needs a structured contract.

---

[← Previous: State Management at Scale](03-state-management.md) | [Back to Course Index](../README.md) | [Next: Error Handling and Resilience →](05-error-handling-resilience.md)
