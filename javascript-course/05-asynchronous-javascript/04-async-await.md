# async / await

Promises made async code much nicer than callbacks. But chains of `.then` can
still feel a bit clunky. Wouldn't it be great if async code could read just like
normal, top-to-bottom code?

That is exactly what `async` and `await` give us. They are a friendlier way to
work with Promises. Under the hood it is **still Promises** — `async`/`await` is
just a nicer "skin" over them. If you understand Promises, you already
understand most of this.

## What You'll Learn

- That `async` functions **always return a Promise**
- That `await` **pauses** a function until a Promise settles
- How to rewrite a `.then` chain using `async`/`await`
- How to handle errors with **`try`/`catch`** and `await`
- The difference between awaiting things **one by one** vs **in parallel**
- The common mistake of **forgetting `await`**
- That `await` only works **inside `async`** (plus a note on top-level `await`)

## `async` Functions Always Return a Promise

Put the word `async` before a function to make it an **async function**. The key
rule: an async function **always returns a Promise**, even if you return a plain
value.

```js
async function greet() {
  return "Hello"; // looks like a string, but...
}

const result = greet();
console.log(result); // => Promise { 'Hello' }  (it is a Promise!)

// To get the actual value, use .then or await
greet().then(function (value) {
  console.log(value); // => Hello
});
```

So `return "Hello"` inside an async function is the same as
`resolve("Hello")` in a Promise. Good to know, but the real magic is `await`.

## `await` Pauses Until a Promise Settles

The `await` keyword goes **before** a Promise. It pauses the async function
until that Promise settles, then gives you the resolved value directly — no
`.then` needed.

It feels like the code is waiting, but remember from the event-loop lesson: the
single worker is **not** frozen. Other code can still run while this function is
paused. `await` only pauses *this one function*.

```js
function wait(ms, value) {
  return new Promise(function (resolve) {
    setTimeout(() => resolve(value), ms);
  });
}

async function run() {
  console.log("Start");
  const result = await wait(1000, "Done!"); // pause here ~1s, then continue
  console.log(result);
}

run();
// => Start
// => Done!   (about 1 second later)
```

The value on the right of `await` becomes the value on the left. Clean and
readable, like normal code.

## Rewriting a `.then` Chain

Let's bring back our fake `getUser` function (it returns a Promise). Here is the
`.then` version from the Promises lesson:

```js
function getUser(id) {
  return new Promise(function (resolve, reject) {
    setTimeout(function () {
      if (id <= 0) {
        reject(new Error("Invalid id: " + id));
      } else {
        resolve({ id: id, name: "Vinay" });
      }
    }, 1000);
  });
}

// The .then version
getUser(1)
  .then(function (user) {
    console.log("Got user:", user.name);
    return getUser(2);
  })
  .then(function (secondUser) {
    console.log("Got second user:", secondUser.name);
  });
```

Now the same logic with `async`/`await`. Notice how flat and readable it is:

```js
async function showUsers() {
  const user = await getUser(1);
  console.log("Got user:", user.name);

  const secondUser = await getUser(2);
  console.log("Got second user:", secondUser.name);
}

showUsers();
// => Got user: Vinay
// => Got second user: Vinay
```

It reads top to bottom, like a normal story. This is why people love
`async`/`await`.

## Error Handling with `try`/`catch`

With Promises you used `.catch`. With `await` you use a `try`/`catch` block,
which you may know from normal error handling. Code that might fail goes in
`try`. If a Promise rejects, control jumps to `catch`.

```js
async function showUser(id) {
  try {
    const user = await getUser(id); // if this rejects, jump to catch
    console.log("Got user:", user.name);
  } catch (error) {
    console.log("Failed:", error.message);
  } finally {
    console.log("Done trying.");
  }
}

showUser(1);
// => Got user: Vinay
// => Done trying.

showUser(-5);
// => Failed: Invalid id: -5
// => Done trying.
```

`try`/`catch` wraps the risky `await`. The `finally` block runs either way,
perfect for cleanup like hiding a loading spinner. We cover errors in more depth
in the next lesson.

## Sequential vs Parallel

This part is important for speed. There are two ways to await multiple things,
and they are very different.

### Sequential (One After Another)

