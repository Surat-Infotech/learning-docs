# Promises

Think about ordering food at a busy counter. They do not make you stand and
wait. They give you a **buzzer**. The buzzer is a promise: "Your food is not
ready yet, but it will be. When it is, this buzzer will go off."

A **Promise** in JavaScript is just like that buzzer. It is a placeholder for a
value that is not ready yet but will arrive later. Promises are the modern,
cleaner replacement for callbacks. They let us escape the "callback hell" from
the previous lesson.

## What You'll Learn

- What a **Promise** is (a placeholder for a future value)
- The three states: **pending**, **fulfilled**, **rejected**
- How to **create** a Promise with `resolve` and `reject`
- How to **use** a Promise with `.then`, `.catch`, and `.finally`
- How to **chain** `.then` calls and pass values down the chain
- `Promise.all`, `Promise.race`, and `Promise.allSettled`

## What Is a Promise?

A **Promise** is an object that represents a value that will be ready later. The
word "promise" fits well: it is a guarantee that you will get *something* in the
future, either a result or an error.

Like the food buzzer, a Promise has a current state:

- It might still be **waiting** (food not ready).
- It might **succeed** (food ready, here it is).
- It might **fail** (sorry, we are out of that dish).

## The Three States

A Promise is always in one of three states:

1. **Pending** — still working, no result yet (the buzzer has not gone off).
2. **Fulfilled** — finished successfully, with a value (your food is ready).
3. **Rejected** — finished with an error (something went wrong).

Once a Promise is fulfilled or rejected, it is **settled**. A settled Promise
never changes again. The buzzer only goes off once.

## Creating a Promise

You create a Promise with `new Promise(...)`. You give it a function with two
parameters: `resolve` and `reject`.

- Call `resolve(value)` when the work **succeeds**. This fulfills the Promise.
- Call `reject(error)` when the work **fails**. This rejects the Promise.

```js
const promise = new Promise(function (resolve, reject) {
  const success = true;

  if (success) {
    resolve("It worked!"); // fulfilled with this value
  } else {
    reject(new Error("It failed.")); // rejected with this error
  }
});
```

By itself this does nothing visible. We need to **consume** the Promise to see
its result.

## Consuming a Promise: `.then`, `.catch`, `.finally`

To get the value out of a Promise, you attach handlers:

- `.then(fn)` runs when the Promise is **fulfilled**. It receives the value.
- `.catch(fn)` runs when the Promise is **rejected**. It receives the error.
- `.finally(fn)` runs at the end, **no matter what** (success or failure).

```js
const promise = new Promise(function (resolve, reject) {
  resolve("Hello from the future");
});

promise
  .then(function (value) {
    console.log("Success:", value);
  })
  .catch(function (error) {
    console.log("Error:", error.message);
  })
  .finally(function () {
    console.log("All done.");
  });

// => Success: Hello from the future
// => All done.
```

If the Promise had been rejected instead, the `.catch` would run instead of the
`.then`, and `.finally` would still run.

```js
const failing = new Promise(function (resolve, reject) {
  reject(new Error("Something broke"));
});

failing
  .then(function (value) {
    console.log("Success:", value); // skipped
  })
  .catch(function (error) {
    console.log("Error:", error.message);
  })
  .finally(function () {
    console.log("All done.");
  });

// => Error: Something broke
// => All done.
```

`.finally` is great for cleanup, like hiding a loading spinner whether the
request worked or not.

## A Fake `getUser` Example

Let's rebuild the `getUser` function from the callbacks lesson, but using a
Promise. We still fake the delay with `setTimeout`.

```js
function getUser(id) {
  // The function RETURNS a Promise
  return new Promise(function (resolve, reject) {
    console.log("Fetching user " + id + "...");

    setTimeout(function () {
      if (id <= 0) {
        reject(new Error("Invalid id: " + id)); // failure
      } else {
        const user = { id: id, name: "Vinay" };
        resolve(user); // success
      }
    }, 1000);
  });
}

getUser(1)
  .then(function (user) {
    console.log("Got user:", user.name);
  })
  .catch(function (error) {
    console.log("Failed:", error.message);
  });

// => Fetching user 1...
// => Got user: Vinay   (about 1 second later)
```

And the failure case:

```js
getUser(-5)
  .then(function (user) {
    console.log("Got user:", user.name);
  })
  .catch(function (error) {
    console.log("Failed:", error.message);
  });

// => Fetching user -5...
// => Failed: Invalid id: -5
```

Compare this to the error-first callback. The success path and the error path
are clearly separated. Much easier to read.

## Chaining `.then`

Here is where Promises shine. Each `.then` returns a **new** Promise. So you can
add another `.then` after it, and another. This makes a flat **chain** instead
of a nested pyramid.

Whatever you `return` inside a `.then` becomes the value for the **next**
`.then`.

