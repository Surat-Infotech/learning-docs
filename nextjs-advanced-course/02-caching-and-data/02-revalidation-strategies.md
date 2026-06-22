# Revalidation and Invalidation Strategies

Phil Karlton's line — "there are only two hard things in Computer Science: cache invalidation and naming things" — is not a joke when you run a Next.js app at scale. The caching layers from the previous lesson make you fast, but the moment data changes, you owe your users a correct view of the world. Get invalidation wrong and you serve stale prices, deleted posts, or another user's data.

This lesson is about *correctness under change*: the mechanisms Next.js gives you to revalidate the Data Cache and Full Route Cache, how to design an invalidation scheme that scales beyond a handful of routes, and the patterns that keep it correct as your data model grows.

## What You'll Learn

- Time-based revalidation: per-fetch `next: { revalidate }` and segment-level `export const revalidate`
- On-demand revalidation by path (`revalidatePath`) and by tag (`revalidateTag`)
- How to design a tagging scheme that mirrors your data model
- Caching non-`fetch` data with `unstable_cache` and React's `cache`
- Stale-while-revalidate behavior and what users actually see during a refresh
- Patterns to keep invalidation correct: tag granularity, mutation-driven invalidation, and avoiding over/under-invalidation

## Time-Based Revalidation

The simplest strategy: let cached data expire after N seconds. The first request after expiry triggers a background refresh while still serving the stale value (stale-while-revalidate).

```tsx
// Per-fetch: this specific request revalidates at most once per 60s.
const res = await fetch('https://api.example.com/posts', {
  next: { revalidate: 60 },
});
```

You can also set it at the **segment level**, which applies to the whole route:

```tsx
// app/blog/page.tsx
// Every cached data access in this segment revalidates at 3600s
// unless a more specific per-fetch value overrides it.
export const revalidate = 3600;

export default async function BlogPage() {
  const posts = await getPosts();
  return <PostList posts={posts} />;
}
```

When per-fetch and segment values conflict, the **lower** value wins for that fetch — Next picks the most frequent revalidation requested. Use segment `revalidate` as a route-wide ceiling and per-fetch values to make hot data fresher.

### Stale-While-Revalidate Behavior

Time-based revalidation does **not** block the request that triggers the refresh. Here's the timeline:

```text
t=0     fetch cached, revalidate: 60
t=0–60  every request served from cache (fresh)
t=61    request A arrives:
          • served the STALE cached value immediately (fast)
          • a background refresh kicks off
t=62    refresh completes, Data Cache updated
t=63    request B arrives → served the NEW value
```

So the user who triggers revalidation still sees old data; the *next* user sees new data. This is usually what you want — no one waits on the origin — but it means time-based caching is eventually consistent, never immediately consistent. For "must be correct now" data, use on-demand invalidation instead.

## On-Demand Revalidation

When you *know* data changed — because you just wrote it — don't wait for a timer. Invalidate immediately from a Server Action or Route Handler.

### `revalidatePath`

Invalidates the cache for a specific route path:

```ts
// app/actions/posts.ts
'use server';
import { revalidatePath } from 'next/cache';

export async function createPost(formData: FormData) {
  await db.post.create({ data: parse(formData) });

  // Bust the Data Cache + Full Route Cache for these routes,
  // and refresh the client Router Cache for the current route.
  revalidatePath('/blog');
  revalidatePath('/blog/[slug]', 'page'); // all dynamic post pages
}
```

`revalidatePath` is blunt: it invalidates everything that route touched. That's fine for a few routes, but it couples invalidation to URL structure — refactor a route and your invalidation calls silently miss.

### `revalidateTag`

Tags decouple invalidation from URLs. You attach tags to cached data, then invalidate by tag from anywhere:

```ts
// app/lib/posts.ts
export async function getPosts() {
  const res = await fetch('https://api.example.com/posts', {
    next: { tags: ['posts'] },          // tag this data
  });
  return res.json();
}

export async function getPost(slug: string) {
  const res = await fetch(`https://api.example.com/posts/${slug}`, {
    next: { tags: ['posts', `post:${slug}`] }, // collection + entity tags
  });
  return res.json();
}
```

```ts
// app/actions/posts.ts
'use server';
import { revalidateTag } from 'next/cache';

