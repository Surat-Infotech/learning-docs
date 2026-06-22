# API Testing Basics (Manual)

So far you've tested through the **user interface** — clicking buttons and reading
screens. But underneath every app, parts talk to each other through **APIs**. You
can test those directly, *without* a UI, using a friendly tool like **Postman**.
Don't panic — this is still manual, no-coding testing, and it makes you noticeably
more valuable. This is a gentle, conceptual introduction.

## What You'll Learn

- What an API is, in plain English
- Why testing the API (not just the UI) is useful
- The basic vocabulary: request, response, method, status code
- How a manual tester uses a tool like Postman

## What Is an API? (No Jargon)

An **API (Application Programming Interface)** is how two pieces of software talk to
each other. A great everyday analogy is a **restaurant**:

```text
   You (the app's UI)  ──order──▶  Waiter (the API)  ──▶  Kitchen (the server/database)
                       ◀─food────                    ◀──
```

You don't walk into the kitchen. You tell the **waiter** (the API) what you want;
the waiter carries your request to the kitchen and brings back the result. The API
is the messenger between the front-end you see and the back-end that does the work.

When you log in, the UI sends your email/password to an API, which checks them and
sends back "success" or "wrong password." The UI just *displays* what the API
returns.

## Why Test the API Directly?

- **Find bugs earlier.** APIs are often built *before* the UI. You can test the
  logic before any screen exists (shift left!).
- **Pinpoint the source.** If the UI shows wrong data, is the *screen* broken or the
  *API*? Testing the API tells you which side to blame.
- **Test things the UI can't easily reach.** Send invalid data the UI would block,
  and see how the back-end *really* behaves.
- **Speed.** API checks are faster than clicking through screens.

This is **grey box** testing (from
[Module 2](../02-types-of-testing/06-black-grey-white-box.md)) — you peek a little
below the UI surface.

## The Core Vocabulary

### Request and Response

You send a **request** to the API; it sends back a **response**. That's the whole
conversation.

### HTTP Methods (the "verb" — what you want to do)

| Method | Means | Restaurant analogy |
| --- | --- | --- |
| **GET** | Read/fetch data | "Show me the menu" |
| **POST** | Create new data | "Place a new order" |
| **PUT / PATCH** | Update existing data | "Change my order" |
| **DELETE** | Remove data | "Cancel my order" |

### Status Codes (the response's "headline")

Every response includes a 3-digit **status code** telling you what happened. Learn
the families:

| Range | Meaning | Common ones |
| --- | --- | --- |
| **2xx** | ✅ Success | 200 OK, 201 Created |
| **4xx** | ❌ *You* (the request) did something wrong | 400 Bad Request, 401 Unauthorized, 404 Not Found |
| **5xx** | 💥 The *server* broke | 500 Internal Server Error |

Memory aid: **4xx = your fault, 5xx = their (server's) fault.** As a tester you'll
check: did I get the status code I *expected*? Sending a bad password should return
401 — if it returns 200, that's a bug.

### Payload / Body

The actual data sent or received, usually in **JSON** format (a simple
human-readable text format of `"key": value` pairs):

```json
{
  "email": "bob@example.com",
  "password": "Correct123!"
}
```

You don't need to *write* code to read JSON — it's just labeled data.

## How a Manual Tester Uses Postman

**Postman** is a free, point-and-click tool for sending API requests — no
programming required. A typical check:

1. **Choose a method** (e.g., POST) and enter the API **URL**.
2. **Add the request body** (e.g., the login JSON above).
3. **Click Send.**
4. **Inspect the response:** the status code, the returned data, the response time.
5. **Compare to expected:** right status? right data? sensible speed?

Then do what you always do — go off the happy path:

- Send a **missing field** → expect a 400, not a crash.
- Send a **wrong password** → expect 401, and make sure it *doesn't* leak whether
  the email exists.
- Send a **huge value** or wrong data type → see how it copes.

It's the same tester mindset, pointed at the API instead of the screen.

## You Don't Need to Be a Programmer

API testing *sounds* technical, but at this level it's: fill in a form, click Send,
read the result, compare to what you expected. That's it. Learning Postman basics is
one of the highest-value things a manual tester can do to stand out — and it's very
beginner-friendly.

## Exercises

1. Explain what an API is using your own analogy (not the restaurant one).
2. Match the method to the intent: read data, create data, update data, delete data.
3. What do 200, 401, 404, and 500 each mean? Which are "your fault" vs "server's
   fault"?
4. You send a login request with a wrong password and get back **200 OK** with a
   success message. Is this a bug? Why?

Next: [Cross-Browser and Mobile Testing](04-cross-browser-and-mobile.md)
