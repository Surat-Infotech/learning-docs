# Internationalization (i18n)

Going multilingual is rarely a "translate the strings" task. At scale it touches routing, caching, SEO, rendering strategy, and bundle size all at once. A naive setup that ships every translation to every client, breaks static generation, or serves the wrong `hreflang` will quietly cost you traffic and performance in every locale you launch. This lesson treats i18n as the architectural concern it actually is.

## What You'll Learn

- Structuring i18n routing in the App Router with a `[locale]` segment and middleware-driven locale detection
- Managing message catalogs and avoiding shipping every translation to the browser
- Translating Server Components vs Client Components without bloating the client bundle
- Locale-aware formatting of dates, numbers, and currency using the `Intl` API
- Handling right-to-left (RTL) layouts cleanly
- Multilingual SEO: `hreflang`, localized metadata, and canonical URLs
- The static-vs-dynamic implications of per-locale routes and how to keep pages static

## Routing: the `[locale]` segment

The App Router has no built-in i18n config (that was a Pages Router feature). You model locales as a dynamic segment instead. This is more work but far more flexible.

```text
app/
  [locale]/
    layout.tsx
    page.tsx
    products/
      [id]/
        page.tsx
  middleware.ts        // actually at project root, shown here for context
i18n/
  config.ts
  messages/
    en.json
    fr.json
    ar.json
```

```ts
// i18n/config.ts
export const locales = ["en", "fr", "ar"] as const;
export type Locale = (typeof locales)[number];

export const defaultLocale: Locale = "en";

// Locales that render right-to-left.
export const rtlLocales: Locale[] = ["ar"];

export function isLocale(value: string): value is Locale {
  return (locales as readonly string[]).includes(value);
}
```

### Middleware locale detection

Middleware runs before routing, so it's the right place to redirect locale-less URLs (`/products`) to a resolved locale (`/en/products`). Detection order matters: an explicit cookie (user chose a language) should beat the `Accept-Language` header (browser default).

```ts
// middleware.ts
import { NextRequest, NextResponse } from "next/server";
import { match } from "@formatjs/intl-localematcher";
import Negotiator from "negotiator";
import { locales, defaultLocale } from "./i18n/config";

function resolveLocale(req: NextRequest): string {
  // 1. Explicit user choice wins.
  const cookie = req.cookies.get("NEXT_LOCALE")?.value;
  if (cookie && locales.includes(cookie as never)) return cookie;

  // 2. Fall back to the browser's Accept-Language preference.
  const headers = { "accept-language": req.headers.get("accept-language") ?? "" };
  const requested = new Negotiator({ headers }).languages();
  try {
    return match(requested, locales as unknown as string[], defaultLocale);
  } catch {
    return defaultLocale;
  }
}

export function middleware(req: NextRequest) {
  const { pathname } = req.nextUrl;

  // Skip if the path already starts with a known locale.
  const hasLocale = locales.some(
    (l) => pathname === `/${l}` || pathname.startsWith(`/${l}/`),
  );
  if (hasLocale) return;

  const locale = resolveLocale(req);
  const url = req.nextUrl.clone();
  url.pathname = `/${locale}${pathname}`;
  // 307 keeps the method; do not 301 a detection redirect (it gets cached forever).
  return NextResponse.redirect(url, 307);
}

export const config = {
  // Exclude static assets and API routes from rewriting.
  matcher: ["/((?!_next|api|.*\\..*).*)"],
};
```

A subtle production point: middleware runs on every matched request, which forces those requests through the Edge runtime even when the underlying page is static. Keep the matcher tight so you don't accidentally route image and font requests through it.

## Message catalogs with next-intl

You can roll your own, but `next-intl` is the de facto standard for the App Router because it understands the Server/Client boundary. Catalogs are plain JSON, namespaced by feature so you can load only what a route needs.

```json
// i18n/messages/en.json
{
  "Home": {
    "title": "Build faster",
    "cta": "Get started",
    "itemsCount": "{count, plural, =0 {No items} one {# item} other {# items}}"
  }
}
```

```ts
// i18n/request.ts  -- next-intl server config
import { getRequestConfig } from "next-intl/server";
import { isLocale, defaultLocale } from "./config";

export default getRequestConfig(async ({ requestLocale }) => {
  const requested = await requestLocale;
  const locale = isLocale(requested ?? "") ? requested! : defaultLocale;
  return {
    locale,
    messages: (await import(`./messages/${locale}.json`)).default,
  };
});
```

### Translating Server Components

In Server Components you read messages at render time. Nothing ships to the client.

