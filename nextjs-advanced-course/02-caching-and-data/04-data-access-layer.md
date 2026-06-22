# Designing a Data Access Layer (DAL)

In a small app, calling your ORM directly from a Server Component is fine. In a real app, scattering `db.post.findMany(...)` across dozens of components is how you end up with inconsistent caching, authorization checks that exist in some places and not others, and a refactor that touches forty files when the schema changes. A Data Access Layer is the discipline of routing *all* data access through a single, typed, secure-by-default module.

The App Router raises the stakes. Because Server Components and Server Actions run trusted code with database access, the boundary between "data the user is allowed to see" and "data the query can return" is now a code-organization problem. The DAL is where you draw and enforce that boundary — not in middleware, not in the component, but in the function that touches the data.

## What You'll Learn

- Why centralizing data access improves security, caching, and testability
- The DAL / repository pattern applied to a Next.js app
- Using `import 'server-only'` to guarantee modules never reach the client
- Putting authorization *inside* the DAL for "secure by default" functions
- Writing typed query functions and mapping DB rows to domain types
- Where Server Actions, Server Components, and Route Handlers call into the DAL
- A worked `lib/dal/posts.ts` you can adapt

## Why Centralize Data Access

Three forces push you toward a DAL as an app grows:

- **Security.** Authorization is easy to forget in one of forty call sites. A DAL gives you *one* place where every read and write checks who's asking. Middleware can't do this reliably — it runs before you know which records are involved.
- **Caching consistency.** Tagging, `revalidate` windows, and `cache()` memoization should be defined once per query, not re-invented at each call site. Centralizing means a query is cached (or not) the same way everywhere.
- **Testability and change resilience.** When the schema or data source changes, you edit the DAL, not every component. And you can unit-test query functions in isolation, mocking the DB at one seam.

```text
WITHOUT a DAL                         WITH a DAL
Component ─┐                          Component ─┐
Action ────┼──▶ db.post.findMany      Action ────┼──▶ lib/dal/posts ──▶ db
Route ─────┘   (auth? cache? ad hoc)  Route ─────┘   (auth + cache + types
                                                      enforced once)
```

## Keeping the DAL Off the Client: `server-only`

DAL modules hold database clients, secrets, and authorization logic. None of that should ever be bundled into client code. The `server-only` package turns an accidental client import into a **build-time error** instead of a silent security leak.

```ts
// lib/dal/posts.ts
import 'server-only'; // first line: importing this from a Client Component fails the build

import { db } from '@/lib/db';
```

Pair it with linting that flags `db` imports outside `lib/dal`, and you've made "query the database from a component" impossible to do by accident.

## Secure by Default: Authorization Inside the DAL

The core idea: a DAL function should be safe to call from anywhere, because *it* verifies the caller's permission. You never have to trust that some upstream middleware ran. Authorization lives next to the data it protects.

```ts
// lib/dal/auth.ts
import 'server-only';
import { cache } from 'react';
import { cookies } from 'next/headers';
import { verifySession } from '@/lib/session';

// Memoized per request so every DAL call shares one session lookup.
export const getCurrentUser = cache(async () => {
  const token = (await cookies()).get('session')?.value;
  const session = token ? await verifySession(token) : null;
  return session?.user ?? null;
});

export async function requireUser() {
  const user = await getCurrentUser();
  if (!user) throw new UnauthorizedError(); // no session → no data, ever
  return user;
}
```

Now every DAL function starts by establishing *who* is asking and *whether* they may proceed:

```ts
// lib/dal/posts.ts (excerpt)
export async function getDraftPost(slug: string): Promise<Post | null> {
  const user = await requireUser();          // 1. authenticate
  const row = await db.post.findUnique({ where: { slug } });
  if (!row) return null;
  if (row.authorId !== user.id) {            // 2. authorize on the actual record
    throw new ForbiddenError();              //    can't read someone else's draft
  }
  return toPost(row);                        // 3. map to domain type
}
```

This record-level check is exactly what middleware *cannot* do — middleware doesn't know `row.authorId` until the query runs. The DAL is the only layer that has both the request context and the data.

## Typed Queries and Domain Mapping

