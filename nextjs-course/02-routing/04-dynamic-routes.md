# Dynamic Routes

Imagine a blog with a thousand posts. You would not create a thousand folders by
hand. Instead you create **one** folder that acts as a template: "any post slug
goes here". This is a **dynamic route**: a single route definition that matches
many different URLs and reads the changing part as a value. Think of it like a
mailbox slot labelled "for any name", where the actual name on each letter is
read when it arrives. In this lesson we build exactly that.

## What You'll Learn

- How `[brackets]` create a dynamic URL segment
- How to read the value from `params` inside a page
- That in Next.js 15 `params` is a Promise you must `await`
- How to read query strings with `searchParams`
- Catch-all routes `[...slug]` and optional catch-all `[[...slug]]`
- How `generateStaticParams` pre-builds dynamic pages
- A complete worked blog post example

## Creating a Dynamic Segment

To make a segment dynamic, wrap the folder name in square brackets. The name
inside the brackets becomes the **parameter name**.

```text
app/
  blog/
    [slug]/
      page.tsx     ->  matches /blog/anything
```

Now `/blog/hello-world`, `/blog/next-tips`, and `/blog/whatever` all hit the
same `page.tsx`. The part that changes (`hello-world`, etc.) is captured under
the name `slug`.

## Reading `params`

Next.js passes the captured values to your page through a prop called `params`.
**In Next.js 15, `params` is a Promise**, so you must `await` it. Make the page
function `async` and await `params` to get the values.

```tsx
// app/blog/[slug]/page.tsx
export default async function BlogPostPage({
  params,
}: {
  params: Promise<{ slug: string }>; // a Promise in Next 15
}) {
  const { slug } = await params; // unwrap it
  return <h1>Reading post: {slug}</h1>;
}
```

Visiting `/blog/hello-world` shows "Reading post: hello-world". The folder name
`[slug]` matched the parameter name `slug`, and `params` carried its value.

If your folder were `[id]`, you would read `const { id } = await params`. The key
always matches the bracket name.

## Reading `searchParams`

A URL can also carry a **query string**, the part after a `?`, like
`/blog?page=2&sort=new`. Those are read through `searchParams`, which (like
`params`) is a Promise in Next.js 15.

```tsx
// app/blog/page.tsx
export default async function BlogPage({
  searchParams,
}: {
  searchParams: Promise<{ page?: string; sort?: string }>;
}) {
  const { page, sort } = await searchParams;
  return (
    <p>
      Page: {page ?? "1"}, sorted by: {sort ?? "default"}
    </p>
  );
}
```

For `/blog?page=2&sort=new`, this shows "Page: 2, sorted by: new". Query values
are always strings (or undefined if missing), so convert with `Number(page)` if
you need a number.

The difference in one line: `params` is part of the **path** (`/blog/[slug]`),
while `searchParams` is the optional **query** after the `?`.

## Catch-All Routes

A normal `[slug]` matches exactly one segment. To match *several* segments at
once, use a **catch-all** with three dots: `[...slug]`.

```text
app/
  docs/
    [...slug]/
      page.tsx
```

This matches `/docs/a`, `/docs/a/b`, `/docs/a/b/c`, and so on. The captured value
is an **array** of the segments.

```tsx
// app/docs/[...slug]/page.tsx
export default async function DocsPage({
  params,
}: {
  params: Promise<{ slug: string[] }>; // an array now
}) {
  const { slug } = await params;
  // for /docs/a/b/c  ->  slug is ["a", "b", "c"]
  return <p>Path parts: {slug.join(" / ")}</p>;
}
```

One catch: a plain `[...slug]` does **not** match `/docs` itself (with nothing
after it). If you want it to match the bare `/docs` too, use the
**optional catch-all** with double brackets: `[[...slug]]`.

```text
app/
  docs/
    [[...slug]]/
      page.tsx     ->  matches /docs AND /docs/a/b/c
```

```tsx
// app/docs/[[...slug]]/page.tsx
export default async function DocsPage({
  params,
}: {
  params: Promise<{ slug?: string[] }>; // optional, may be undefined
}) {
  const { slug } = await params;
  if (!slug) return <h1>Docs home</h1>; // for /docs with no extra path
  return <p>Path parts: {slug.join(" / ")}</p>;
}
```

## Pre-Rendering with `generateStaticParams`

