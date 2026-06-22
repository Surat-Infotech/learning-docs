# Error Handling

Things go wrong. The internet drops. A user types text where a number should be.
A file is missing. Good programs do not crash when this happens — they handle
the problem and keep going, or fail in a calm, friendly way.

This lesson teaches you how to deal with errors in JavaScript, both in normal
code and in async code. Handling errors well is one of the biggest differences
between beginner code and professional code.

## What You'll Learn

- What **errors** are and why they happen
- The **Error object** (`name` and `message`)
- How to **`throw`** an error
- `try` / `catch` / `finally`
- Catching specific situations
- **Re-throwing** an error
- Errors in **async** code (`try`/`catch` with `await`, and `.catch` with
  Promises)
- Writing **custom error messages**
- **Graceful failure**: showing the user a friendly message

## What Are Errors?

An **error** is JavaScript's way of saying "I cannot do that". When something
goes wrong, JavaScript **throws** an error. If nothing catches it, the program
**crashes** (stops) and prints a red error message.

```js
const num = 10;
num.toUpperCase(); // numbers have no toUpperCase
// => TypeError: num.toUpperCase is not a function
```

Errors are not bad. They are helpful. They tell you exactly what went wrong and
where. The goal is not to avoid all errors, but to **handle** the ones you
expect (like a failed network request).

## The Error Object

When an error is thrown, it is an **Error object**. It has two useful
properties:

- `name` — the type of error (e.g. `"TypeError"`, `"Error"`).
- `message` — a human-readable description of what went wrong.

You can create your own Error object with `new Error("some message")`:

```js
const myError = new Error("Something went wrong");

console.log(myError.name);    // => Error
console.log(myError.message); // => Something went wrong
```

There are several built-in error types, like `TypeError` (wrong type) and
`RangeError` (number out of range). For now, the plain `Error` is all you need
to create your own.

## Throwing Errors with `throw`

You can throw your own errors with the `throw` keyword. Do this when your code
hits a situation it cannot handle. Throwing stops the current function
immediately.

```js
function divide(a, b) {
  if (b === 0) {
    throw new Error("Cannot divide by zero"); // stop and report
  }
  return a / b;
}

console.log(divide(10, 2)); // => 5
console.log(divide(10, 0)); // => throws: Cannot divide by zero
```

**Tip:** always throw an `Error` object (`throw new Error("...")`), not a plain
string. Error objects carry a helpful `message` and a stack trace.

## `try` / `catch` / `finally`

To stop a thrown error from crashing your program, wrap risky code in a
**`try`** block. If an error is thrown, control jumps to the **`catch`** block,
which receives the error. The program keeps running.

```js
try {
  console.log("Before");
  throw new Error("Boom!");
  console.log("After"); // skipped — throw jumps to catch
} catch (error) {
  console.log("Caught:", error.message);
}

console.log("Program keeps going");

// => Before
// => Caught: Boom!
// => Program keeps going
```

You can add a **`finally`** block. It runs at the end **no matter what** —
whether or not there was an error. It is perfect for cleanup.

```js
function readData() {
  try {
    console.log("Opening connection");
    throw new Error("Connection lost");
  } catch (error) {
    console.log("Handling:", error.message);
  } finally {
    console.log("Closing connection"); // always runs
  }
}

readData();
// => Opening connection
// => Handling: Connection lost
// => Closing connection
```

## Catching Specific Situations

Sometimes you want to react differently depending on the error. You can inspect
the error inside `catch` and decide what to do.

```js
function parseJson(text) {
  try {
    return JSON.parse(text);
  } catch (error) {
    if (error.name === "SyntaxError") {
      console.log("That is not valid JSON.");
    } else {
      console.log("Some other error:", error.message);
    }
    return null; // return a safe fallback
  }
}

parseJson('{"ok": true}'); // works fine
parseJson("not json!!!");  // => That is not valid JSON.
```

Returning a safe fallback value (like `null` or an empty array) lets the rest of
your program continue sensibly instead of crashing.

## Re-throwing an Error

Sometimes you catch an error, do a little work (like logging it), but you cannot
truly handle it here. In that case you can **re-throw** it so code higher up can
deal with it.

```js
function loadConfig() {
  try {
    throw new Error("Config file missing");
  } catch (error) {
    console.log("Logging the problem:", error.message);
    throw error; // pass it up — we cannot fix it here
  }
}

try {
  loadConfig();
} catch (error) {
  console.log("Top level handled it:", error.message);
}

// => Logging the problem: Config file missing
// => Top level handled it: Config file missing
```

Re-throwing is useful when a lower function knows *something happened* but only a
higher function knows *what to do about it*.

## Errors in Async Code

Async errors need a little care. There are two styles, matching the two ways of
writing async code.

### With `async` / `await`: use `try` / `catch`

When you `await` a Promise that rejects, it throws an error right at the `await`
line. So you wrap it in `try`/`catch`, exactly like normal code.