DB rows are not your domain model. They carry columns you don't want to expose (password hashes, internal flags), use storage shapes (snake_case, raw timestamps) you don't want to leak, and change when the schema changes. Map rows to a stable domain type at the DAL boundary so the rest of the app depends on *your* types, not the database's.

```ts
// lib/dal/types.ts
export type Post = {
  id: string;
  slug: string;
  title: string;
  body: string;
  authorId: string;
  publishedAt: Date | null;
  // Note: no `internalScore`, no `deletedAt` — those stay in the DB.
};
```

```ts
// lib/dal/posts.ts (mapper)
import type { Post as PostRow } from '@prisma/client';

// One place that decides what a "Post" is to the rest of the app.
function toPost(row: PostRow): Post {
  return {
    id: row.id,
    slug: row.slug,
    title: row.title,
    body: row.body,
    authorId: row.authorId,
    publishedAt: row.publishedAt,
  };
}
```

If the DB column renames or a sensitive field is added, you fix `toPost` once and every consumer stays correct and safe.

## Worked Example: `lib/dal/posts.ts`

A complete, secure-by-default module with reads (cached + tagged) and writes (authorized + invalidating).

```ts
// lib/dal/posts.ts
import 'server-only';

import { cache } from 'react';
import { unstable_cache } from 'next/cache';
import { revalidateTag } from 'next/cache';
import type { Post as PostRow } from '@prisma/client';

import { db } from '@/lib/db';
import { requireUser, getCurrentUser } from './auth';
import { ForbiddenError } from '@/lib/errors';
import type { Post } from './types';

// --- mapping -------------------------------------------------------------
function toPost(row: PostRow): Post {
  return {
    id: row.id,
    slug: row.slug,
    title: row.title,
    body: row.body,
    authorId: row.authorId,
    publishedAt: row.publishedAt,
  };
}

// --- tag helpers (single source of truth for reads + writes) -------------
const tags = {
  all: 'posts',
  one: (slug: string) => `post:${slug}`,
  byAuthor: (id: string) => `author:${id}:posts`,
};

// --- reads (public, cached in the Data Cache, tagged) --------------------
export const getPublishedPosts = unstable_cache(
  async (): Promise<Post[]> => {
    const rows = await db.post.findMany({
      where: { publishedAt: { not: null } },
      orderBy: { publishedAt: 'desc' },
    });
    return rows.map(toPost);
  },
  ['published-posts'],
  { tags: [tags.all], revalidate: 3600 },
);

// Per-request memoized public read by slug.
export const getPublishedPost = cache(
  async (slug: string): Promise<Post | null> => {
    const row = await db.post.findFirst({
      where: { slug, publishedAt: { not: null } },
    });
    return row ? toPost(row) : null;
  },
);

// --- reads (private, authorized at the record level) ---------------------
export async function getMyDraft(slug: string): Promise<Post | null> {
  const user = await requireUser();
  const row = await db.post.findUnique({ where: { slug } });
  if (!row) return null;
  if (row.authorId !== user.id) throw new ForbiddenError();
  return toPost(row);
}

export async function listMyPosts(): Promise<Post[]> {
  const user = await requireUser();
  const rows = await db.post.findMany({ where: { authorId: user.id } });
  return rows.map(toPost);
}

// --- writes (authorized + invalidating) ----------------------------------
export async function createPost(input: {
  title: string;
  body: string;
  slug: string;
}): Promise<Post> {
  const user = await requireUser();
  const row = await db.post.create({
    data: { ...input, authorId: user.id },
  });
  // Invalidation lives WITH the write — callers can't forget it.
  revalidateTag(tags.all);
  revalidateTag(tags.byAuthor(user.id));
  return toPost(row);
}

export async function updatePost(
  slug: string,
  input: Partial<{ title: string; body: string }>,
): Promise<Post> {
  const user = await requireUser();
  const existing = await db.post.findUnique({ where: { slug } });
  if (!existing) throw new ForbiddenError();
  if (existing.authorId !== user.id) throw new ForbiddenError(); // own it to edit it

  const row = await db.post.update({ where: { slug }, data: input });
  revalidateTag(tags.one(slug));
  revalidateTag(tags.all);
  return toPost(row);
}
```

Every function authenticates, authorizes against the actual record where relevant, maps to the domain type, and (for writes) invalidates the right tags. There is no way to call into posts without those guarantees.