By default, a dynamic page is built when someone first requests it. But if you
know all the possible values ahead of time (for example, all your blog slugs),
you can tell Next.js to build the pages in advance for speed. You do this by
exporting a `generateStaticParams` function that returns the list of params.

```tsx
// app/blog/[slug]/page.tsx
export async function generateStaticParams() {
  // return one object per page you want pre-built
  return [{ slug: "hello-world" }, { slug: "next-tips" }];
}

export default async function BlogPostPage({
  params,
}: {
  params: Promise<{ slug: string }>;
}) {
  const { slug } = await params;
  return <h1>Post: {slug}</h1>;
}
```

At build time, Next.js calls `generateStaticParams`, sees two slugs, and
generates `/blog/hello-world` and `/blog/next-tips` as ready-made pages. We will
go deeper into static rendering later; for now just know this is how you
pre-build dynamic pages.

## Worked Example: A Blog Post Page

Let us tie it together. We have a small list of posts and a dynamic page that
shows one post by its slug, falling back gracefully if the slug is unknown.

```tsx
// app/blog/posts.ts
export const posts = [
  { slug: "hello-world", title: "Hello World", body: "My first post." },
  { slug: "next-tips", title: "Next Tips", body: "Routing is fun." },
];
```

```tsx
// app/blog/[slug]/page.tsx
import { posts } from "../posts";

// pre-build a page for every known post
export async function generateStaticParams() {
  return posts.map((post) => ({ slug: post.slug }));
}

export default async function BlogPostPage({
  params,
}: {
  params: Promise<{ slug: string }>;
}) {
  const { slug } = await params; // e.g. "hello-world"
  const post = posts.find((p) => p.slug === slug);

  if (!post) {
    // unknown slug: show a simple message (we'll learn notFound() next lesson)
    return <p>Sorry, that post does not exist.</p>;
  }

  return (
    <article>
      <h1>{post.title}</h1>
      <p>{post.body}</p>
    </article>
  );
}
```

Visiting `/blog/hello-world` renders the "Hello World" post; visiting
`/blog/unknown` shows the fallback message. One file serves every post.

## Common Mistakes

- **Forgetting to `await params`.** In Next.js 15 `params` is a Promise. Reading
  `params.slug` directly (without `await`) gives the wrong value or a warning.
  Make the component `async` and `await params`.
- **Mismatching the bracket name.** The key you destructure must match the folder
  name: `[slug]` gives `{ slug }`, `[id]` gives `{ id }`.
- **Confusing `params` and `searchParams`.** `params` is the dynamic path
  segment; `searchParams` is the query string after `?`.
- **Expecting `[...slug]` to match the bare parent.** It does not match `/docs`;
  use `[[...slug]]` if you need the parent path to match too.
- **Assuming searchParams are numbers.** They are strings. Convert with
  `Number(...)` before doing math.

## Exercises

1. Create `app/products/[id]/page.tsx` that reads `id` from `params` (remember to
   `await`) and shows "Product: {id}".
2. Build a `app/search/page.tsx` that reads a `q` value from `searchParams` and
   shows "Searching for: {q}". Test it with `/search?q=shoes`.
3. Make `app/docs/[...slug]/page.tsx` that joins the segments with " > " and
   displays them. Try `/docs/setup/install`.
4. Convert the docs route to an optional catch-all so `/docs` shows a "Docs home"
   message while deeper paths still work.
5. Add `generateStaticParams` to a blog `[slug]` page returning three slugs, then
   run `npm run build` and find those three pages listed in the output.

## Key Takeaways

- Wrap a folder name in `[brackets]` to create a **dynamic segment** that matches
  many URLs.
- Read the captured value from `params`; in **Next.js 15 `params` is a Promise**,
  so `await` it (`const { slug } = await params`).
- `searchParams` reads the query string after `?` and is also a Promise to
  await; its values are strings.
- `[...slug]` is a **catch-all** (an array of segments); `[[...slug]]` is an
  **optional catch-all** that also matches the bare parent path.
- `generateStaticParams` lists known params so Next.js can **pre-render** those
  dynamic pages at build time.

---

[← Previous: Linking and Navigation](03-linking-and-navigation.md) | [Back to Course Index](../README.md) | [Next: Route Groups and Nested Layouts →](05-route-groups-nested-layouts.md)