If you `await` each Promise on its own line, they run **one at a time**. The
second one does not start until the first finishes. If each takes 1 second,
three of them take **3 seconds total**.

```js
async function sequential() {
  console.time("sequential");

  const a = await getUser(1); // wait 1s
  const b = await getUser(2); // then wait 1s more
  const c = await getUser(3); // then wait 1s more

  console.log(a.name, b.name, c.name);
  console.timeEnd("sequential"); // => ~3000 ms
}

sequential();
```

Use this only when each step **needs the result** of the step before it.

### Parallel (All at Once with `Promise.all`)

If the tasks do **not** depend on each other, start them all at once and `await`
them together with `Promise.all`. They run in parallel, so three 1-second tasks
take about **1 second total**.

```js
async function parallel() {
  console.time("parallel");

  // Start all three NOW (no await yet), then await them together
  const [a, b, c] = await Promise.all([
    getUser(1),
    getUser(2),
    getUser(3),
  ]);

  console.log(a.name, b.name, c.name);
  console.timeEnd("parallel"); // => ~1000 ms
}

parallel();
```

This is much faster. When tasks are independent, prefer parallel. Reaching for
`await` on every line out of habit can make your app needlessly slow.

## The Common Mistake: Forgetting `await`

If you forget `await`, you get the **Promise object** instead of the value. Your
code does not wait, and things go wrong quietly.

```js
async function broken() {
  const user = getUser(1); // OOPS: forgot await
  console.log(user.name);  // => undefined  (user is a Promise, not the data)
}

broken();
```

A Promise has no `.name` property, so you get `undefined` — and no error to warn
you. Always `await` a Promise when you need its value.

```js
async function fixed() {
  const user = await getUser(1); // correct
  console.log(user.name);        // => Vinay
}
```

## You Can Only `await` Inside `async`

`await` is only allowed inside an `async` function. Using it in a normal
function is a syntax error.

```js
function normal() {
  const user = await getUser(1); // ERROR: await outside async function
}
```

Fix it by marking the function `async`:

```js
async function correct() {
  const user = await getUser(1); // OK
  console.log(user.name);
}
```

**A note on top-level `await`:** in modern JavaScript modules (and in the
browser console and Node.js REPL), you *can* use `await` at the top level,
outside any function. But inside a regular script or a normal function, you
still need to wrap it in an `async` function. When in doubt, put your `await`
code inside an `async` function — that always works.

## Common Mistakes

- **Forgetting `await`.** You get a Promise instead of the value, often showing
  up as `undefined` with no error.
- **Using `await` outside an `async` function.** That is a syntax error; mark
  the function `async`.
- **Awaiting independent tasks one by one.** This is slow. Use `Promise.all` to
  run them in parallel.
- **Forgetting `try`/`catch`.** A rejected Promise you `await` throws an error.
  Without `try`/`catch` (or a `.catch`), it becomes an unhandled error.
- **Thinking `await` freezes the whole program.** It only pauses *that* async
  function; other code keeps running.

## Exercises

1. Write an `async` function `delayedHello(name)` that uses a `wait` helper to
   pause 1 second, then prints `"Hello, <name>!"`. Call it.
2. Rewrite this `.then` chain using `async`/`await`:
   ```js
   getUser(1).then((u) => console.log(u.name));
   ```
3. Write an async function that calls `getUser(0)` inside a `try`/`catch` and
   prints the error message in the `catch` block.
4. Write two async functions that each fetch `getUser(1)`, `getUser(2)`, and
   `getUser(3)`: one **sequentially** and one in **parallel** with
   `Promise.all`. Use `console.time` to compare how long each takes.

## Key Takeaways

- An `async` function **always returns a Promise**.
- `await` pauses the async function until a Promise settles, giving you its
  value directly.
- `async`/`await` is a cleaner way to write Promise code — it is still Promises
  underneath.
- Use `try`/`catch`/`finally` to handle errors with `await`.
- Awaiting one by one is **sequential** (slower); `Promise.all` runs tasks in
  **parallel** (faster) when they are independent.
- Forgetting `await` is a common bug: you get a Promise, not the value.
- `await` works only inside `async` functions (top-level `await` works in
  modules).

---

[← Back to Course Index](../README.md) | [Next: Fetch and JSON →](05-fetch-and-json.md)
