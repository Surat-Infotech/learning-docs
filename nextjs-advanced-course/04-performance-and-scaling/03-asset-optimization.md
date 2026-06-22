# Asset Optimization at Scale

Images, fonts, and third-party scripts are the assets that dominate a typical page's byte weight and most of its layout shift. Get them right and LCP, CLS, and bandwidth all improve at once; get them wrong and no amount of bundle trimming saves you. The subtlety at scale is cost: the built-in image optimizer is a compute service you pay for per transformation, fonts can block render or shift layout, and a single careless third-party tag can cost you more than your entire app bundle. This lesson goes deep on each.

## What You'll Learn

- How `next/image` sizing, `sizes`, `priority`, and formats actually work — and the failure modes
- Configuring remote images securely with `remotePatterns`
- The real cost of the built-in optimizer and when to use an external loader/CDN instead
- Eliminating font-driven layout shift and render-blocking with `next/font`
- Setting cache headers for static and optimized assets
- Loading third-party scripts safely with `next/script` strategies
- Using `preconnect` and `preload` to shorten the critical path

## `next/image`, Deeply

`next/image` does four things: it serves modern formats (AVIF/WebP), it generates a responsive `srcset`, it lazy-loads by default, and it reserves space to prevent CLS. To get all four you must give it the information it needs.

### Sizing and the `sizes` prop

Every image needs intrinsic dimensions so the browser reserves a box before load. You provide them via `width`/`height`, or `fill` plus a positioned, sized parent.

```tsx
// app/components/Card.tsx
import Image from "next/image";

// Fixed-dimension image: width/height reserve the box, preventing CLS.
export function Logo() {
  return <Image src="/logo.png" alt="Logo" width={120} height={40} />;
}

// Responsive image: `fill` stretches to the parent (which must be sized
// and position:relative). `sizes` tells the browser how wide the image
// will render at each breakpoint, so it picks the right srcset entry.
export function Hero() {
  return (
    <div style={{ position: "relative", width: "100%", aspectRatio: "16 / 9" }}>
      <Image
        src="/hero.jpg"
        alt="Hero"
        fill
        sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 800px"
        style={{ objectFit: "cover" }}
        priority
      />
    </div>
  );
}
```

`sizes` is the prop people skip and then wonder why mobile downloads a 2000px image. Without it, a `fill` image defaults to `100vw`, so every device fetches the largest candidate. Describe the rendered width per breakpoint and the browser downloads the smallest sufficient variant — often a 5-10x byte saving on mobile.

### `priority`, formats, and quality

```ts
// next.config.ts
const config: NextConfig = {
  images: {
    // AVIF first (smaller, slower to encode), WebP fallback.
    formats: ["image/avif", "image/webp"],
    // The widths the optimizer will generate. Trim to what you actually use
    // — every extra width is more cache entries and more transforms.
    deviceSizes: [640, 750, 828, 1080, 1200, 1920],
    imageSizes: [16, 32, 48, 64, 96, 128, 256],
  },
};
```

- `priority` preloads the image and disables lazy-loading — use it only on the LCP image (see lesson 1).
- AVIF is ~20-30% smaller than WebP but slower to encode; the cost is paid once per transform and cached. For high-traffic sites the bandwidth win usually justifies it.

### Remote images and `remotePatterns`

Remote sources must be allowlisted or the optimizer refuses them. `remotePatterns` is the secure way (it supports protocol, host, port, and path matching); the older `domains` array is deprecated.

```ts
// next.config.ts
const config: NextConfig = {
  images: {
    remotePatterns: [
      {
        protocol: "https",
        hostname: "cdn.example.com",
        pathname: "/uploads/**", // scope it — don't allow the whole host
      },
      { protocol: "https", hostname: "*.cloudinary.com" },
    ],
  },
};
```

## The Optimizer's Cost and Loader Options

The built-in optimizer transforms images on demand (resize + reformat) and caches the result. That transform is **compute you pay for** — on Vercel it's billed per source image transformed, and self-hosted it consumes CPU and disk on your server. At low volume it's free and excellent. At scale (large catalogs, many unique source images, frequent cache misses) it can dominate your bill or saturate a self-hosted box.

When the optimizer becomes the bottleneck, hand transformation to a dedicated image CDN with a custom loader:

