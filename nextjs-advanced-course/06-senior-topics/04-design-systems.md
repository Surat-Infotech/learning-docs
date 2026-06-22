# Design Systems and Component Libraries in Next.js

A design system is leverage: build a button once, every team ships consistent, accessible UI forever. But the React Server Components model changed the rules for component libraries. A library authored for the Pages Router world — one that assumes everything is a client component, leans on a runtime CSS-in-JS engine, and exports a barrel of everything — can sabotage an App Router app's bundle size and rendering strategy. This lesson is about building and consuming design systems that work *with* RSC instead of against it.

## What You'll Learn

- How the Server/Client boundary reshapes design-system authoring
- Which components can stay on the server and which must be marked `"use client"`
- Styling at scale: Tailwind + variants, and why many runtime CSS-in-JS libraries struggle with RSC
- Shipping a component library as a package inside a monorepo
- Theming that survives the server/client split
- Treating accessibility as a non-negotiable, first-class requirement

## The RSC mental model for component libraries

In an App Router app, most UI can render on the server with zero client JS. A design system that forces everything to be a client component throws that away. The author's job is to **maximize what consumers can render on the server** and ship `"use client"` only where interactivity genuinely requires it.

Three categories every component falls into:

- **Server-compatible (no directive).** Pure presentational components: `Card`, `Badge`, `Stack`, `Heading`, typography, layout primitives. No state, no event handlers, no browser APIs. These render on the server and add nothing to the bundle.
- **Client-only (`"use client"`).** Anything with `useState`, `useEffect`, event handlers, refs to DOM, context, or browser APIs: `Dropdown`, `Tabs`, `Dialog`, `Tooltip`, `Accordion`, form inputs with local state.
- **Hybrid / composable.** A server shell that accepts interactive children, or a component split into a server part and a small client part.

### Marking client components correctly

When a component needs interactivity, put the directive on the smallest possible unit, not the whole module.

```tsx
// packages/ui/src/badge.tsx
// No directive: pure, renders on the server, zero client JS.
import { cn } from "./cn";

export function Badge({
  tone = "neutral",
  className,
  ...props
}: React.ComponentProps<"span"> & { tone?: "neutral" | "success" | "danger" }) {
  return (
    <span
      className={cn(
        "inline-flex items-center rounded-full px-2 py-0.5 text-xs font-medium",
        tone === "success" && "bg-green-100 text-green-800",
        tone === "danger" && "bg-red-100 text-red-800",
        tone === "neutral" && "bg-gray-100 text-gray-800",
        className,
      )}
      {...props}
    />
  );
}
```

```tsx
// packages/ui/src/accordion.tsx
"use client"; // needs state -> client component
import { useState } from "react";

export function Accordion({ items }: { items: { title: string; body: React.ReactNode }[] }) {
  const [open, setOpen] = useState<number | null>(null);
  return (
    <div>
      {items.map((item, i) => (
        <div key={i}>
          <button aria-expanded={open === i} onClick={() => setOpen(open === i ? null : i)}>
            {item.title}
          </button>
          {open === i && <div role="region">{item.body}</div>}
        </div>
      ))}
    </div>
  );
}
```

### The "use client" boundary trap

A critical authoring rule: **a `"use client"` component cannot accept a Server Component as a child via direct import, but it can via the `children` prop.** Design your interactive shells to take `children` so consumers can pass server-rendered content into them.

```tsx
// packages/ui/src/dialog.tsx
"use client";
import { useState } from "react";

// Takes `children` — a consumer can pass a Server Component into the dialog body.
export function Dialog({ trigger, children }: { trigger: React.ReactNode; children: React.ReactNode }) {
  const [open, setOpen] = useState(false);
  return (
    <>
      <button onClick={() => setOpen(true)}>{trigger}</button>
      {open && (
        <div role="dialog" aria-modal="true">
          {children /* can be server-rendered content */}
          <button onClick={() => setOpen(false)} aria-label="Close">×</button>
        </div>
      )}
    </>
  );
}
```

A second trap: a barrel `index.ts` that re-exports a `"use client"` component can, depending on bundler behavior, pull client code into modules that only wanted a server component. Prefer explicit per-component entry points or keep the barrel clean and tree-shakeable.

## Styling at scale

### Tailwind + variants (the RSC-friendly default)

Tailwind generates static CSS at build time and adds zero runtime, so it works identically in server and client components. Pair it with a variant helper (`cva` / `tailwind-variants`) to express component APIs without a styling runtime.

