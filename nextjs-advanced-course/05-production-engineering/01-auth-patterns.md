# Authentication and Authorization Patterns

Auth is where most production Next.js apps leak data. The App Router makes it deceptively easy to put a check in the wrong place — a `middleware.ts` file that *feels* like a gate but isn't one. At scale, with dozens of routes, Server Components, Server Actions, and Route Handlers all touching the same data, you need a single principle that survives refactors: **verify authorization as close to the data as possible**. This lesson is about getting that principle into your architecture, not just your code.

## What You'll Learn

- The real trade-offs between session-based and JWT-based auth, and why "stateless" is rarely free
- The "verify close to the data" principle and why middleware-only auth is a security bug
- Why the Next.js middleware-bypass CVE happened and what it teaches about defense in depth
- How to model RBAC and ABAC without painting yourself into a corner
- Reading and setting cookies/headers correctly with `cookies()` and `headers()`, including secure flags
- Protecting Server Actions and Route Handlers as the public endpoints they actually are
- How Auth.js, Clerk, and Lucia differ and when each is the right call

## Session vs JWT: The Real Trade-offs

The "stateless JWT vs stateful session" debate is usually framed wrong. Both store a credential in a cookie. The difference is **where the source of truth lives** and **how fast you can revoke it**.

### Sessions (server-side state)

A random opaque session ID lives in a cookie; the server stores the actual session data (user ID, expiry) in a database or Redis.

```ts
// lib/session.ts
import { cookies } from "next/headers";
import { db } from "@/lib/db";

// Look up the session row on every request. The cookie is just a pointer.
export async function getSession() {
  const cookieStore = await cookies(); // async in Next 15
  const sessionId = cookieStore.get("session")?.value;
  if (!sessionId) return null;

  // Source of truth is the DB — revocation is instant.
  const session = await db.session.findUnique({
    where: { id: sessionId },
    include: { user: true },
  });
  if (!session || session.expiresAt < new Date()) return null;
  return session;
}
```

**Pros:** Instant revocation (delete the row → logged out everywhere). You can store rich state. The cookie value is meaningless if stolen without your DB.
**Cons:** A datastore read on every authenticated request. You need that datastore to be fast and available (Redis, not a cold serverless Postgres connection).

### JWTs (self-contained, signed)

The cookie *is* the credential: a signed token containing the claims (`sub`, `exp`, roles). You verify the signature; no lookup needed.

```ts
// lib/jwt.ts
import { jwtVerify } from "jose"; // edge-compatible, unlike `jsonwebtoken`

const secret = new TextEncoder().encode(process.env.AUTH_SECRET);

export async function verifyJwt(token: string) {
  // Signature + exp validated locally — no network call.
  const { payload } = await jwtVerify(token, secret, {
    algorithms: ["HS256"], // pin the algorithm; never trust the token's header
  });
  return payload;
}
```

**Pros:** No per-request DB read; scales horizontally for free. Works at the edge.
**Cons:** **Revocation is the hard problem.** A valid JWT is valid until it expires. To revoke early you need a denylist (which... reintroduces server state). Short expiries + refresh tokens are the standard mitigation, but now you have a refresh flow to secure. Claims can also go stale — if you bake `role: "admin"` into a 1-hour token and demote the user, they stay admin for up to an hour.

### When to use what

- **Sessions** for most apps. The DB read is cheap with Redis, and instant revocation is worth a lot when handling logout, password resets, and compromised accounts.
- **JWTs** when you genuinely need stateless scaling, cross-service auth, or edge verification — and you accept the revocation complexity. Keep access tokens short (5–15 min) and never put authorization decisions (like roles) you can't tolerate being stale into the token.

A common hybrid: a JWT for fast edge checks of *authentication* (is this a logged-in user?), but re-verify *authorization* (what can they do?) against the DB close to the data.

## The Core Principle: Verify Close to the Data

Here is the rule that matters more than any library choice:

> Every code path that reads or mutates protected data must verify authorization itself — in the Data Access Layer, the Server Component, or the Server Action — **not** rely on a check that happened "earlier" in middleware.