```ts
// image-loader.ts — route every next/image through Cloudinary/imgix/etc.
"use client";
import type { ImageLoaderProps } from "next/image";

export default function cloudinaryLoader({ src, width, quality }: ImageLoaderProps) {
  const params = ["f_auto", "c_limit", `w_${width}`, `q_${quality || "auto"}`];
  return `https://res.cloudinary.com/demo/image/upload/${params.join(",")}/${src}`;
}
```

```ts
// next.config.ts
const config: NextConfig = {
  images: {
    loader: "custom",
    loaderFile: "./image-loader.ts", // Next stops optimizing; the CDN does it
  },
};
```

### Trade-offs: built-in optimizer vs external image CDN

| | Built-in optimizer | External image CDN (loader) |
| --- | --- | --- |
| Setup | Zero | Account + loader file |
| Cost model | Per-transform compute (or self-host CPU) | Per-CDN pricing, usually volume-friendly |
| Caching/edge | Tied to your platform | Global, purpose-built CDN |
| Features | Resize, AVIF/WebP | Smart crop, face detection, watermarks, etc. |
| Best for | Small/medium sites, modest catalogs | Large catalogs, media-heavy, global traffic |

Rule of thumb: start with the built-in optimizer, watch your transform/compute bill, and switch to a CDN loader when image transforms become a top cost line or your self-hosted instance is CPU-bound on resizing.

## `next/font` — No Shift, No Blocking

Web fonts cause two problems: render-blocking (text invisible until the font loads — FOIT) and layout shift (text reflows when the font swaps — FOUT → CLS). `next/font` self-hosts the font (no extra connection to Google), inlines the CSS, preloads the files, and applies `size-adjust` so the fallback's metrics match the real font, eliminating the swap shift.

```tsx
// app/layout.tsx
import { Inter } from "next/font/google";
import localFont from "next/font/local";

// Self-hosted at build time. `display: swap` shows fallback text immediately,
// and size-adjust makes the swap shift-free.
const inter = Inter({
  subsets: ["latin"],
  display: "swap",
  variable: "--font-inter",
});