```tsx
// app/[locale]/page.tsx
import { getTranslations, setRequestLocale } from "next-intl/server";

export default async function HomePage({
  params,
}: {
  params: Promise<{ locale: string }>;
}) {
  const { locale } = await params;
  // Required for static rendering of localized pages.
  setRequestLocale(locale);

  const t = await getTranslations("Home");
  return (
    <main>
      <h1>{t("title")}</h1>
      <p>{t("itemsCount", { count: 3 })}</p>
      <a href={`/${locale}/products`}>{t("cta")}</a>
    </main>
  );
}
```

### Translating Client Components

Client Components use a hook, fed by a provider. The critical decision is *which* messages you hand to the provider. Passing the entire catalog re-serializes every string into the HTML and ships it to the browser. Instead, scope it.

```tsx
// app/[locale]/layout.tsx
import { NextIntlClientProvider } from "next-intl";
import { getMessages } from "next-intl/server";
import { rtlLocales, isLocale } from "@/i18n/config";

export default async function LocaleLayout({
  children,
  params,
}: {
  children: React.ReactNode;
  params: Promise<{ locale: string }>;
}) {
  const { locale } = await params;
  const messages = await getMessages();
  const dir = rtlLocales.includes(locale as never) ? "rtl" : "ltr";

  return (
    <html lang={isLocale(locale) ? locale : "en"} dir={dir}>
      <body>
        {/* Pick only the namespaces client components need. */}
        <NextIntlClientProvider messages={{ Home: messages.Home }}>
          {children}
        </NextIntlClientProvider>
      </body>
    </html>
  );
}
```

```tsx
// components/locale-switcher.tsx
"use client";

import { useTranslations } from "next-intl";
import { usePathname, useRouter } from "next/navigation";

export function LocaleSwitcher({ locale }: { locale: string }) {
  const t = useTranslations("Home");
  const router = useRouter();
  const pathname = usePathname();

  function switchTo(next: string) {
    document.cookie = `NEXT_LOCALE=${next};path=/;max-age=31536000`;
    // Swap the leading locale segment.
    router.push(pathname.replace(/^\/[^/]+/, `/${next}`));
  }

  return <button onClick={() => switchTo("fr")}>{t("cta")}</button>;
}
```

## Locale-aware formatting with `Intl`

Never concatenate strings to build dates or currency. The built-in `Intl` API handles separators, ordering, currency symbols, and pluralization rules per locale. It's in every runtime, so it costs you nothing.

```tsx
// lib/format.ts
export function formatMoney(amount: number, locale: string, currency: string) {
  return new Intl.NumberFormat(locale, { style: "currency", currency }).format(
    amount,
  );
}

export function formatDate(date: Date, locale: string) {
  return new Intl.DateTimeFormat(locale, { dateStyle: "long" }).format(date);
}

// formatMoney(1234.5, "en-US", "USD") -> "$1,234.50"
// formatMoney(1234.5, "fr-FR", "EUR") -> "1 234,50 €"
// formatMoney(1234.5, "ar-EG", "EGP") -> "١٬٢٣٤٫٥٠ ج.م.‏"
```

Watch the **server/client time-zone gap**: a `Date` formatted on the server uses the server's zone, then re-rendered on the client uses the browser's zone, producing a hydration mismatch. Pin a `timeZone` explicitly or format on one side only.

## RTL layouts

Direction is a layout concern, not a translation concern. Set `dir` on `<html>` (done above) and write CSS with logical properties so the browser flips automatically.

```css
/* Use logical properties instead of left/right. */
.card {
  margin-inline-start: 1rem; /* flips to the right in RTL */
  padding-inline: 0.5rem 1rem;
  border-inline-start: 2px solid;
}
```

Tailwind users: enable the logical-property utilities (`ms-4`, `pe-2`, `start-0`) and avoid hard-coded `ml-`/`pr-` in shared components. Test the actual RTL render, not just the LTR mirror in your head; icons and chevrons usually need a manual flip.

## SEO for multilingual sites

Search engines need to know which URL serves which language and that they're alternates of one another, not duplicate content.

```tsx
// app/[locale]/products/[id]/page.tsx
import type { Metadata } from "next";
import { locales } from "@/i18n/config";

export async function generateMetadata({
  params,
}: {
  params: Promise<{ locale: string; id: string }>;
}): Promise<Metadata> {
  const { locale, id } = await params;
  const base = "https://example.com";
  const path = `/products/${id}`;

  // Emit hreflang alternates for every locale plus x-default.
  const languages: Record<string, string> = Object.fromEntries(
    locales.map((l) => [l, `${base}/${l}${path}`]),
  );
  languages["x-default"] = `${base}/en${path}`;

  return {
    title: `Product ${id}`,
    alternates: {
      canonical: `${base}/${locale}${path}`,
      languages,
    },
  };
}
```