### Why middleware-only auth is insufficient

Middleware runs *before* a request is matched to a route. It's tempting to write:

```ts
// middleware.ts — DO NOT rely on this as your only gate
import { NextResponse } from "next/server";
import type { NextRequest } from "next/server";

export function middleware(req: NextRequest) {
  const isLoggedIn = req.cookies.has("session");
  if (!isLoggedIn && req.nextUrl.pathname.startsWith("/dashboard")) {
    return NextResponse.redirect(new URL("/login", req.url));
  }
}

export const config = { matcher: ["/dashboard/:path*"] };
```

This is fine as a **UX optimization** (redirect anonymous users to login). It is a **disaster as your security boundary**, for several reasons:

1. **It doesn't cover everything.** A Server Action invoked from a public page, or a Route Handler hit directly with `fetch`, may not match your `matcher`. Data leaks through the paths middleware never saw.
2. **Matcher logic is fragile.** Path-based matching drifts as routes are added. One forgotten path = one open door. Authorization that depends on remembering to update a regex is not authorization.
3. **It's authentication, not authorization.** "Has a session cookie" is not "is allowed to see *this* record." Object-level checks (does this invoice belong to this user?) cannot live in middleware — middleware doesn't know which invoice.
4. **CVE-2025-29927.** A vulnerability allowed attackers to **bypass Next.js middleware entirely** by sending a crafted `x-middleware-subrequest` header — meaning any auth check done *only* in middleware was skipped. Apps that treated middleware as their sole gate were instantly exploitable. The lesson isn't "patch Next" (do that too); it's that **a single layer you can spoof past is not a security boundary.** Defense in depth means the real check lives where the data lives, so even a totally bypassed middleware leaks nothing.

Treat middleware as a coarse, best-effort filter and a place for cross-cutting concerns (redirects, headers, A/B routing). Never as the thing standing between an attacker and your database.

### The Data Access Layer (DAL) pattern

Centralize data reads behind functions that verify auth internally. The rest of the app calls these functions and physically cannot reach the data without passing the check.

```ts
// lib/dal.ts
import "server-only"; // build error if this is ever imported into a client bundle
import { cache } from "react";
import { redirect } from "next/navigation";
import { cookies } from "next/headers";
import { db } from "@/lib/db";

// `cache` dedupes this across one request — call it freely in every component.
export const verifySession = cache(async () => {
  const cookieStore = await cookies();
  const sessionId = cookieStore.get("session")?.value;
  if (!sessionId) redirect("/login");

  const session = await db.session.findUnique({
    where: { id: sessionId },
    include: { user: { select: { id: true, role: true, orgId: true } } },
  });
  if (!session || session.expiresAt < new Date()) redirect("/login");

  // Return only the minimal identity the app needs — never the whole user row.
  return { userId: session.user.id, role: session.user.role, orgId: session.user.orgId };
});

// Object-level authorization lives WITH the query, not in middleware.
export async function getInvoice(invoiceId: string) {
  const { userId, orgId } = await verifySession();
  const invoice = await db.invoice.findUnique({ where: { id: invoiceId } });

  // The check the middleware could never do: does this row belong to this tenant?
  if (!invoice || invoice.orgId !== orgId) {
    throw new Error("Not found"); // don't leak existence; 404, not 403
  }
  return invoice;
}
```

Now a Server Component is trivially safe:

```tsx
// app/invoices/[id]/page.tsx
import { getInvoice } from "@/lib/dal";

export default async function InvoicePage({ params }: { params: Promise<{ id: string }> }) {
  const { id } = await params;
  const invoice = await getInvoice(id); // auth + ownership enforced inside
  return <h1>{invoice.number}</h1>;
}
```

There is no path to `invoice` data that skips the check, because the check *is* the data access. Forgetting to authorize becomes hard rather than easy — exactly what you want from an architecture.

## Modeling Authorization: RBAC and ABAC

**RBAC (Role-Based Access Control):** permissions attach to roles, roles attach to users. Simple, auditable, the right default.