```tsx
// packages/ui/src/button.tsx
// No "use client": this presentational button renders on the server.
// (Add the directive only if it manages its own state.)
import { cva, type VariantProps } from "class-variance-authority";
import { cn } from "./cn";

const button = cva(
  "inline-flex items-center justify-center rounded-md font-medium transition focus-visible:outline focus-visible:outline-2",
  {
    variants: {
      intent: {
        primary: "bg-blue-600 text-white hover:bg-blue-700",
        ghost: "bg-transparent text-gray-900 hover:bg-gray-100",
      },
      size: { sm: "h-8 px-3 text-sm", md: "h-10 px-4" },
    },
    defaultVariants: { intent: "primary", size: "md" },
  },
);

type Props = React.ComponentProps<"button"> & VariantProps<typeof button>;

export function Button({ intent, size, className, ...props }: Props) {
  return <button className={cn(button({ intent, size }), className)} {...props} />;
}
```

### Why runtime CSS-in-JS struggles with RSC

Libraries like styled-components and Emotion (in their classic runtime mode) compute styles *at render time using React context and hooks*. Server Components don't run client hooks and don't have that context, so these libraries either don't work in Server Components or force you to mark everything `"use client"` — exactly the bloat you're trying to avoid. They also typically need a `_document`/registry shim to avoid flash-of-unstyled-content with SSR, which is awkward in the App Router.

The viable styling approaches for an RSC design system, ranked by friction:

- **Tailwind (+ variants).** Zero runtime, build-time CSS, works everywhere. Default recommendation.
- **CSS Modules.** Build-time, scoped, framework-native, server-compatible. Great for teams who prefer plain CSS.
- **Zero-runtime CSS-in-JS** (e.g. compile-time extraction like vanilla-extract / Linaria / Panda). CSS-in-JS authoring ergonomics with no render-time cost; server-compatible.
- **Runtime CSS-in-JS** (classic styled-components/Emotion). Use only if you must, accept the `"use client"` cost, and configure the SSR registry. Not recommended for new App Router design systems.

```text
Styling decision:
  Want utility-first + variants?      → Tailwind + cva/tailwind-variants
  Prefer writing plain CSS?            → CSS Modules
  Want CSS-in-JS ergonomics, RSC-safe? → vanilla-extract / Panda (compile-time)
  Have a legacy runtime CSS-in-JS dep? → isolate it, mark "use client", set up registry
```

## Shipping the library in a monorepo

Co-locating the design system with the apps that consume it (Turborepo / pnpm workspaces) gives you instant feedback and shared tooling.

```text
repo/
  apps/
    web/                  // Next.js app, depends on @acme/ui
  packages/
    ui/
      src/
      package.json
      tsconfig.json
  package.json
  pnpm-workspace.yaml
```

```json
// packages/ui/package.json
{
  "name": "@acme/ui",
  "version": "0.0.0",
  "type": "module",
  "exports": {
    "./button": "./src/button.tsx",
    "./badge": "./src/badge.tsx",
    "./dialog": "./src/dialog.tsx"
  },
  "sideEffects": ["**/*.css"],
  "peerDependencies": { "react": "^18 || ^19", "react-dom": "^18 || ^19" }
}
```

Authoring decisions that matter at scale:

- **Per-component export paths** (`@acme/ui/button`) over a single barrel keep tree-shaking honest and stop a client component from leaking into server-only imports.
- **Preserve the `"use client"` directive in your build.** If you transpile/bundle the library, ensure the directive survives to the published output, or consumers' Server Components break. Many teams ship source `.tsx` directly via `exports` and let the consuming Next app transpile it (configure `transpilePackages`).
- **`react`/`react-dom` are peer dependencies**, never bundled, to avoid duplicate React.
- **`sideEffects`** lets bundlers drop unused code while keeping CSS imports.

## Theming across the server/client split

Theming must work without forcing client rendering. CSS custom properties (variables) are the clean answer: set them on a wrapper, components read them via CSS, and nothing needs a JS theme context.

```tsx
// apps/web/app/layout.tsx
export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en" data-theme="light">
      <body>{children}</body>
    </html>
  );
}
```

```css
/* tokens.css */
:root,
[data-theme="light"] { --color-bg: #fff; --color-fg: #111; }
[data-theme="dark"] { --color-bg: #0a0a0a; --color-fg: #fafafa; }

.surface { background: var(--color-bg); color: var(--color-fg); }
```