```js
function getUser(id) {
  return new Promise(function (resolve, reject) {
    setTimeout(function () {
      if (id <= 0) {
        reject(new Error("Invalid id: " + id));
      } else {
        resolve({ id: id, name: "Vinay" });
      }
    }, 500);
  });
}

async function showUser(id) {
  try {
    const user = await getUser(id); // a rejection throws here
    console.log("Got:", user.name);
  } catch (error) {
    console.log("Async error:", error.message);
  }
}

showUser(1);  // => Got: Vinay
showUser(-1); // => Async error: Invalid id: -1
```

### With Promises: use `.catch`

If you use `.then` instead of `await`, attach a `.catch` to handle rejections.

```js
getUser(-1)
  .then(function (user) {
    console.log("Got:", user.name);
  })
  .catch(function (error) {
    console.log("Promise error:", error.message);
  });

// => Promise error: Invalid id: -1
```

Both styles do the same job. Use `try`/`catch` with `await`, and `.catch` with
`.then` chains. **Never leave a Promise without error handling** — an unhandled
rejection is a hidden bug.

## Custom Error Messages

Clear error messages save hours of debugging. Make your messages say **what**
went wrong and, if helpful, **why** or **what value** caused it.

```js
function setAge(age) {
  if (typeof age !== "number") {
    throw new Error("Age must be a number, but got: " + typeof age);
  }
  if (age < 0) {
    throw new Error("Age cannot be negative, but got: " + age);
  }
  return age;
}

try {
  setAge("twenty");
} catch (error) {
  console.log(error.message); // => Age must be a number, but got: string
}

try {
  setAge(-5);
} catch (error) {
  console.log(error.message); // => Age cannot be negative, but got: -5
}
```

A vague message like `"Error"` helps no one. A specific message like
`"Age cannot be negative, but got: -5"` tells you exactly what happened.

## Graceful Failure

**Graceful failure** means: when something goes wrong, do not crash and do not
show the user a scary technical error. Show a calm, friendly message and keep
the app usable.

Here is a realistic example. We try to fetch data; if anything fails, we log the
real error for ourselves (developers) but show the user a friendly message.

```js
async function loadProfile(username) {
  try {
    const response = await fetch("https://api.github.com/users/" + username);

    if (!response.ok) {
      throw new Error("HTTP status " + response.status);
    }

    const user = await response.json();
    return "Welcome, " + (user.name || user.login) + "!";
  } catch (error) {
    // For us (developers): the real, detailed error
    console.log("DEV LOG:", error.message);

    // For the user: something friendly and calm
    return "Sorry, we could not load this profile. Please try again later.";
  }
}

loadProfile("octocat").then((message) => console.log(message));
// => Welcome, The Octocat!

loadProfile("no-such-user-xyz12").then((message) => console.log(message));
// => DEV LOG: HTTP status 404
// => Sorry, we could not load this profile. Please try again later.
```

This is what good apps do: the developer gets the technical details, the user
gets a friendly message, and nothing crashes.

## Common Mistakes

- **Throwing a plain string.** Use `throw new Error("...")` so you get a
  `message` and a stack trace.
- **Empty `catch` blocks.** Swallowing an error silently hides bugs. At least
  log it.
- **Forgetting async errors.** A rejected Promise needs `try`/`catch` (with
  `await`) or `.catch` (with `.then`).
- **Showing raw errors to users.** Log technical details for yourself; show
  users a calm, friendly message.
- **Vague messages.** "Error" tells you nothing. Say what went wrong and which
  value caused it.

## Exercises

1. Write a function `getFirst(arr)` that returns the first item of an array, but
   `throw`s an `Error` with a clear message if the array is empty. Wrap a call in
   `try`/`catch`.
2. Write a function `safeDivide(a, b)` that throws if `b` is `0`. Call it inside
   `try`/`catch` and print the error message when dividing by zero.
3. Take the async `getUser` function from this lesson. Call `getUser(0)` inside
   an `async` function with `try`/`catch`, and print a friendly message in the
   `catch`.
4. Write a function that uses `JSON.parse` inside `try`/`catch`. If the text is
   invalid JSON, return an empty object `{}` instead of crashing.

## Key Takeaways

- An **error** is JavaScript saying it cannot do something; uncaught errors crash
  the program.
- The **Error object** has a `name` and a `message`.
- Use `throw new Error("...")` to raise your own errors.
- `try`/`catch`/`finally` lets you handle errors and run cleanup that always
  happens.
- You can inspect an error to handle **specific** cases, and **re-throw** when a
  higher level should handle it.
- Handle async errors with **`try`/`catch`** (`await`) or **`.catch`**
  (Promises) — never leave a rejection unhandled.
- Write **clear, specific** messages, and **fail gracefully** with a friendly
  message for the user.

---

[← Back to Course Index](../README.md)