## Where Each Caller Plugs In

The DAL is consumed by all three server-side entry points — and only by server-side code.

```tsx
// Server Component: read directly.
// app/blog/page.tsx
import { getPublishedPosts } from '@/lib/dal/posts';

export default async function BlogPage() {
  const posts = await getPublishedPosts(); // cached + tagged for free
  return <PostList posts={posts} />;
}
```

```ts
// Server Action: write through the DAL (auth + invalidation handled there).
// app/actions/posts.ts
'use server';
import { createPost } from '@/lib/dal/posts';

export async function createPostAction(formData: FormData) {
  return createPost({
    title: String(formData.get('title')),
    body: String(formData.get('body')),
    slug: String(formData.get('slug')),
  });
}
```

```ts
// Route Handler: same DAL, e.g. for an external API client.
// app/api/posts/route.ts
import { getPublishedPosts } from '@/lib/dal/posts';

export async function GET() {
  return Response.json(await getPublishedPosts());
}
```

### Trade-offs: DAL vs direct access, and how thick to make it

| Approach | Pros | Cons | Use when |
|---|---|---|---|
| Direct ORM in components | Zero boilerplate | Scattered auth/caching; risky | Prototypes, tiny apps |
| Thin DAL (typed query fns) | Centralized, testable | Some boilerplate | Most apps |
| Repository + service layers | Strong boundaries, swappable DB | More indirection, slower to write | Large teams, multiple data sources |

Most production Next apps want the **thin DAL**: typed functions that own auth, caching, and mapping, without a full repository abstraction. Add service/repository layers only when you have multiple data sources or genuinely need to swap the persistence layer — otherwise the indirection costs more than it pays.

## Pitfalls

- **Authorization only in middleware.** Middleware can't see record ownership. Without a DAL check, any code path that skips middleware (a Server Action, a Route Handler) can leak data. Authorize where the data is.
- **Leaking raw DB rows.** Returning ORM objects exposes internal fields and couples the whole app to the schema. Always map to domain types at the boundary.
- **Missing `server-only`.** Without it, a careless import can pull DB code and secrets into the client bundle. Make it the first line of every DAL module.
- **Caching private data globally.** `unstable_cache` is shared across all users. Never cache user-scoped reads in it without a user-specific key/tag, or one user sees another's data.
- **Inconsistent invalidation.** If writes live in the DAL but some still happen elsewhere, tags drift. Funnel *all* mutations through the DAL so invalidation is guaranteed.
- **Over-abstracting early.** A repository/service stack on a three-table app is pure friction. Start thin; add layers when a real need appears.

## Exercises

1. **Extract a DAL.** Take a route that calls the ORM directly and move every query into `lib/dal/`. Add `server-only` and confirm the build fails if a Client Component imports it.
2. **Add record-level authorization** to a "get my X" function so it throws `ForbiddenError` when the record isn't owned by the current user. Write a test proving cross-user access is blocked.
3. **Design domain types** for one entity and write the row→domain mapper. List which DB fields you deliberately exclude and why.
4. **Audit caching scope.** Find any user-scoped data cached in `unstable_cache` without a per-user key and fix it. Explain the leak it would have caused.
5. **Route all writes through the DAL.** Find a mutation that happens outside the DAL, move it in, and centralize its `revalidateTag` calls. Verify lists and entities both invalidate.

## Key Takeaways

- A DAL centralizes data access so security, caching, and types are defined once and enforced everywhere.
- `import 'server-only'` turns accidental client imports of DB code into build errors — make it the first line.
- Authorization belongs *inside* the DAL, checked against the actual record — middleware cannot do record-level checks.
- Map DB rows to stable domain types at the boundary so the rest of the app never depends on the schema or sees sensitive fields.
- Server Components, Server Actions, and Route Handlers should all call into the same DAL; never query the database directly from a component.
- Prefer a thin, typed DAL; add repository/service layers only when multiple data sources or swappable persistence genuinely demand them.

---

[← Previous: Advanced Data Fetching Patterns](03-data-fetching-patterns.md) | [Back to Course Index](../README.md) | [Next: Project Architecture at Scale →](../03-architecture/01-project-architecture.md)