For a user-toggleable theme, a small `"use client"` component flips `data-theme` (and persists the choice), but the *components themselves stay server-compatible* because they only reference CSS variables. Avoid JS-based theme objects passed through React context as your primary mechanism — that pushes the whole tree client-side.

## Accessibility as a first-class concern

In a design system, accessibility compounds: a button that's accessible once is accessible in a thousand places. Bake it in at the library layer so product teams inherit it.

- **Build interactive primitives on a tested headless library** (Radix UI, React Aria) rather than re-implementing focus traps, roving tabindex, and ARIA wiring by hand. These are client components by nature, which is correct — interactivity belongs on the client.
- **Enforce semantics**: real `<button>`/`<a>`, labeled form controls, `aria-*` on custom widgets, visible focus styles (never `outline: none` without a replacement).
- **Test it**: run `eslint-plugin-jsx-a11y` in CI, add `axe`/`@axe-core/playwright` checks, and verify keyboard navigation and screen-reader output for every interactive component before it ships.
- **Document the contract**: each component's docs state its keyboard interactions and required labels so consumers don't undo them.

### Trade-offs: when to use what

- **Tailwind + variants vs CSS Modules vs zero-runtime CSS-in-JS** — Tailwind for speed and a shared scale; CSS Modules for teams who want plain CSS; compile-time CSS-in-JS for CSS-in-JS ergonomics without the runtime. All three are RSC-safe.
- **Headless library (Radix/React Aria) vs hand-rolled** — Use headless for any non-trivial interactive widget; the accessibility and edge cases are enormous. Hand-roll only the truly trivial.
- **Source-shipping vs pre-built package** — Source-shipping with `transpilePackages` preserves directives effortlessly and is great inside a monorepo; pre-building matters when publishing to an external registry, where you must verify directives survive.
- **CSS-variable theming vs JS context theming** — Prefer CSS variables so components stay server-renderable; reach for JS context only for theming that genuinely requires runtime logic.

## Pitfalls

- **Marking the whole library `"use client"`.** Defeats RSC and bloats every consumer's bundle. Mark only genuinely interactive components.
- **Adopting a runtime CSS-in-JS engine for a new App Router design system.** Forces client rendering and SSR registry shims. Prefer build-time styling.
- **Barrel exports that leak client code.** A single `index.ts` can drag client components into server-only imports. Use per-component entry points.
- **Losing the `"use client"` directive in the build.** Published output without it breaks consumers' server rendering. Verify it survives or ship source.
- **Theming via React context as the default.** Pushes the tree client-side. Use CSS custom properties.
- **Re-implementing accessibility primitives.** Focus management and ARIA are easy to get subtly wrong. Build on a tested headless library.
- **Bundling React into the library.** Causes duplicate-React bugs. Keep it a peer dependency.

## Exercises

1. Build three components in a `packages/ui` workspace: a server-compatible `Badge`, a Tailwind+`cva` `Button`, and a `"use client"` `Dialog` that accepts server-rendered `children`. Confirm only the dialog adds client JS.
2. Set up per-component `exports` in the package and `transpilePackages` in the app. Verify importing `Badge` into a Server Component adds zero client bundle while `Dialog` adds the expected amount.
3. Implement light/dark theming with CSS custom properties plus a tiny `"use client"` toggle that flips `data-theme`. Confirm the styled components themselves remain server components.
4. Wire `eslint-plugin-jsx-a11y` and an `axe` check into CI, then fix at least one violation it surfaces in your components.
5. Take a component currently built on runtime CSS-in-JS (or write one) and migrate it to Tailwind or a compile-time approach, documenting the bundle and rendering-mode difference.

## Key Takeaways

- Author design systems to maximize server-renderable components; mark `"use client"` only on genuinely interactive ones, at the smallest unit.
- Design interactive shells to accept `children` so consumers can pass server-rendered content through the client boundary.
- Prefer build-time styling (Tailwind + variants, CSS Modules, compile-time CSS-in-JS); runtime CSS-in-JS fights RSC.
- Ship per-component export paths, keep React a peer dependency, and ensure `"use client"` directives survive the build (or ship source via `transpilePackages`).
- Theme with CSS custom properties so components stay server-compatible.
- Treat accessibility as a library-level guarantee: build on tested headless primitives and enforce it in CI.

---

[← Previous: Migration and Incremental Adoption](03-migration.md) | [Back to Course Index](../README.md) | [Next: Leading a Next.js Codebase →](05-leading-a-codebase.md)
