# Security Hardening

In a traditional SPA-plus-API setup, the trust boundary is obvious: the browser is untrusted, the API is the gate. The App Router blurs that line. Server Components, Server Actions, and client components live in the same files, and a single careless import can ship a secret to the browser or expose a query to the public internet. Security in Next.js is mostly about **knowing which code runs where**, treating every server entry point as hostile, and never trusting the network. This lesson is the threat model and the hardening checklist.

## What You'll Learn

- The Next.js threat model: where the trust boundary actually sits in the App Router
- Using `import 'server-only'` and avoiding the `NEXT_PUBLIC_` secret-leak trap
- Validating every input at the boundary with Zod, and why "the form already validated it" is not security
- Treating Server Actions as public endpoints: CSRF, origin checks, and what Next does for you
- Setting security headers and a Content Security Policy via config or middleware
- Preventing SSRF in server-side `fetch` and safely rendering user content (XSS, `dangerouslySetInnerHTML`)
- Supply-chain hygiene and the data-exposure risk of over-fetching into client components

## The Next.js Threat Model

Map every piece of your app to a trust zone:

```text
UNTRUSTED (attacker-controlled)        |  TRUSTED (your server)
---------------------------------------|--------------------------------
Browser / client components            |  Server Components
Request body, query, headers, cookies  |  Server Actions (runtime)
Server Action arguments                |  Route Handlers (runtime)
Anything with "use client"             |  Data Access Layer
NEXT_PUBLIC_* env vars                  |  Non-public env vars
                                        |  `server-only` modules
```

The mistakes that cause breaches almost always come from confusing the two columns: shipping a trusted value into the untrusted browser, or treating untrusted input as if a UI already sanitized it. Two rules cover most of it: **secrets and queries stay in the right column**, and **everything crossing from left to right is validated**.

## Keeping Server Code on the Server

### `import 'server-only'`

Any module imported (even transitively) by a client component is bundled and shipped to the browser. A module that touches secrets or the database must refuse to be imported client-side. The `server-only` package turns that mistake into a **build-time error** instead of a production leak:

```ts
// lib/db.ts
import "server-only"; // build fails if a client component imports this, directly or not

import { Pool } from "pg";

// This connection string would otherwise end up in a client bundle.
export const pool = new Pool({ connectionString: process.env.DATABASE_URL });
```

Make this a habit for your DAL, any module reading non-public env vars, and anything with secrets. There's a `client-only` counterpart for browser-only code (e.g., touching `window`).

### The `NEXT_PUBLIC_` trap

Next inlines any `process.env.NEXT_PUBLIC_*` variable into the **client bundle at build time**. The prefix is a one-way ticket to the browser. The trap is naming convenience:

```ts
// WRONG — this key is now visible to every visitor via DevTools
const stripe = new Stripe(process.env.NEXT_PUBLIC_STRIPE_SECRET_KEY!);

// RIGHT — secret stays server-side; only the publishable key is public
const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!); // no prefix
const publishable = process.env.NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY; // intentionally public
```

A second, subtler leak: even non-public secrets can escape if you pass them into a client component as props or include them in serialized data. Server Components serialize their output (and props passed to client children) into the HTML/RSC payload. If you `select *` a user row and pass it to `<ClientProfile user={user} />`, the password hash and email travel to the browser. Pass only what the client needs.

## Input Validation at Every Boundary

Client-side validation is UX. It improves the form experience and does **nothing** for security, because an attacker doesn't use your form — they call your endpoint directly. Validate on the server at every boundary where untrusted data enters, with a schema library like Zod.

```ts
// app/account/actions.ts
"use server";
import { z } from "zod";
import { verifySession } from "@/lib/dal";

const UpdateProfile = z.object({
  displayName: z.string().min(1).max(80),
  bio: z.string().max(500).optional(),
  // Coerce + bound numbers; never trust a "number" from the wire.
  age: z.coerce.number().int().min(13).max(120),
});

export async function updateProfile(raw: unknown) {
  const { userId } = await verifySession();
  // parse() throws on bad input; safeParse() if you want to return field errors.
  const data = UpdateProfile.parse(raw);
  // `data` is now typed AND validated — the only shape that reaches the DB.
  await db.user.update({ where: { id: userId }, data });
}
```

Validate Route Handler bodies the same way, and validate `searchParams` and dynamic `params` before using them in queries. Treat the schema as the boundary contract: nothing untyped reaches your business logic.

## Server Actions Are Public Endpoints

Repeating it because it's the single most misunderstood thing: a Server Action is a POST endpoint with a public, discoverable ID. `"use server"` marks a *boundary*, not a *gate*.

