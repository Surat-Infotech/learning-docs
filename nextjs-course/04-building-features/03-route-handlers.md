# Route Handlers (API Routes)

So far your pages have shown HTML. But sometimes you need a different kind of
endpoint — one that returns *raw data* (usually JSON) instead of a web page. That's a
**Route Handler**. Think of a page as a restaurant dining room (you sit and eat the
plated meal), and a Route Handler as the takeout window (you just grab the food in a
box and go). Both are served by the same kitchen. 

## What You'll Learn

- What a **Route Handler** is and where it lives (`app/api/.../route.ts`)
- How to build a JSON **API endpoint** with `GET` and `POST`
- How to read the **request**: query parameters and the body
- How to return responses with `Response` and `NextResponse.json`
- How to make **dynamic** API routes with `[id]`
- When to use a Route Handler **vs** a Server Action

## What Is a Route Handler?

A **Route Handler** is a file named `route.ts` (or `route.js`) that lives inside the
`app/` folder. Instead of exporting a React component, it exports functions named
after HTTP methods: `GET`, `POST`, `PUT`, `PATCH`, `DELETE`.

> An **HTTP method** is the *kind* of request: `GET` to read data, `POST` to create,
> `PUT`/`PATCH` to update, `DELETE` to remove.

The folder path becomes the URL, just like pages — but conventionally we put APIs
under `app/api/`:

```text
app/
  api/
    hello/
      route.ts   -> responds at /api/hello
```

Here's the simplest possible one:

```tsx
// app/api/hello/route.ts

// runs on the server when someone visits /api/hello
export function GET() {
  // return plain JSON
  return Response.json({ message: "Hello from the server!" });
}
```

Visit `/api/hello` in your browser and you'll see the JSON. No HTML, just data.

> Note: a folder can have *either* a `page.tsx` *or* a `route.ts`, not both — they'd
> fight over the same URL.

## Returning Responses

You have two easy ways to send data back.

### Plain `Response`

`Response.json(...)` is a built-in helper that returns JSON with the right headers:

```tsx
// app/api/status/route.ts
export function GET() {
  return Response.json({ status: "ok" });
}
```

### `NextResponse.json`

Next.js adds `NextResponse`, which does the same thing but with extra superpowers
(setting cookies, custom status codes, etc.):

```tsx
// app/api/status/route.ts
import { NextResponse } from "next/server";

export function GET() {
  // the second argument lets you set the status code
  return NextResponse.json({ status: "ok" }, { status: 200 });
}
```

Both work. Use `NextResponse` when you need those extras; `Response.json` is fine for
simple cases.

## Reading the Request

Your handler receives a `request` object describing the incoming call.

### Query parameters

**Query parameters** are the bits after a `?` in a URL, like
`/api/search?q=cats&page=2`. Read them from `request.nextUrl.searchParams`:

```tsx
// app/api/search/route.ts
import { NextRequest, NextResponse } from "next/server";

export function GET(request: NextRequest) {
  // get the value of ?q=...
  const query = request.nextUrl.searchParams.get("q");
  return NextResponse.json({ youSearchedFor: query });
}
```

Visiting `/api/search?q=cats` returns `{ "youSearchedFor": "cats" }`.

### The request body

For `POST` requests, the data usually comes in the **body**. If it's JSON, read it
with `await request.json()`:

```tsx
// app/api/echo/route.ts
import { NextRequest, NextResponse } from "next/server";

export async function POST(request: NextRequest) {
  // read and parse the JSON the client sent
  const data = await request.json();
  return NextResponse.json({ youSent: data });
}
```

## Dynamic API Routes

Just like pages, API routes can have a **dynamic segment** using square brackets. This
lets one handler serve many ids — `/api/users/1`, `/api/users/2`, and so on.

```text
app/
  api/
    users/
      [id]/
        route.ts   -> /api/users/anything
```

The dynamic part arrives in the second argument, `context`, under `params`:

```tsx
// app/api/users/[id]/route.ts
import { NextRequest, NextResponse } from "next/server";

export async function GET(
  request: NextRequest,
  { params }: { params: Promise<{ id: string }> }, // params is a Promise in Next 15
) {
  const { id } = await params; // unwrap it
  return NextResponse.json({ userId: id });
}
```

Visiting `/api/users/42` returns `{ "userId": "42" }`.

## Worked Example: A Tiny Notes API

Let's build a small API that can list notes and add a new one. We'll use an in-memory
array to keep it simple (a real app would use a database).

```tsx
// app/api/notes/route.ts
import { NextRequest, NextResponse } from "next/server";

// pretend database (resets when the server restarts)
let notes = [{ id: 1, text: "Buy milk" }];

// GET /api/notes -> list all notes
export function GET() {
  return NextResponse.json(notes);
}

// POST /api/notes -> add a new note
export async function POST(request: NextRequest) {
  const body = await request.json(); // expect { text: "..." }

  // simple validation: make sure text exists
  if (!body.text) {
    return NextResponse.json(
      { error: "text is required" },
      { status: 400 }, // 400 = bad request
    );
  }

  const newNote = { id: Date.now(), text: body.text };
  notes.push(newNote);

  return NextResponse.json(newNote, { status: 201 }); // 201 = created
}
```

You can test it from the terminal:

```bash
# list the notes
curl http://localhost:3000/api/notes

# add a note
curl -X POST http://localhost:3000/api/notes \
  -H "Content-Type: application/json" \
  -d '{"text":"Walk the dog"}'
```

```text
{"id":1718000000000,"text":"Walk the dog"}
```

## Route Handler vs Server Action

Both run on the server and can change data, so when do you use which? Here's the rule
of thumb:

| Use a **Route Handler** when...        | Use a **Server Action** when...           |
|----------------------------------------|-------------------------------------------|
| You need a **public API** others call  | You're mutating data from *your own* form |
| A **webhook** must hit a fixed URL     | The action lives inside your React app    |
| A **mobile app** or script needs data  | You want progressive-enhancement forms    |
| You call a **third-party** service     | You want to avoid hand-wiring fetch calls |

In short: **Route Handlers** are doorways for the *outside world* (public APIs,
webhooks, integrations). **Server Actions** are the internal plumbing for *your own*
forms and buttons. If only your own app talks to it, a Server Action is usually
simpler. If anything external needs a stable URL, build a Route Handler.

## Common Mistakes

- **Putting `page.tsx` and `route.ts` in the same folder.** They both claim the URL —
  keep them separate (that's why APIs live under `app/api/`).
- **Forgetting `await request.json()`.** Reading the body is asynchronous; you must
  `await` it.
- **Forgetting to `await params`** on Next 15 dynamic routes.
- **Returning a plain object.** You must return a `Response` (use `Response.json` or
  `NextResponse.json`), not just an object.
- **Reaching for a Route Handler when a Server Action fits.** For internal form
  mutations, the Server Action is simpler.

## Exercises

1. Create `/api/hello` that returns `{ message: "Hi" }`. Visit it in your browser.
2. Add `/api/greet` that reads a `?name=` query param and returns
   `{ greeting: "Hello, <name>" }`.
3. Build a `POST /api/echo` that returns whatever JSON body it received.
4. Make a dynamic `/api/products/[id]` route that returns the id it was given.
5. Extend the Notes API with a `DELETE` method that removes a note by id (passed as a
   query param or dynamic segment).

## Key Takeaways

- A **Route Handler** is a `route.ts` file exporting `GET`/`POST`/etc.; it returns
  data, not HTML.
- Read query params from `request.nextUrl.searchParams`, and the body with
  `await request.json()`.
- Return data with `Response.json(...)` or `NextResponse.json(...)`.
- Dynamic API routes use `[id]` and read it from `await params`.
- Use **Route Handlers** for public APIs, webhooks, and third-party calls; use
  **Server Actions** for your own internal mutations.

---

[← Previous: Images, Fonts, and Metadata](02-images-fonts-metadata.md) | [Back to Course Index](../README.md) | [Next: Forms and Data Mutations →](04-forms-and-mutations.md)