export async function updatePost(slug: string, data: PostInput) {
  await db.post.update({ where: { slug }, data });

  // Invalidate just this post everywhere it's cached...
  revalidateTag(`post:${slug}`);
  // ...and any list that includes it.
  revalidateTag('posts');
}
```

## Designing a Tagging Scheme

A tagging scheme is a contract between your *reads* (which tags they attach) and your *writes* (which tags they invalidate). Design it around your data model, not your routes.

A workable convention for a posts model:

```text
Collection tags    posts                 — any list of posts
Entity tags        post:{slug}           — one specific post
Relation tags      author:{id}:posts     — posts by an author
                   tag:{name}:posts      — posts in a category
User-scoped tags   user:{id}:drafts      — a user's private data
```

Apply them on reads:

```ts
// app/lib/posts.ts
export async function getPostsByAuthor(authorId: string) {
  const res = await fetch(
    `https://api.example.com/authors/${authorId}/posts`,
    { next: { tags: ['posts', `author:${authorId}:posts`] } },
  );
  return res.json();
}
```

Invalidate the matching set on writes:

```ts
'use server';
import { revalidateTag } from 'next/cache';

export async function publishPost(post: Post) {
  await db.post.update({ where: { id: post.id }, data: { published: true } });

  revalidateTag(`post:${post.slug}`);           // the entity
  revalidateTag('posts');                        // every list
  revalidateTag(`author:${post.authorId}:posts`);// author's list
  for (const t of post.categories) {
    revalidateTag(`tag:${t}:posts`);             // each category list
  }
}
```

### Trade-offs: path vs tag, and tag granularity

| Approach | Pros | Cons | Use when |
|---|---|---|---|
| `revalidatePath` | Simple, no read-side setup | Coupled to URLs; over-invalidates | Few routes, or "refresh this page now" |
| `revalidateTag` | Decoupled from routes; precise | Requires disciplined tagging | Shared data across many routes |
| Coarse tags (`posts`) | Easy, never under-invalidate | Refreshes too much, more origin load | Low write rate |
| Fine tags (`post:{slug}`) | Minimal refresh, cheap | Easy to forget a tag → stale | High write rate, hot lists |

The pragmatic answer is **both**: tag each entity finely *and* with its collection. Invalidate the entity tag for precision and the collection tag to catch lists. Under-invalidation (stale data) is worse than over-invalidation (extra origin calls), so when unsure, invalidate the broader tag too.

## Caching Non-`fetch` Data

`fetch` tagging only helps when you fetch over HTTP. Direct DB/ORM calls need `unstable_cache` to live in the Data Cache, and React's `cache` for per-request dedupe.

```ts
// app/lib/posts.ts
import { unstable_cache } from 'next/cache';
import { cache } from 'react';
import { db } from './db';

// Persistent Data Cache for a DB query, with tags + revalidate.
export const getCachedPost = unstable_cache(
  async (slug: string) => db.post.findUnique({ where: { slug } }),
  ['post-by-slug'],                  // key parts (plus the args)
  { tags: ['posts'], revalidate: 3600 },
);

// Per-request memoization for non-fetch data (no cross-request persistence).
export const getPostMemoized = cache(async (slug: string) =>
  db.post.findUnique({ where: { slug } }),
);
```

`unstable_cache` tags behave exactly like `fetch` tags — `revalidateTag('posts')` invalidates both. Note its first argument's *closure* values are not part of the cache key; pass anything that affects the result as a function argument or include it in the key array.

## Cache Invalidation as a Hard Problem

The failure modes are predictable. Build habits that prevent them.

### Pattern 1: Invalidate at the mutation, not on a guess

Every write path should be the single place that knows what it invalidates. Colocate the `revalidateTag` calls with the mutation (ideally inside your data access layer, see the DAL lesson) so no caller can forget.

```ts
// app/lib/dal/posts.ts
import { revalidateTag } from 'next/cache';

