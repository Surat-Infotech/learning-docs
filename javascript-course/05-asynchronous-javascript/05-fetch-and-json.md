# Fetch and JSON

Now we use everything you have learned to do something real and exciting: get
live data from the internet. Want a list of users, the weather, or movie info?
That data lives on a server somewhere. We **fetch** it.

This lesson ties async together. We will use `async`/`await`, talk to a real
public API, and turn the response into data we can use. By the end you will have
a complete, working example.

## What You'll Learn

- What an **API** is, in simple words
- What **JSON** is and what it looks like
- `JSON.parse` and `JSON.stringify`
- The `fetch` function with `async`/`await`
- `response.ok` and `response.json()`
- Handling errors (**network** errors vs **HTTP** errors)
- A complete example using a free public API (no key needed)
- The idea of a **loading state**

## What Is an API?

**API** stands for Application Programming Interface. That sounds heavy, so here
is the plain version: an API is a **waiter** between you and a kitchen.

You (the app) do not walk into the kitchen (the server's database). You tell the
waiter what you want. The waiter goes to the kitchen, gets it, and brings it
back to you. The API is the waiter — a middle layer that lets you ask for data
in a clear, agreed way.

When we "call an API", we send a request to a web address (a **URL**) and we get
back some data. Most of the time, that data comes back as **JSON**.

## What Is JSON?

**JSON** stands for JavaScript Object Notation. It is a simple text format for
sending data. It looks almost exactly like JavaScript objects and arrays, which
makes it easy to read.

Here is what JSON looks like:

```json
{
  "name": "Vinay",
  "age": 25,
  "isStudent": true,
  "hobbies": ["coding", "reading"],
  "address": {
    "city": "Delhi"
  }
}
```

A few rules that make JSON different from a JavaScript object:

- Keys must be in **double quotes** (`"name"`, not `name`).
- Strings use **double quotes** only.
- Values can be: string, number, `true`/`false`, `null`, array, or object.
- **No** functions, and **no** comments.

The important thing to remember: **JSON is text**, not a real JavaScript object.
Before you can use it, you usually have to convert it.

## `JSON.parse` and `JSON.stringify`

JavaScript gives you two tools to convert between JSON text and real objects.

- `JSON.parse(text)` turns a JSON **string** into a real JavaScript object.
- `JSON.stringify(object)` turns a JavaScript object into a JSON **string**.

```js
// A JSON string (notice the quotes around the whole thing — it is text)
const jsonText = '{"name": "Vinay", "age": 25}';

// Parse: text -> real object
const person = JSON.parse(jsonText);
console.log(person.name); // => Vinay
console.log(person.age);  // => 25

// Stringify: real object -> text
const obj = { name: "Asha", age: 30 };
const backToText = JSON.stringify(obj);
console.log(backToText); // => {"name":"Asha","age":30}
```

Easy way to remember: **parse** = "make it usable" (text to object).
**stringify** = "make it sendable" (object to text). We often `stringify` data
to send it, and `parse` data we receive.

## The `fetch` Function

`fetch` is a built-in function that requests data from a URL over the internet.
Because the internet takes time, `fetch` is **asynchronous**: it returns a
**Promise**. That means we use `await` (inside an `async` function) to wait for
the result.

```js
async function loadData() {
  const response = await fetch("https://api.github.com/users/octocat");
  console.log(response); // a Response object (not the data yet!)
}

loadData();
```

Important: `fetch` gives you a **Response** object first, not the actual data.
The Response is like the sealed envelope the waiter brings. You still have to
open it.

## `response.ok` and `response.json()`

Two things you do with the Response:

- `response.ok` is `true` if the request succeeded (status code 200–299), and
  `false` for errors like 404 (not found) or 500 (server error).
- `response.json()` reads the body and turns the JSON into a real object. It is
  **also async**, so it returns a Promise — you `await` it too.

```js
async function loadUser() {
  const response = await fetch("https://api.github.com/users/octocat");

  // Check if the request was successful
  if (!response.ok) {
    console.log("Request failed with status:", response.status);
    return;
  }

  // Read and parse the JSON body (await again!)
  const user = await response.json();
  console.log("Name:", user.name);
  console.log("Public repos:", user.public_repos);
}

loadUser();
// => Name: The Octocat
// => Public repos: 8   (numbers may differ)
```

Two `await`s: one for the response, one for reading the JSON. People often
forget the second one.

## Handling Errors: Network vs HTTP

There are **two different kinds** of failure, and they are handled differently.
This trips up many beginners.

1. **Network errors** — the request could not even be sent or completed (no
   internet, bad URL, server unreachable). In this case `fetch` **rejects**, so
   the error is caught by `try`/`catch`.
2. **HTTP errors** — the request reached the server, but the server replied with
   an error status like 404 or 500. Surprisingly, `fetch` does **not** reject
   for these! It treats them as a "successful" response with `ok === false`. You
   must check `response.ok` yourself.

So to be safe, handle **both**:

```js
async function loadUserSafely(username) {
  try {
    const response = await fetch("https://api.github.com/users/" + username);

    // HTTP error (e.g. 404 user not found) — fetch does NOT throw, so we do
    if (!response.ok) {
      throw new Error("HTTP error! Status: " + response.status);
    }

    const user = await response.json();
    console.log("Found:", user.name);
  } catch (error) {
    // Catches BOTH network errors AND the error we threw above
    console.log("Could not load user:", error.message);
  }
}

loadUserSafely("octocat");           // => Found: The Octocat
loadUserSafely("no-such-user-xyz12"); // => Could not load user: HTTP error! Status: 404
```

Remember: `try`/`catch` alone is **not enough** with `fetch`. Always check
`response.ok` too, and `throw` on a bad status so your `catch` handles it.

## A Complete Example

Let's put it all together. We will fetch a GitHub user (a free public API, no
key needed) and log a clean summary. This uses async, fetch, JSON, error
handling, and a loading state.

