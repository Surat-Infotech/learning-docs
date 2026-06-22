# Performance and Optimization

A fast app feels good. A slow app makes people leave before they even see what
you built. The good news is that Next.js does most of the hard performance work
for you — your job is mostly to **stay out of its way** and follow a few habits.

Think of your app like a delivery van. Every kilogram of JavaScript you send to
the browser is extra weight the van has to carry. The lighter the van, the faster
it arrives. Most of this lesson is about keeping the van light and packing it
smartly.

## What You'll Learn

- Why leaning on Server Components ships less JavaScript and feels faster
- How `<Image>` and `next/font` save bytes for free
- How to load heavy code only when needed with `next/dynamic`
- How caching and streaming keep pages quick (a quick recap)
- How to keep your `"use client"` boundaries small
- How to measure your bundle with `@next/bundle-analyzer`
- What Core Web Vitals are, in plain terms
- How to avoid "waterfalls" by fetching data in parallel

## Server Components Are Your Best Performance Tool

The single biggest win is something you already have: **Server Components ship no
JavaScript to the browser**. They render to HTML on the server. Less JavaScript
means a faster first load and a snappier page.

```tsx
// app/page.tsx
// No "use client" -> 0 JS for this component in the browser.

export default async function HomePage() {
  const posts = await getPosts(); // runs on the server
  return (
    <ul>
      {posts.map((p) => (
        <li key={p.id}>{p.title}</li>
      ))}
    </ul>
  );
}
```

So the first rule of performance is simply: **keep components on the server
unless they truly need to be on the client.**

## Keep `"use client"` Boundaries Small

When a file says `"use client"`, everything it imports becomes client code too.
So do not slap `"use client"` at the top of a big page. Instead, push it down to
the smallest piece that needs interactivity.

```tsx
// Bad: the whole page becomes client code just for one button.
// app/page.tsx
"use client";
import { useState } from "react";
// ...hundreds of lines, all shipped to the browser
```

```tsx
// Good: only the tiny button is a Client Component.
// app/components/LikeButton.tsx
"use client";
import { useState } from "react";

export default function LikeButton() {
  const [likes, setLikes] = useState(0);
  return <button onClick={() => setLikes(likes + 1)}>{likes}</button>;
}
```

```tsx
// app/page.tsx  (stays a Server Component)
import LikeButton from "./components/LikeButton";

export default function HomePage() {
  return (
    <main>
      <h1>My blog</h1>
      <LikeButton /> {/* only this island ships JS */}
    </main>
  );
}
```

## Images and Fonts: Free Wins

You met these earlier. They matter so much for speed that they deserve a recap.

### `<Image>` resizes and lazy-loads for you

```tsx
// app/page.tsx
import Image from "next/image";

export default function HomePage() {
  return (
    <Image
      src="/hero.jpg"
      alt="A friendly robot"
      width={800}
      height={400}
      priority // load this big above-the-fold image first
    />
  );
}
```

The `<Image>` component serves the right size for each screen, uses modern
formats, and lazy-loads images that are off-screen — all automatically.

### `next/font` avoids slow font loading

```tsx
// app/layout.tsx
import { Inter } from "next/font/google";

const inter = Inter({ subsets: ["latin"] });

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en" className={inter.className}>
      <body>{children}</body>
    </html>
  );
}
```

`next/font` downloads and hosts the font at build time, so there is no extra
network request and no flicker while the font loads.

## Loading Heavy Code Only When Needed

Some components are big (a chart library, a rich text editor) and not needed
right away. Use `next/dynamic` to **lazy-load** them — that means the code only
downloads when the component actually renders.

```tsx
// app/dashboard/page.tsx
import dynamic from "next/dynamic";

// HeavyChart is downloaded only when this page is shown, not before.
const HeavyChart = dynamic(() => import("../components/HeavyChart"), {
  loading: () => <p>Loading chart...</p>, // shown while it downloads
});

export default function Dashboard() {
  return (
    <div>
      <h1>Stats</h1>
      <HeavyChart />
    </div>
  );
}
```

This keeps your initial bundle (the first chunk of JavaScript the browser loads)
small, so the page becomes interactive sooner.

## Caching and Streaming: A Quick Recap

Two features you have already seen also boost speed:

- **Caching** lets Next.js reuse a result instead of redoing work. A `fetch` can
  be cached so repeat visitors get an instant response.