```ts
// lib/rbac.ts
const PERMISSIONS = {
  admin: ["invoice:read", "invoice:write", "user:manage"],
  member: ["invoice:read", "invoice:write"],
  viewer: ["invoice:read"],
} as const;

type Role = keyof typeof PERMISSIONS;
type Permission = (typeof PERMISSIONS)[Role][number];

export function can(role: Role, permission: Permission): boolean {
  return (PERMISSIONS[role] as readonly string[]).includes(permission);
}
```

**ABAC (Attribute-Based Access Control):** decisions depend on attributes of the user, resource, and context (ownership, org membership, time, IP). RBAC answers "can admins delete invoices?"; ABAC answers "can *this* user delete *this* invoice *right now*?"

```ts
// lib/abac.ts — combine role + resource attributes
import { can } from "./rbac";

export function canEditInvoice(
  user: { id: string; role: "admin" | "member" | "viewer"; orgId: string },
  invoice: { orgId: string; createdById: string; locked: boolean }
): boolean {
  if (invoice.orgId !== user.orgId) return false; // tenant isolation
  if (invoice.locked && user.role !== "admin") return false; // contextual rule
  if (!can(user.role, "invoice:write")) return false; // role gate
  return user.role === "admin" || invoice.createdById === user.id; // ownership
}
```

### When to use what

Start with **RBAC** — most authorization is "what kind of user are you?" Reach for **ABAC** when you hit ownership, multi-tenancy, or contextual rules ("only during business hours", "only your own records"). In practice mature apps are RBAC for coarse gates plus ABAC checks at the object level inside the DAL. Don't build a full policy engine on day one; you can grow into one (Oso, Cedar, OpenFGA) when the `if` statements get unwieldy.

## Cookies, Headers, and Secure Flags

When you set the auth cookie yourself, the flags are the security:

```ts
// app/login/actions.ts
"use server";
import { cookies } from "next/headers";

export async function setSessionCookie(sessionId: string) {
  const cookieStore = await cookies();
  cookieStore.set("session", sessionId, {
    httpOnly: true, // JS can't read it → blunts XSS token theft
    secure: process.env.NODE_ENV === "production", // HTTPS only
    sameSite: "lax", // sent on top-level nav, blocks most CSRF
    path: "/",
    maxAge: 60 * 60 * 24 * 7, // 7 days; match your server-side expiry
  });
}
```

`httpOnly` is non-negotiable for auth cookies — without it, any XSS becomes account takeover. Use `sameSite: "lax"` unless you have a cross-site embedding need; `strict` breaks inbound links from email. Read request headers with `await headers()` (also async in Next 15), but treat client-supplied headers as untrusted input.

## Protecting Server Actions and Route Handlers

A Server Action compiles to a **public HTTP endpoint**. The `"use server"` boundary does not authorize anything. Anyone who can find the action ID can invoke it with arbitrary arguments.

```ts
// app/invoices/actions.ts
"use server";
import { verifySession } from "@/lib/dal";
import { can } from "@/lib/rbac";
import { z } from "zod";
import { db } from "@/lib/db";

const DeleteInput = z.object({ invoiceId: z.string().uuid() });

export async function deleteInvoice(raw: unknown) {
  // 1. Authenticate — every action, every time.
  const { role, orgId } = await verifySession();
  // 2. Authorize the action type.
  if (!can(role, "invoice:write")) throw new Error("Forbidden");
  // 3. Validate input — arguments are attacker-controlled.
  const { invoiceId } = DeleteInput.parse(raw);
  // 4. Authorize the specific object (tenant scope in the query).
  const result = await db.invoice.deleteMany({ where: { id: invoiceId, orgId } });
  if (result.count === 0) throw new Error("Not found");
}
```

Route Handlers are the same story — they're just HTTP:

