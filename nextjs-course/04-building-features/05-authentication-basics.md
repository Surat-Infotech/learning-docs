# Authentication Basics

Most real apps need to know *who* a user is — to show their profile, save their data,
or keep certain pages private. That's **authentication**. This lesson keeps things
gentle and conceptual: we'll learn the core ideas, peek at how Next.js reads login
info on the server, and protect a page. Think of authentication like a club's front
door: the bouncer checks your wristband before letting you in. 

## What You'll Learn

- The difference between **authentication** and **authorization**
- What **sessions**, **JWTs**, and **cookies** are
- How to read cookies and headers on the server with `cookies()` from `next/headers`
- How to **protect a page** by checking the session and using `redirect()`
- How **middleware** can guard routes before they even render
- When to reach for a library like **Auth.js (NextAuth)** or **Clerk**

## Authentication vs Authorization

These two words sound alike but mean different things:

- **Authentication** = *Who are you?* Proving identity (logging in with email and
  password).
- **Authorization** = *What are you allowed to do?* Checking permissions (is this user
  an admin? can they delete this post?).

Analogy: authentication is showing your ID at the airport. Authorization is whether
your ticket lets you into the first-class lounge. You authenticate first, then the app
authorizes specific actions.

## Sessions, JWTs, and Cookies

After you log in, the app needs to *remember* you on the next request (the web is
"forgetful" by default — each request is separate). There are two common ways to
remember:

### Cookies

A **cookie** is a small piece of data the server asks the browser to store and send
back on every request. It's like a coat-check ticket: you hand it over each time and
the server knows it's you. Cookies are how *both* approaches below actually travel
between browser and server.

### Sessions

With a **session**, the server keeps the real login info in its own storage (a
database or memory) and gives the browser only a random **session ID** in a cookie.
Each request, the server looks up that ID to find who you are.

- Like a hotel key card: the card itself is meaningless, but the front desk knows which
  room it opens.

### JWT (JSON Web Token)

A **JWT** is a self-contained token that holds the user info *inside it*, signed so it
can't be faked. The server doesn't need to look anything up — it just verifies the
signature.

- Like a tamper-proof wristband with your details printed on it: anyone can read it,
  nobody can forge it.

Neither is "better" — sessions are easy to revoke; JWTs avoid a lookup. Most libraries
handle the choice for you.

## Reading Cookies and Headers on the Server

In a Server Component or Server Action, you can read cookies with `cookies()` from
`next/headers`. In Next 15 it's `async`, so you `await` it.

```tsx
// app/dashboard/page.tsx
import { cookies } from "next/headers";

export default async function Dashboard() {
  const cookieStore = await cookies(); // read the request's cookies
  const session = cookieStore.get("session"); // grab one by name

  return <p>Session cookie: {session?.value ?? "none"}</p>;
}
```

You can read request headers the same way with `headers()`:

```tsx
// reading a header on the server
import { headers } from "next/headers";

const headerList = await headers();
const userAgent = headerList.get("user-agent"); // what browser is this?
```

## Protecting a Page

To make a page private, check for a valid session at the top of a Server Component. If
there isn't one, send the user to the login page with `redirect()`.

```tsx
// app/dashboard/page.tsx
import { cookies } from "next/headers";
import { redirect } from "next/navigation";

// pretend helper: turns a cookie value into a user, or null
async function getUser() {
  const cookieStore = await cookies();
  const token = cookieStore.get("session")?.value;
  if (!token) return null;
  return verifyToken(token); // your real check here
}

export default async function Dashboard() {
  const user = await getUser();

  // not logged in? bounce them to /login
  if (!user) {
    redirect("/login");
  }

  return <h1>Welcome back, {user.name}!</h1>;
}
```

Because this runs on the **server**, the protected content is never even sent to a
logged-out visitor. That's safer than hiding things with JavaScript in the browser.

## Middleware for Route Protection (Brief)

Checking the session in *every* page gets repetitive. **Middleware** is code that runs
*before* a request reaches your page — a single guard at the gate for many routes. You
put it in a `middleware.ts` file at the project root.

```tsx
// middleware.ts
import { NextRequest, NextResponse } from "next/server";

export function middleware(request: NextRequest) {
  // is there a session cookie?
  const session = request.cookies.get("session");

  // no session and trying to reach a protected area? redirect
  if (!session) {
    return NextResponse.redirect(new URL("/login", request.url));
  }

  return NextResponse.next(); // allow the request through
}

// only run this middleware on these paths
export const config = {
  matcher: ["/dashboard/:path*", "/account/:path*"],
};
```

The `matcher` says *which* URLs the middleware applies to. Keep middleware checks
lightweight (just "is there a cookie?") — do the deep verification in the page or a
library, since middleware runs on every matching request.

## Using a Library (High Level)

Rolling your own auth securely is *hard* — password hashing, token expiry, social
logins, CSRF protection, and more. For real apps, use a trusted library:

- **Auth.js (NextAuth)** — a popular, free, open-source option. It handles login with
  Google, GitHub, email, and more, and manages sessions/cookies for you. You configure
  *providers* and read the session in your components.
- **Clerk** — a hosted service with ready-made sign-in/sign-up UI components and user
  management. Less to build yourself; you drop in their components and wrap your app.

You don't need to learn these in depth now. Just know the pattern is the same: the
library logs the user in, stores a session (usually in a cookie), and gives you a way
to ask "who is the current user?" on the server — then you protect pages exactly like
we did above.

```tsx
// rough idea with a library helper (names vary by library)
import { auth } from "@/auth"; // provided by Auth.js setup

export default async function Page() {
  const session = await auth(); // who's logged in?
  if (!session) redirect("/login");
  return <p>Hi {session.user.name}</p>;
}
```

## Common Mistakes

- **Confusing authentication with authorization.** First *who are you*, then *what can
  you do*.
- **Hiding private content only in the browser.** A logged-out user could still grab
  it. Check on the **server** and `redirect()`.
- **Forgetting to `await cookies()` / `await headers()`** in Next 15.
- **Doing heavy verification in middleware.** Keep it light; middleware runs on every
  matching request.
- **Building your own auth from scratch for a real app.** It's risky — prefer a
  trusted library like Auth.js or Clerk.

## Exercises

1. In words, explain the difference between authentication and authorization with your
   own analogy.
2. Write a Server Component that reads a `session` cookie with `cookies()` and prints
   its value (or "no session").
3. Protect a `/dashboard` page: if there's no session cookie, `redirect()` to
   `/login`.
4. Add a `middleware.ts` that redirects unauthenticated users away from
   `/account/*` paths.
5. Read the home page of the Auth.js (NextAuth) or Clerk docs and write three
   sentences on how it would handle login for you.

## Key Takeaways

- **Authentication** asks *who are you*; **authorization** asks *what can you do*.
- **Cookies** carry login info; **sessions** store data on the server, while **JWTs**
  carry signed data in the token itself.
- Read login info on the server with `cookies()` and `headers()` from `next/headers`
  (await them in Next 15).
- Protect a page by checking the session in a Server Component and calling
  `redirect()` if it's missing.
- **Middleware** (`middleware.ts`) can guard many routes before they render.
- For real apps, use a library like **Auth.js (NextAuth)** or **Clerk** instead of
  rolling your own.

---

[← Previous: Forms and Data Mutations](04-forms-and-mutations.md) | [Back to Course Index](../README.md) | [Next: Working with a Database →](../05-data-and-state/01-databases-overview.md)