```tsx
// Cached: served fast, refreshed at most once an hour.
const res = await fetch("https://api.example.com/posts", {
  next: { revalidate: 3600 }, // seconds
});
```

- **Streaming** with `loading.tsx` and `<Suspense>` lets the page show instantly
  while slow parts fill in. The user sees something right away instead of staring
  at a blank screen.

```tsx
// app/page.tsx
import { Suspense } from "react";

export default function HomePage() {
  return (
    <main>
      <h1>Home</h1> {/* shows immediately */}
      <Suspense fallback={<p>Loading feed...</p>}>
        <SlowFeed /> {/* streams in when ready */}
      </Suspense>
    </main>
  );
}
```

## Avoiding Waterfalls with Parallel Fetching

A **waterfall** is when you wait for one request to finish before starting the
next, even though they do not depend on each other. Each `await` on its own line
runs one at a time.

```tsx
// Slow: these run one after another (a waterfall).
const user = await getUser();     // wait...
const posts = await getPosts();   // ...then wait again
```

If the two requests are independent, start them together with `Promise.all` so
they run **in parallel**.

```tsx
// Fast: both start at once, then we wait for both.
const [user, posts] = await Promise.all([getUser(), getPosts()]);
```

Only keep them sequential when the second genuinely needs the first (for example,
you need the user's ID before you can fetch that user's posts).

## Measuring Your Bundle

You can not improve what you do not measure. `@next/bundle-analyzer` draws a map
of what is inside your JavaScript bundle, so you can spot a surprisingly large
library.

```bash
# Install the analyzer
npm install @next/bundle-analyzer
```

```ts
// next.config.ts
import bundleAnalyzer from "@next/bundle-analyzer";

const withAnalyzer = bundleAnalyzer({
  enabled: process.env.ANALYZE === "true", // only when we ask for it
});

export default withAnalyzer({});
```

```bash
# Build with analysis turned on; a chart opens in your browser
ANALYZE=true npm run build
```

If you see a giant block for a library you barely use, that is a clue to lazy-load
it or find a lighter alternative.

## Core Web Vitals (The High-Level View)

Google measures real-world speed with three **Core Web Vitals**. You do not need
to memorize the math — just know what each one means.

- **LCP (Largest Contentful Paint):** how fast the biggest thing on screen
  appears. Helped by `<Image priority>` and Server Components.
- **CLS (Cumulative Layout Shift):** how much the page jumps around while
  loading. Helped by giving images a `width` and `height` so space is reserved.
- **INP (Interaction to Next Paint):** how quickly the page responds to a click
  or tap. Helped by shipping less client JavaScript.

The same habits in this lesson — fewer client components, optimized images,
parallel fetching — improve all three.

## Common Mistakes

- **Putting `"use client"` at the top of a whole page.** It drags all imported
  code into the browser. Push it down to the smallest interactive piece.
- **Importing heavy libraries eagerly.** A chart or editor used on one page should
  be lazy-loaded with `next/dynamic`.
- **Creating waterfalls.** Independent `await`s on separate lines run one at a
  time. Use `Promise.all` to run them together.
- **Using plain `<img>` tags.** You lose automatic resizing, modern formats, and
  layout-shift protection that `<Image>` gives you.
- **Guessing about bundle size.** Measure with the analyzer before optimizing.

## Exercises

1. Find a page that has `"use client"` at the top and move the directive down into
   the one small component that actually needs it.
2. Convert a plain `<img>` to `<Image>` with proper `width` and `height`, and add
   `priority` to your main hero image.
3. Take a heavy component and load it with `next/dynamic`, including a `loading`
   fallback. Confirm in the network tab that its code loads only when shown.
4. Turn two sequential, independent `await` fetches into a single `Promise.all`
   call.
5. Set up `@next/bundle-analyzer`, run `ANALYZE=true npm run build`, and write
   down the largest item in your bundle.

## Key Takeaways

- Server Components are your biggest performance tool because they ship no
  JavaScript; keep code on the server when you can.
- Keep `"use client"` boundaries as small as possible.
- Use `<Image>` and `next/font` for free image and font optimizations.
- Lazy-load heavy components with `next/dynamic`, and reuse work with caching and
  streaming.
- Fetch independent data in parallel with `Promise.all` to avoid waterfalls, and
  measure your bundle before optimizing.

---

[← Previous: Error Handling in Production](01-error-handling.md) | [Back to Course Index](../README.md) | [Next: Testing →](03-testing.md)