```ts
// app/api/invoices/[id]/route.ts
import { getInvoice } from "@/lib/dal"; // auth lives in the DAL call
import { NextResponse } from "next/server";

export async function GET(_req: Request, { params }: { params: Promise<{ id: string }> }) {
  const { id } = await params;
  try {
    return NextResponse.json(await getInvoice(id));
  } catch {
    return new NextResponse("Not found", { status: 404 });
  }
}
```

## Libraries: Auth.js, Clerk, Lucia

- **Auth.js (NextAuth v5):** Free, open source, owns your data, huge provider catalog (OAuth, email, credentials). Flexible but you assemble the pieces (adapter, callbacks, session strategy) and own the security decisions. Best when you want control and no per-MAU bill.
- **Clerk:** Managed auth-as-a-service. Pre-built UI, org/RBAC, MFA, bot protection out of the box. Fastest to production and offloads security to a specialist — but it's a paid dependency, your users live in their system, and you're coupled to their SDK. Best for teams that want auth solved, not built.
- **Lucia:** Not a library anymore — it's now a **learning resource** showing you how to implement sessions yourself with your DB. The right "library" when you want zero auth dependency and full understanding, at the cost of writing (and maintaining) the code.

### When to use what

Greenfield product, ship fast, budget for SaaS → **Clerk**. Want ownership, self-hosted, comfortable wiring callbacks → **Auth.js**. Want to truly understand and control the session lifecycle, or have unusual requirements → roll your own following the **Lucia** patterns. Whatever you choose, the DAL/verify-close-to-data principle is identical — the library only manages *how you know who the user is*, not *where you enforce what they can do*.

## Pitfalls

- **Middleware as the only gate.** It's a UX redirect, not security. CVE-2025-29927 let attackers skip it entirely. Verify in the DAL.
- **Trusting JWT claims for authorization that can change.** Roles baked into a long-lived token go stale; demoted users keep their power until expiry.
- **Missing object-level checks.** Authenticating the user but not verifying they own *this* record — the classic IDOR / broken-object-level-authorization bug. Always scope queries by tenant/owner.
- **Forgetting `await` on `cookies()`/`headers()`** in Next 15 — they're async now.
- **Returning 403 instead of 404** for resources a user can't see — leaks existence. Return "Not found".
- **No `httpOnly` on the auth cookie** — turns any XSS into full account takeover.
- **Authorizing Server Actions by hiding the button.** The endpoint is public regardless of UI; the check must be server-side.

## Exercises

1. Take a route currently protected only by `middleware.ts` and move the real authorization into a `verifySession()`-backed DAL function. Then delete the cookie and confirm the data endpoint refuses you even when you call the Route Handler directly with `curl`.
2. Implement `getInvoice(id)` with tenant scoping. Write a test that a user from org A gets a 404 (not 403) for an org B invoice.
3. Build the `can()` RBAC helper and add it to three Server Actions. Then add one ABAC rule (e.g., locked invoices are admin-only) and explain where it had to live and why.
4. Add a denylist to a JWT setup so you can revoke a token before expiry. Discuss what state you just reintroduced and whether sessions would have been simpler.
5. Audit one feature for IDOR: list every Server Action and Route Handler that takes an ID, and verify each scopes its query by owner or org.

## Key Takeaways

- Authorization belongs **close to the data** — in the DAL, Server Component, or Action — never solely in middleware.
- Middleware is a UX filter you can be spoofed past (CVE-2025-29927); it is not a security boundary.
- Sessions buy instant revocation at the cost of a DB read; JWTs buy stateless scale at the cost of revocation complexity. Don't put mutable authz decisions in long-lived tokens.
- Server Actions and Route Handlers are public endpoints: authenticate, authorize the action, validate input, then authorize the specific object — every time.
- RBAC for coarse gates, ABAC at the object level; centralize both in the DAL so forgetting to check is hard.
- Set `httpOnly`, `secure`, `sameSite` on auth cookies, and remember `cookies()`/`headers()` are async in Next 15.

---

[← Previous: Scaling and Infrastructure Considerations](../04-performance-and-scaling/05-scaling-infrastructure.md) | [Back to Course Index](../README.md) | [Next: Security Hardening →](02-security.md)
