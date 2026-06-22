# Callbacks

Imagine you order a pizza by phone. You give the shop your phone number and say:
"Call me when it is ready." You do not stand at the shop and wait. You go home
and do other things. When the pizza is done, the shop **calls you back**.

That phone number you left is like a **callback** in JavaScript. It is a
function you hand to another function and say: "Run this later, when you are
ready." Callbacks are the oldest and simplest way to handle async code in
JavaScript.

## What You'll Learn

- What a **callback** is (a function passed to another function to run later)
- **Synchronous** callbacks (like in `forEach`)
- **Asynchronous** callbacks (like in `setTimeout`)
- How to write your own function that takes a callback
- The **error-first** callback style
- What **callback hell** is and why it is hard to read
- Why this pain leads us to **Promises**

## What Is a Callback?

A **callback** is just a function that you pass into another function as an
argument, so it can be called ("called back") later.

You already know functions are values. That means you can pass a function to
another function, just like passing a number or a string.

```js
function sayBye() {
  console.log("Goodbye!");
}

function doThenCall(callback) {
  console.log("Doing the main work...");
  callback(); // run the function we were given
}

doThenCall(sayBye);
// => Doing the main work...
// => Goodbye!
```

Notice we passed `sayBye` **without** the `()`. We hand over the function
itself, not its result. The other function decides **when** to call it.

You can also pass a function written right there (an **anonymous** function):

```js
doThenCall(function () {
  console.log("I was written inline!");
});
// => Doing the main work...
// => I was written inline!
```

## Synchronous Callbacks

A **synchronous** callback runs right away, during the function call, in order.
You have already used these. Array methods like `forEach`, `map`, and `filter`
take a callback and run it for each item.

```js
const fruits = ["apple", "banana", "cherry"];

// The arrow function is the callback. forEach runs it for each item.
fruits.forEach(function (fruit) {
  console.log("I like " + fruit);
});
// => I like apple
// => I like banana
// => I like cherry

console.log("Done"); // runs AFTER the loop, because forEach is synchronous
// => Done
```

Here the callback runs immediately, one item at a time. Nothing waits in the
background. This is a normal, in-order callback.

## Asynchronous Callbacks

An **asynchronous** callback runs **later**, after some time or after a slow job
finishes. `setTimeout` is the classic example. It takes a callback and a delay
in milliseconds (1000 ms = 1 second).

```js
console.log("Before timer");

setTimeout(function () {
  console.log("3 seconds passed!"); // runs LATER
}, 3000);

console.log("After timer");

// => Before timer
// => After timer
// => 3 seconds passed!   (appears about 3 seconds later)
```

See the order? `"After timer"` prints **before** the callback. JavaScript did
not wait. It set the timer, kept going, and ran the callback later when the
timer finished. This is the async behavior from the previous lesson in action.

## Writing a Function That Takes a Callback

Let's pretend we are fetching a user from a database. Real data takes time, so
we fake the delay with `setTimeout`. We give our function a callback to run when
the "data" is ready.

```js
function getUser(id, callback) {
  console.log("Fetching user " + id + "...");

  // Pretend this takes 1 second
  setTimeout(function () {
    const user = { id: id, name: "Vinay" };
    callback(user); // give the result back through the callback
  }, 1000);
}

getUser(1, function (user) {
  console.log("Got user:", user.name);
});

// => Fetching user 1...
// => Got user: Vinay   (about 1 second later)
```

Because the data is not ready right away, we cannot just `return` it. Instead we
pass a callback, and the function calls it **when the data arrives**. This is
the core idea of async programming.

## The Error-First Callback Style

Real async work can fail. The network might be down. The user might not exist.
We need a way to report errors through the callback too.

The common pattern in JavaScript is the **error-first callback**. The rule is
simple: the callback's **first** parameter is always the error. If there is no
error, that first value is `null`.

```js
function getUser(id, callback) {
  setTimeout(function () {
    if (id <= 0) {
      // Something went wrong: pass an error as the FIRST argument
      callback(new Error("Invalid id"), null);
      return;
    }

    // Success: error is null, data is second
    const user = { id: id, name: "Vinay" };
    callback(null, user);
  }, 1000);
}

getUser(1, function (error, user) {
  if (error) {
    console.log("Problem:", error.message);
    return; // stop here on error
  }
  console.log("Success:", user.name);
});
// => Success: Vinay

getUser(-5, function (error, user) {
  if (error) {
    console.log("Problem:", error.message);
    return;
  }
  console.log("Success:", user.name);
});
// => Problem: Invalid id
```

Always check the error first. If there is an error, handle it and stop. Only
then use the data.

## Callback Hell

Callbacks work, but they get ugly fast. What if you need to do several async
steps in order? For example: get a user, then get that user's posts, then get
the comments on the first post.

Each step needs the result of the step before it. So you nest callbacks inside
callbacks inside callbacks.

```js
getUser(1, function (error, user) {
  if (error) {
    console.log("Error:", error.message);
    return;
  }
  getPosts(user.id, function (error, posts) {
    if (error) {
      console.log("Error:", error.message);
      return;
    }
    getComments(posts[0].id, function (error, comments) {
      if (error) {
        console.log("Error:", error.message);
        return;
      }
      console.log("Comments:", comments);
      // ...imagine even more nesting here
    });
  });
});
```

Look at the shape. The code drifts further and further to the right, like a
staircase. People call this **callback hell** or the **pyramid of doom**.

Why is it bad?

- It is **hard to read.** Your eyes get lost in the nesting.
- It is **hard to follow the order** of what happens.
- **Error handling repeats** in every single level.
- It is **easy to make mistakes** and hard to change later.

This pain is exactly why **Promises** were created. Promises let us write async
steps in a flat, readable chain instead of a deep pyramid. That is the next
lesson.

## Common Mistakes

- **Calling the function instead of passing it.** Pass `myFn`, not `myFn()`. The
  `()` runs it immediately and passes the *result*, not the function.
- **Expecting an async result with `return`.** You cannot `return` a value that
  is not ready yet. Use the callback to deliver it.
- **Forgetting to check the error first.** In error-first callbacks, always
  handle `error` before touching the data.
- **Forgetting to `return` after handling an error.** If you do not stop, the
  code keeps running with bad or missing data.
- **Nesting too deeply.** If your callbacks form a deep pyramid, that is a sign
  to switch to Promises.

## Exercises

1. Write a function `repeat(times, callback)` that calls the given `callback`
   `times` times. Test it by printing `"Hi"` three times.
2. Write a function `delayedHello(name, callback)` that waits 1 second (use
   `setTimeout`) and then calls `callback` with the string
   `"Hello, <name>!"`. Print the result.
3. Take the `getUser` example with the error-first style. Call it once with a
   valid id and once with `0`, and make sure both the success and error
   branches print correctly.
4. Look at the callback hell example. In plain words, describe two reasons it is
   hard to maintain.

## Key Takeaways

- A **callback** is a function passed to another function to be run later.
- **Synchronous** callbacks run right away (e.g. `forEach`).
- **Asynchronous** callbacks run later (e.g. `setTimeout`).
- You cannot `return` an async result; deliver it through the callback instead.
- The **error-first** style puts the error as the first callback argument
  (`null` when there is no error).
- Deeply nested callbacks create **callback hell**: hard to read and maintain.
- **Promises** were invented to fix callback hell. They are next.

---

[← Back to Course Index](../README.md) | [Next: Promises →](03-promises.md)