```js
async function getGitHubUser(username) {
  console.log("Loading..."); // loading state ON

  try {
    const response = await fetch("https://api.github.com/users/" + username);

    if (!response.ok) {
      throw new Error("User not found (status " + response.status + ")");
    }

    const user = await response.json();

    // Use the data
    console.log("Username:", user.login);
    console.log("Name:", user.name);
    console.log("Bio:", user.bio);
    console.log("Followers:", user.followers);
    console.log("Public repos:", user.public_repos);
  } catch (error) {
    console.log("Something went wrong:", error.message);
  } finally {
    console.log("Done loading."); // loading state OFF (always runs)
  }
}

getGitHubUser("octocat");
// => Loading...
// => Username: octocat
// => Name: The Octocat
// => Bio: null
// => Followers: 9000   (numbers may differ)
// => Public repos: 8
// => Done loading.
```

Try changing `"octocat"` to your own GitHub username and run it.

## The Idea of a Loading State

Notice the `"Loading..."` and `"Done loading."` messages. Fetching takes time.
During that time, your user should **know something is happening**, or they will
think the app is broken.

In a real web page, instead of `console.log`, you would show a spinner or a
"Loading..." message, then hide it when the data arrives. The pattern is always
the same three steps:

```js
async function loadWithState() {
  // 1. Turn loading ON (show spinner)
  let isLoading = true;
  console.log("isLoading:", isLoading);

  try {
    const response = await fetch("https://api.github.com/users/octocat");
    const user = await response.json();
    console.log("Show the data:", user.name);
  } catch (error) {
    console.log("Show an error message:", error.message);
  } finally {
    // 3. Turn loading OFF (hide spinner) — runs on success AND failure
    isLoading = false;
    console.log("isLoading:", isLoading);
  }
}

loadWithState();
```

The `finally` block is the perfect place to turn loading off, because it runs no
matter what happens. You will use this exact pattern in real apps and frameworks
like React.

## Common Mistakes

- **Forgetting the second `await`.** `fetch` is async *and* `response.json()` is
  async. You need `await` for both.
- **Thinking `fetch` throws on a 404.** It does not. Always check
  `response.ok` and `throw` yourself for bad statuses.
- **Confusing JSON text with a real object.** A JSON string is text. Use
  `JSON.parse` (or `response.json()`) to get a usable object.
- **Calling `fetch` outside an `async` function without handling the Promise.**
  Either `await` it inside `async`, or use `.then`.
- **Not showing a loading state.** The user will think your app froze. Show a
  spinner while waiting.

## Exercises

1. Fetch `https://api.github.com/users/octocat` and log only the user's `name`
   and `followers`. Use `async`/`await`.
2. Wrap your fetch in a `try`/`catch`. Then change the username to a fake one and
   confirm your error message prints. Add a check on `response.ok`.
3. Take the object `{ title: "Learn JS", done: false }`, turn it into a JSON
   string with `JSON.stringify`, then turn it back into an object with
   `JSON.parse`. Log both.
4. Write a `getUserRepos(username)` function that fetches
   `https://api.github.com/users/<username>/repos` and logs the **name** of each
   repo. (Hint: the response is an array, so use a loop or `forEach`.)

## Key Takeaways

- An **API** is like a waiter: it brings you data from a server.
- **JSON** is a text format for data. It looks like JS objects but is text.
- `JSON.parse` turns JSON text into an object; `JSON.stringify` does the reverse.
- `fetch` requests data and returns a **Promise**; `await` it.
- `fetch` gives a **Response** first; call `await response.json()` to get data.
- Check `response.ok` for **HTTP errors** (404, 500); `fetch` does not throw on
  those. `try`/`catch` handles **network errors**.
- Show a **loading state** while waiting, and clear it in `finally`.

---

[← Back to Course Index](../README.md) | [Next: Error Handling →](06-error-handling.md)
