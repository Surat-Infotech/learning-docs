# Images, Fonts, and Metadata

Your app looks styled — now let's make it *fast* and *findable*. Next.js has built-in
helpers for three things every real site needs: images, fonts, and metadata. Think of
them as a polishing kit: the image tool shrinks heavy photos, the font tool stops the
page from "jumping," and the metadata tool tells Google what your page is about. 

## What You'll Learn

- Why the `<Image>` component beats a plain `<img>` tag
- How to size images with `width`/`height` or `fill`, and why `alt` matters
- How to load fonts with `next/font` (Google fonts *and* your own files)
- What **layout shift** is and how fonts avoid it
- How to set the page `<title>` and description with the **Metadata API**
- How to generate metadata dynamically with `generateMetadata`
- The basics of **favicons** and **Open Graph** images for social sharing

## The `<Image>` Component

A plain HTML `<img>` works, but it has problems: big photos download slowly, and the
page can jump around as images load. Next.js gives you a smarter `<Image>` component
that fixes these automatically.

```tsx
// app/page.tsx
import Image from "next/image"; // the special Next.js image component

export default function Home() {
  return (
    <Image
      src="/cat.jpg" // file in the public/ folder
      width={400} // tells the browser the size up front
      height={300}
      alt="A sleepy orange cat" // describes the image
    />
  );
}
```

### Why use it over `<img>`?

- **Automatic optimization** — Next.js resizes the image and serves a smaller, modern
  format, so it downloads faster.
- **No layout shift** — because you give a `width` and `height`, the browser reserves
  space, so text doesn't jump when the image arrives.
- **Lazy loading** — images below the fold load only when needed, saving bandwidth.

> **Layout shift** is when content jumps around as the page loads (annoying!).
> Reserving space ahead of time prevents it.

### `width`/`height` vs `fill`

Sometimes you don't know the exact size, or you want the image to fill its container.
Use the `fill` prop instead. The parent must have `position: relative`.

```tsx
// app/components/banner.tsx
import Image from "next/image";

export default function Banner() {
  // the wrapper sets the size; the image fills it
  return (
    <div style={{ position: "relative", width: "100%", height: 300 }}>
      <Image
        src="/beach.jpg"
        fill // stretch to fill the parent box
        alt="A sunny beach"
        style={{ objectFit: "cover" }} // crop nicely instead of squishing
      />
    </div>
  );
}
```

The `alt` text is not optional — it describes the image for screen readers (people who
can't see it) and shows if the image fails to load. Always write a meaningful `alt`.

## Fonts with `next/font`

Custom fonts make a site feel polished, but loading them naively causes that "jump"
again: the page shows a fallback font, then swaps to the real one. The `next/font`
tool loads fonts smartly with **no layout shift**.

### Google fonts

```tsx
// app/layout.tsx
import { Inter } from "next/font/google"; // import the font by name

// load it once, choosing the character sets you need
const inter = Inter({ subsets: ["latin"] });

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  // apply the font's class to the whole body
  return (
    <html lang="en" className={inter.className}>
      <body>{children}</body>
    </html>
  );
}
```

Next.js downloads the font *at build time* and serves it from your own site, so it's
fast and private (no extra request to Google).

### Local fonts (your own files)

```tsx
// app/layout.tsx
import localFont from "next/font/local";

// point at a font file inside your project
const myFont = localFont({ src: "./fonts/Brand-Bold.woff2" });

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en" className={myFont.className}>
      <body>{children}</body>
    </html>
  );
}
```

Either way, you get a `className` you apply to an element, and the font loads without
that ugly swap.

## The Metadata API

**Metadata** is information *about* your page that doesn't show on screen but matters a
lot: the browser tab title, the description search engines display, and the preview
card when someone shares your link. This is the heart of **SEO** (Search Engine
Optimization — helping people find you on Google).

### Static metadata

In any `page.tsx` or `layout.tsx`, export a `metadata` object:

```tsx
// app/about/page.tsx
import type { Metadata } from "next";

// Next.js reads this and builds the right <head> tags for you
export const metadata: Metadata = {
  title: "About Us",
  description: "Learn who we are and what we do.",
};

export default function About() {
  return <h1>About Us</h1>;
}
```

You never write `<title>` or `<meta>` tags by hand — Next.js generates them from this
object.

### Dynamic metadata with `generateMetadata`

When the title depends on data (like a blog post's name), use the `generateMetadata`
function instead. It can be `async` and fetch data.

```tsx
// app/blog/[slug]/page.tsx
import type { Metadata } from "next";

// runs on the server; gets the same params as the page
export async function generateMetadata({
  params,
}: {
  params: Promise<{ slug: string }>;
}): Promise<Metadata> {
  const { slug } = await params; // params is awaited in Next 15
  const post = await getPost(slug); // fetch your data
  return {
    title: post.title, // use the real post title
    description: post.summary,
  };
}

export default async function PostPage() {
  return <article>...</article>;
}
```

### Favicon and Open Graph basics

- **Favicon** — the little icon in the browser tab. Just drop a `favicon.ico` (or
  `icon.png`) into your `app/` folder and Next.js wires it up automatically.
- **Open Graph (OG)** — the preview image and text shown when your link is shared on
  social media. Set it in the metadata:

```tsx
// app/page.tsx
export const metadata = {
  title: "My Cool Site",
  openGraph: {
    title: "My Cool Site",
    description: "The best site on the internet.",
    images: ["/og-preview.png"], // the share-preview image
  },
};
```

Drop an `opengraph-image.png` in a route folder and Next.js even picks it up
automatically — a nice shortcut.

## Common Mistakes

- **Using `<img>` for big photos.** You lose optimization and risk layout shift. Use
  `<Image>`.
- **Forgetting `alt` text.** It's required for accessibility and good practice.
- **Using `fill` without a positioned parent.** The parent needs
  `position: relative` (or absolute/fixed) and a set size.
- **Loading Google fonts with a plain `<link>` tag.** Use `next/font` so they're
  optimized and shift-free.
- **Hand-writing `<title>` tags in JSX.** Export `metadata` instead — Next.js builds
  the head for you.
- **Forgetting to `await params`** in `generateMetadata` on Next 15.

## Exercises

1. Replace an `<img>` on your home page with `<Image>`, giving it proper `width`,
   `height`, and `alt`. Note how the page no longer jumps.
2. Create a full-width banner using `<Image fill>` inside a positioned wrapper.
3. Add a Google font with `next/font/google` and apply it to your whole app via the
   root layout.
4. Add a `metadata` export to your home page with a custom `title` and `description`.
   Check the browser tab.
5. Add `generateMetadata` to a dynamic `[slug]` page so each page's title matches its
   content.

## Key Takeaways

- The `<Image>` component gives **automatic optimization**, **no layout shift**, and
  lazy loading — prefer it over `<img>`.
- Size images with `width`/`height`, or use `fill` inside a positioned parent.
- `next/font` loads Google and local fonts with **no layout shift**.
- The **Metadata API** (`export const metadata`) sets your `<title>` and description
  for SEO.
- Use `generateMetadata` when the title depends on data.
- Drop a `favicon`/`icon` and `opengraph-image` into `app/` and Next.js wires them up.

---

[← Previous: Styling Your App](01-styling.md) | [Back to Course Index](../README.md) | [Next: Route Handlers (API Routes) →](03-route-handlers.md)