```js
getUser(1)
  .then(function (user) {
    console.log("Step 1: got", user.name);
    return user.name.toUpperCase(); // pass this to the next .then
  })
  .then(function (upperName) {
    console.log("Step 2: name is", upperName);
    return upperName.length; // pass this to the next .then
  })
  .then(function (length) {
    console.log("Step 3: name has", length, "letters");
  })
  .catch(function (error) {
    console.log("Failed somewhere:", error.message);
  });

// => Fetching user 1...
// => Step 1: got Vinay
// => Step 2: name is VINAY
// => Step 3: name has 5 letters
```

Notice we only need **one** `.catch` at the bottom. It catches an error from
**any** step in the chain. This is a huge improvement over repeating error
checks at every level like we did with callbacks.

You can also return another Promise from a `.then`. The chain will wait for it
to settle before moving on. This is how you do several async steps in order,
flatly.

```js
getUser(1)
  .then(function (user) {
    console.log("Got user:", user.name);
    return getUser(2); // returning a Promise; chain waits for it
  })
  .then(function (secondUser) {
    console.log("Got second user:", secondUser.name);
  })
  .catch(function (error) {
    console.log("Failed:", error.message);
  });
```

## Running Many Promises Together

Sometimes you have several Promises and you want to handle them as a group.
JavaScript gives you helpers for this.

### `Promise.all` — Wait for All to Succeed

`Promise.all` takes an array of Promises. It waits for **all** of them to
fulfill, then gives you an array of all the results. If **any one** rejects, the
whole thing rejects right away.

```js
Promise.all([getUser(1), getUser(2), getUser(3)])
  .then(function (users) {
    // users is an array of all results, in order
    console.log("All users:", users.map((u) => u.name));
  })
  .catch(function (error) {
    console.log("At least one failed:", error.message);
  });

// => All users: [ 'Vinay', 'Vinay', 'Vinay' ]
```

Use `Promise.all` when you need **all** results and any failure means you cannot
continue.

### `Promise.race` — First One to Finish Wins

`Promise.race` settles as soon as the **first** Promise settles, whether it
succeeds or fails. The rest are ignored.

```js
const fast = new Promise((resolve) => setTimeout(() => resolve("fast"), 100));
const slow = new Promise((resolve) => setTimeout(() => resolve("slow"), 1000));

Promise.race([fast, slow]).then(function (winner) {
  console.log("Winner:", winner);
});

// => Winner: fast
```

A common real use: race a request against a timeout, so you can give up if it
takes too long.

### `Promise.allSettled` — Wait for All, Never Reject

`Promise.allSettled` waits for **every** Promise to finish, and tells you the
outcome of each one. It never rejects. You get an array of result objects, each
with a `status` of `"fulfilled"` or `"rejected"`.

```js
Promise.allSettled([getUser(1), getUser(-5), getUser(3)])
  .then(function (results) {
    results.forEach(function (result) {
      if (result.status === "fulfilled") {
        console.log("OK:", result.value.name);
      } else {
        console.log("FAILED:", result.reason.message);
      }
    });
  });

// => OK: Vinay
// => FAILED: Invalid id: -5
// => OK: Vinay
```

Use `allSettled` when you want every result even if some fail, like loading
several widgets where one broken widget should not hide the others.

## Common Mistakes

- **Forgetting to `return` inside a `.then`.** If you do not return a value, the
  next `.then` receives `undefined`.
- **Forgetting `.catch`.** An unhandled rejection is a silent bug. Always add a
  `.catch` (usually at the end of the chain).
- **Nesting `.then` calls inside each other.** That rebuilds callback hell.
  Instead, `return` the next Promise and chain flatly.
- **Thinking a settled Promise can change.** Once it resolves or rejects, it is
  fixed forever.
- **Using `Promise.all` when one failure should not stop everything.** Use
  `Promise.allSettled` instead.

## Exercises

1. Create a Promise that resolves with the string `"Pizza is ready"` after 2
   seconds (use `setTimeout`). Print the value with `.then`.
2. Create a Promise that **rejects** with `new Error("Out of stock")`. Handle it
   with `.catch` and print the error message.
3. Using the `getUser` Promise function from this lesson, chain three `.then`
   calls: get the user, return the name in uppercase, then print how many
   letters it has.
4. Call `Promise.all` with `getUser(1)` and `getUser(2)`. Print both names. Then
   change one id to `0` and see how `Promise.all` behaves versus
   `Promise.allSettled`.

## Key Takeaways

- A **Promise** is a placeholder for a value that arrives later.
- Its three states are **pending**, **fulfilled**, and **rejected**.
- Create one with `new Promise((resolve, reject) => { ... })`.
- Consume it with `.then` (success), `.catch` (error), and `.finally` (always).
- **Chain** `.then` calls and `return` values to pass them down the chain.
- One `.catch` at the end catches errors from the whole chain.
- `Promise.all` (all must succeed), `Promise.race` (first wins), and
  `Promise.allSettled` (all results, never rejects) handle groups of Promises.

---

[← Back to Course Index](../README.md) | [Next: async / await →](04-async-await.md)