### Static vs dynamic per-locale routes

Per-locale routing multiplies your page count: 200 pages times 5 locales is 1,000 routes. `generateStaticParams` lets you pre-render them at build time so each locale gets static HTML served from the CDN.

```tsx
// app/[locale]/page.tsx
import { locales } from "@/i18n/config";

export function generateStaticParams() {
  return locales.map((locale) => ({ locale }));
}
```

The trap: if any part of your localized tree reads cookies, headers, or `searchParams`, the whole route opts into dynamic rendering and you lose the static benefit for *every* locale. The locale switcher cookie read in middleware is fine (it runs before the page), but reading `cookies()` inside the page is not. Keep locale resolution at the edge and pass the resolved locale down as a param.

### Trade-offs: where to draw lines

- **next-intl / next-i18next vs hand-rolled** — Libraries give you ICU pluralization, the Server/Client provider split, and type-safe keys for free. Roll your own only for trivial two-locale marketing sites where the dependency isn't worth it.
- **Subpath (`/fr/...`) vs subdomain (`fr.example.com`) vs ccTLD (`example.fr`)** — Subpaths are the App Router default and simplest to operate (one deployment, shared cache). ccTLDs send the strongest geo-targeting signal to search but multiply infra and certs. Pick subpaths unless you have a specific regional-trust or legal-entity reason.
- **Static per-locale vs runtime translation** — Pre-render when content is stable; fetch translations at runtime only when a CMS drives copy that changes hourly. Runtime translation forfeits static rendering.
- **Loading all messages vs namespaced** — Always namespace. The convenience of one global catalog turns into hundreds of KB of duplicated strings in your HTML.

## Pitfalls

- **Shipping the full catalog to the client.** Passing all messages to `NextIntlClientProvider` serializes every locale's worth of strings into HTML. Scope to the namespaces client components actually use.
- **Reading `cookies()`/`headers()` inside localized pages.** It silently opts the route into dynamic rendering, defeating `generateStaticParams` across all locales.
- **301 redirects for locale detection.** A permanent redirect gets cached by browsers and CDNs forever; a user who later switches languages stays stuck. Use 307.
- **Hard-coded `left`/`right` CSS.** Breaks RTL. Use logical properties (`inline-start`, `inline-end`).
- **Hydration mismatches from `Intl` date formatting.** Server and client time zones differ. Pin `timeZone` or format on one side.
- **Forgetting `x-default` in `hreflang`.** Search engines need a fallback for unmatched locales or they pick one arbitrarily.
- **Middleware matcher too broad.** Routing static assets through middleware adds latency to every image and font.

## Exercises

1. Stand up a `[locale]` route tree for `en`, `fr`, and `ar` with middleware that prefers a `NEXT_LOCALE` cookie over `Accept-Language`. Verify a 307 redirect from `/products` to the resolved locale.
2. Add a product detail page with `generateMetadata` emitting correct `hreflang` alternates plus `x-default`, and confirm the canonical points to the current locale.
3. Build a `LocaleSwitcher` client component that sets the cookie and rewrites the path. Confirm only the `Home` namespace ships to the browser by inspecting the HTML payload.
4. Implement RTL for Arabic using logical CSS properties and verify a card component mirrors correctly. Flip any directional icons manually.
5. Add `generateStaticParams`, then deliberately call `cookies()` inside a page and observe in the build output that the route flips from static to dynamic. Remove it and confirm static rendering returns.

## Key Takeaways

- The App Router models i18n as a `[locale]` segment plus middleware detection; there's no built-in config.
- Keep locale resolution at the edge and pass the locale down as a param so pages stay statically renderable.
- Translate Server Components at render time (zero client cost) and scope Client Component messages to needed namespaces.
- Use `Intl` for all locale-aware formatting and pin the time zone to avoid hydration mismatches.
- Treat RTL as a layout concern handled with logical CSS properties and `dir` on `<html>`.
- Multilingual SEO needs `hreflang` alternates, `x-default`, and per-locale canonicals or you risk duplicate-content penalties.

---

[← Previous: CI/CD and Deployment Strategies](../05-production-engineering/05-cicd-deployment.md) | [Back to Course Index](../README.md) | [Next: Caching, CDN, and Multi-Region →](02-cdn-multiregion.md)