export async function deletePost(slug: string) {
  const post = await db.post.delete({ where: { slug } });
  // Invalidation lives WITH the write — callers can't get it wrong.
  revalidateTag(`post:${slug}`);
  revalidateTag('posts');
  revalidateTag(`author:${post.authorId}:posts`);
  return post;
}
```

### Pattern 2: Make tags derivable, not memorized

If your tags follow a strict convention (`post:{slug}`), write small helpers so reads and writes can't drift:

```ts
// app/lib/cache-tags.ts
export const postTags = {
  all: () => 'posts' as const,
  one: (slug: string) => `post:${slug}` as const,
  byAuthor: (id: string) => `author:${id}:posts` as const,
};
```

Now both the fetch (`next: { tags: [postTags.one(slug)] }`) and the invalidation (`revalidateTag(postTags.one(slug))`) reference the same source of truth.

### Pattern 3: Prefer narrow time windows as a safety net

Even with on-demand invalidation, set a modest `revalidate` (e.g. an hour) on cached data. If a mutation path ever forgets a tag, the data self-corrects within the window instead of staying stale forever. On-demand for freshness; time-based as the backstop.

## Pitfalls

- **Tagging reads but forgetting some writes.** The classic stale bug. Every mutation that affects tagged data must invalidate it. Centralize writes so this is enforced, not hoped for.
- **Over-relying on `revalidatePath` with dynamic segments.** `revalidatePath('/blog/[slug]', 'page')` is needed to hit all dynamic instances; `revalidatePath('/blog/my-post')` only hits that one URL. Mixing them up leaves pages stale.
- **Expecting time-based revalidation to be immediate.** The triggering request sees stale data; only subsequent requests see fresh. Use on-demand for "correct now."
- **Closure values in `unstable_cache`.** Variables captured from the closure are not part of the key. Pass them as arguments or stale results leak across keys.
- **Invalidating from a non-action context.** `revalidateTag`/`revalidatePath` must run in a Server Action or Route Handler — not during render. Calling them in a Server Component render throws.
- **Forgetting the Router Cache.** On-demand revalidation from a Server Action refreshes the current route's client cache; a plain Route Handler does not push to the client. For client-visible freshness after navigation, ensure the action runs or call `router.refresh()`.

## Exercises

1. **Design a tagging scheme** for an e-commerce model (products, categories, carts, orders). List the tags each read attaches and each mutation invalidates. Identify where coarse vs fine tags belong.
2. **Build a `cache-tags.ts` helper** for that model so reads and writes share one source of truth. Refactor two existing fetches and one mutation to use it.
3. **Prove stale-while-revalidate.** Instrument a `revalidate: 30` fetch to log whether each request was served fresh or stale and whether it triggered a background refresh. Confirm the triggering request gets the stale value.
4. **Find the under-invalidation bug.** Given a mutation that updates a post but only calls `revalidateTag('post:{slug}')`, identify which views go stale and fix it.
5. **Add a time-based backstop.** Take an on-demand-only cached query and add a `revalidate` window. Justify the duration based on the data's tolerance for staleness.

## Key Takeaways

- Time-based revalidation is eventually consistent and uses stale-while-revalidate: the triggering request still sees stale data.
- On-demand revalidation (`revalidatePath`, `revalidateTag`) is for "data changed now" — invalidate it at the mutation.
- Tags decouple invalidation from URLs; design them to mirror your data model with both entity and collection tags.
- Use `unstable_cache` for persistent non-`fetch` data (it honors tags) and React `cache` for per-request dedupe.
- Centralize writes and derive tags from a single source so invalidation can't drift. Keep a modest `revalidate` window as a safety net against forgotten tags.

---

[← Previous: The Next.js Caching Layers](01-caching-layers.md) | [Back to Course Index](../README.md) | [Next: Advanced Data Fetching Patterns →](03-data-fetching-patterns.md)