### CSRF and origin checks

Because actions are invoked via POST with cookies attached, they're a CSRF target. Next.js mitigates this: Server Actions only accept `POST`, and Next **compares the `Origin` header against the `Host`** and rejects mismatches by default — a built-in CSRF defense that works as long as the attacker can't forge the `Origin` (browsers won't let them on cross-site requests). You should still:

- Configure `serverActions.allowedOrigins` in `next.config` if you're behind a proxy that rewrites `Host`, so the check doesn't false-positive or false-negative.
- Not rely on `SameSite` cookies *alone* — combine them with the origin check.
- Re-authenticate and re-authorize inside every action regardless (see the auth lesson). Origin checks stop CSRF; they don't stop a legitimately-authenticated user from doing something they shouldn't.

```ts
// next.config.ts
const nextConfig = {
  experimental: {
    serverActions: {
      // Only needed when a reverse proxy changes the Host; pin your real origins.
      allowedOrigins: ["app.example.com", "*.preview.example.com"],
    },
  },
};
export default nextConfig;
```

## Security Headers and CSP

Defense in depth in the browser comes from response headers. Set baseline headers in `next.config`:

```ts
// next.config.ts
const securityHeaders = [
  { key: "X-Content-Type-Options", value: "nosniff" }, // no MIME sniffing
  { key: "X-Frame-Options", value: "DENY" }, // clickjacking
  { key: "Referrer-Policy", value: "strict-origin-when-cross-origin" },
  { key: "Strict-Transport-Security", value: "max-age=63072000; includeSubDomains; preload" },
];

const nextConfig = {
  async headers() {
    return [{ source: "/:path*", headers: securityHeaders }];
  },
};
export default nextConfig;
```

A **Content Security Policy** is your strongest XSS mitigation but needs per-request nonces for inline scripts, so it belongs in middleware:

```ts
// middleware.ts
import { NextResponse } from "next/server";
import type { NextRequest } from "next/server";

export function middleware(req: NextRequest) {
  const nonce = Buffer.from(crypto.randomUUID()).toString("base64");
  const csp = [
    `default-src 'self'`,
    // Next needs the nonce for its inline bootstrap; 'strict-dynamic' lets trusted scripts load others.
    `script-src 'self' 'nonce-${nonce}' 'strict-dynamic'`,
    `style-src 'self' 'unsafe-inline'`, // tighten if you can avoid inline styles
    `img-src 'self' data: https:`,
    `object-src 'none'`,
    `base-uri 'self'`,
    `frame-ancestors 'none'`,
  ].join("; ");

  const headers = new Headers(req.headers);
  headers.set("x-nonce", nonce); // pass to layout so <Script nonce> can read it
  const res = NextResponse.next({ request: { headers } });
  res.headers.set("Content-Security-Policy", csp);
  return res;
}
```

CSP is iterative: start in `Content-Security-Policy-Report-Only` mode, watch the violation reports, then enforce. A reporting-mode CSP that you never enforce is theater — schedule the cutover.

## Avoiding SSRF in Server Fetches

Server-Side Request Forgery happens when user input controls the URL of a server-side `fetch`. Because the fetch runs from *your* server, an attacker can reach internal services, cloud metadata endpoints (`169.254.169.254`), or your private network.

```ts
// WRONG — attacker picks the target; your server makes the request
export async function fetchPreview(url: string) {
  return fetch(url); // SSRF: url = "http://169.254.169.254/latest/meta-data/" leaks IAM creds
}

// BETTER — allowlist, scheme check, and block private ranges
const ALLOWED_HOSTS = new Set(["images.example.com", "cdn.partner.com"]);

export async function fetchPreview(rawUrl: string) {
  const url = new URL(rawUrl); // throws on garbage
  if (url.protocol !== "https:") throw new Error("https only");
  if (!ALLOWED_HOSTS.has(url.hostname)) throw new Error("host not allowed");
  // Note: an allowlist beats a denylist; DNS rebinding can defeat naive IP checks,
  // so prefer fixed, known hosts over "block private IPs" heuristics.
  return fetch(url, { redirect: "error" }); // don't follow redirects into the private net
}
```

The strongest defense is an allowlist of known hosts. Blocking private IP ranges is necessary but insufficient on its own (DNS rebinding, redirects). Disable redirect-following when the user controls the URL.

## Safe Handling of User Content (XSS)

React escapes everything you interpolate by default — `{userInput}` is safe. The danger is the one escape hatch:

```tsx
// DANGEROUS — renders raw HTML; a stored XSS if `html` came from a user
<div dangerouslySetInnerHTML={{ __html: comment.html }} />
```

If you must render user-provided HTML (rich text, markdown), **sanitize on the server** before it reaches React, with a vetted sanitizer:

```ts
// lib/sanitize.ts
import "server-only";
import DOMPurify from "isomorphic-dompurify";

export function sanitizeHtml(dirty: string): string {
  // Strip scripts, event handlers, javascript: URLs. Allow only safe tags.
  return DOMPurify.sanitize(dirty, {
    ALLOWED_TAGS: ["p", "a", "strong", "em", "ul", "ol", "li", "code", "pre"],
    ALLOWED_ATTR: ["href"],
  });
}
```

Prefer rendering markdown to a structured tree (no raw HTML pass-through) over sanitizing arbitrary HTML. Never build HTML strings by concatenating user input.

## Supply-Chain Hygiene

Your `node_modules` runs with your server's privileges. A compromised dependency can read your env vars and exfiltrate your database. Minimum hygiene:

- **Lockfile + CI audit.** Commit the lockfile; run `npm audit --audit-level=high` (or `pnpm audit`) in CI and fail the build on high/critical.
- **Automated updates with review.** Dependabot/Renovate, but read the diffs — don't auto-merge transitive jumps blindly.
- **Minimize the dependency surface.** Every package is attack surface. Prefer the standard library or a small, audited package over a tree of micro-dependencies.
- **Pin and verify.** Use exact versions for security-critical packages; consider provenance/signing (`npm`'s provenance, Sigstore) for what you publish.

## The Over-Fetching Data-Exposure Risk

Tying the lessons together: the most common *quiet* leak isn't an attacker — it's over-fetching. A Server Component fetches a full record and hands it to a client component "to be safe":

```tsx
// LEAK — full user object (hash, email, internal flags) serialized into the RSC payload
const user = await db.user.findUnique({ where: { id } }); // SELECT *
return <ClientProfile user={user} />;

// SAFE — project to exactly what the client renders
const user = await db.user.findUnique({
  where: { id },
  select: { id: true, displayName: true, avatarUrl: true },
});
return <ClientProfile user={user} />;
```

Anything passed to a client component is in the page source. Treat the props of a client component as a public API: select narrowly, and consider an explicit "DTO" mapping step so the shape you expose is deliberate, not accidental.

## Pitfalls

- **`NEXT_PUBLIC_` on a secret** — it's inlined into the client bundle forever (in old deploys too). Rotate the key, not just the code.
- **Validating only on the client.** Attackers skip your form; the server is the only validation that counts.
- **Passing whole DB rows to client components.** The RSC payload is public; `select` the fields you actually render.
- **`dangerouslySetInnerHTML` with unsanitized input** — stored XSS. Sanitize server-side or render structured content.
- **User-controlled fetch URLs** — SSRF into cloud metadata and internal services. Allowlist hosts; don't follow redirects.
- **CSP in report-only forever.** It does nothing until you enforce it.
- **Assuming `"use server"` authorizes.** It marks a boundary; the action is a public endpoint that still needs auth + validation.
- **No CI dependency audit.** A transitive package can read every secret your server holds.

## Exercises

1. Add `import 'server-only'` to your DAL and DB modules, then deliberately import one into a client component and observe the build error. Remove the bad import.
2. Find one secret in your env that's prefixed `NEXT_PUBLIC_` (or audit that none are). For a publishable/secret pair like Stripe, confirm only the publishable key is exposed by grepping the built client bundle.
3. Add Zod validation to a Server Action that currently trusts its arguments. Call the action directly (bypassing the form) with malformed input and confirm it's rejected.
4. Implement a nonce-based CSP in middleware in report-only mode. Trigger a violation, read the report, then switch to enforcing.
5. Audit one Server Component for over-fetching: list every field it passes to client children and replace the query with a narrow `select`.

## Key Takeaways

- Security in the App Router is mostly knowing which trust zone code runs in; confusing server and client zones causes most leaks.
- `import 'server-only'` turns accidental client imports into build errors; the `NEXT_PUBLIC_` prefix ships values to the browser permanently.
- Validate every untrusted input at the boundary with a schema; client validation is UX, not security.
- Server Actions are public POST endpoints — Next's origin check stops CSRF, but you still authenticate, authorize, and validate inside.
- Set baseline security headers in config and a nonce-based CSP in middleware; SSRF is prevented by host allowlists, XSS by sanitizing or avoiding raw HTML.
- Treat client-component props as a public API and audit dependencies in CI — over-fetching and a single bad package are the quiet breaches.

---

[← Previous: Authentication and Authorization Patterns](01-auth-patterns.md) | [Back to Course Index](../README.md) | [Next: Observability →](03-observability.md)