// A local/brand font you ship yourself.
const brand = localFont({
  src: "./fonts/Brand-Variable.woff2",
  variable: "--font-brand",
  display: "swap",
});

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en" className={`${inter.variable} ${brand.variable}`}>
      <body>{children}</body>
    </html>
  );
}
```

Subset aggressively (`latin` only if you don't need extended ranges), prefer variable fonts (one file covers all weights), and use `woff2`. Each unused subset or weight is bytes on the critical path.

## Static Asset Caching Headers

Files in `/public` are served as-is, and by default without aggressive caching. For fingerprinted assets (Next's build output under `/_next/static`) you want long, immutable caching — Next sets this automatically. For your own static files, set headers explicitly:

```ts
// next.config.ts
const config: NextConfig = {
  async headers() {
    return [
      {
        // Fingerprinted build assets never change for a given URL — cache forever.
        source: "/_next/static/:path*",
        headers: [
          { key: "Cache-Control", value: "public, max-age=31536000, immutable" },
        ],
      },
      {
        // Your own static images/fonts in /public: cache long, but these
        // URLs are NOT fingerprinted, so don't use `immutable` unless you
        // version the filename. Allow revalidation instead.
        source: "/assets/:path*",
        headers: [
          { key: "Cache-Control", value: "public, max-age=86400, stale-while-revalidate=604800" },
        ],
      },
    ];
  },
};
```

The key distinction: `immutable` is safe only when the URL changes whenever the content changes (content hashing). For un-fingerprinted files, use a finite `max-age` plus `stale-while-revalidate` so updates eventually propagate.

## Third-Party Scripts With `next/script`

A single analytics or chat widget can ship more JS than your whole app and block the main thread during the worst possible moment — initial load. `next/script` lets you control *when* a script loads.

```tsx
// app/layout.tsx
import Script from "next/script";

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body>
        {children}

        {/* afterInteractive (default): loads after the page is interactive.
            Good for analytics that must run early but not block paint. */}
        <Script src="https://example.com/analytics.js" strategy="afterInteractive" />

        {/* lazyOnload: loads during idle time. Best for chat widgets,
            social embeds — anything non-essential to the core experience. */}
        <Script src="https://example.com/chat.js" strategy="lazyOnload" />

        {/* beforeInteractive: blocks until loaded. Reserve for critical
            consent/bot-protection scripts only — it hurts FCP/LCP. */}
        <Script src="https://example.com/consent.js" strategy="beforeInteractive" />

        {/* worker (experimental): offloads to a web worker via Partytown,
            keeping the main thread free for interactions. */}
      </body>
    </html>
  );
}
```

Default everything to `lazyOnload` and only promote a script when you have a concrete reason. Most "must load early" claims from third parties are not true for your UX.

### `preconnect` and `preload`

If a third-party domain is on your critical path (a font host, an image CDN, a payment SDK), warm the connection early so the DNS/TLS handshake doesn't serialize behind the request.

```tsx
// app/head-hints.tsx — rendered in layout
export function ResourceHints() {
  return (
    <>
      {/* Open the TCP+TLS connection ahead of time. */}
      <link rel="preconnect" href="https://res.cloudinary.com" crossOrigin="" />
      {/* DNS-only hint for likely-but-not-certain domains (cheaper). */}
      <link rel="dns-prefetch" href="https://widget.intercom.io" />
    </>
  );
}
```

Use `preconnect` sparingly — each one ties up a connection slot. Two or three high-value origins, not a dozen. `next/font` and `next/image` with `priority` already emit the right preloads for their own assets, so don't duplicate them.

### Trade-offs

- **AVIF vs WebP:** AVIF saves bandwidth but costs more encode time per transform. Worth it at high traffic where the transform is cached and amortized; marginal for a tiny site.
- **`beforeInteractive` vs `lazyOnload`:** earlier execution vs faster paint. Almost always favor late loading.
- **Self-hosted fonts vs external font host:** `next/font` self-hosting removes a connection and a render-block; the cost is fonts are bundled into your deploy. Nearly always the right call.

## Pitfalls

- **Omitting `sizes` on `fill`/responsive images.** Mobile downloads the desktop-size image. Always describe rendered width.
- **`priority` on more than the LCP image.** It floods the network and delays the element that actually matters.
- **Un-scoped `remotePatterns` or leftover `domains`.** Allowlist specific hosts/paths; `domains` is deprecated.
- **`immutable` on un-fingerprinted files.** Users get stale assets forever. Version the filename or drop `immutable`.
- **Loading `<link>` to Google Fonts manually.** Adds a connection and a render-block that `next/font` exists to remove.
- **Third-party tags with default `<script>` placement.** They block the main thread at load. Route them through `next/script` with a deferred strategy.
- **Ignoring the optimizer bill.** A large catalog with many unique source images can quietly become your biggest line item; move to a CDN loader before it surprises you.

## Exercises

1. Audit one responsive image: add an accurate `sizes` value and compare the bytes downloaded on a mobile viewport before and after in DevTools.
2. Convert any Google Fonts `<link>` (or manual font CSS) to `next/font`, verify the network panel shows no font-host request, and confirm CLS drops.
3. Configure `formats: ["image/avif", "image/webp"]` and measure the byte difference for your hero image across formats.
4. Take every third-party script in your app and assign each an explicit `next/script` strategy with a one-line justification; demote anything that doesn't need `afterInteractive` to `lazyOnload`.
5. Estimate your monthly image transforms (unique source images × variants). If it's large, prototype a custom loader against an image CDN and compare projected cost.

## Key Takeaways

- `next/image` only delivers its benefits if you feed it correct dimensions and an accurate `sizes`; `priority` belongs on the LCP image alone.
- The built-in optimizer is great until transform compute becomes a cost or CPU bottleneck — then move to an external CDN via a custom loader.
- `next/font` eliminates both render-blocking and font-swap CLS by self-hosting, preloading, and metric-matching fallbacks.
- Cache fingerprinted assets `immutable` forever; cache your own un-versioned files with a finite `max-age` plus `stale-while-revalidate`.
- Route third-party scripts through `next/script` (default to `lazyOnload`) and reserve `preconnect` for a couple of genuinely critical origins.

---

[← Previous: Bundle Optimization and Reducing Client JS](02-bundle-optimization.md) | [Back to Course Index](../README.md) | [Next: Edge, Middleware, and Personalization →](04-edge-and-middleware.md)
