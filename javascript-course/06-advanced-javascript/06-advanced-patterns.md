# Useful Advanced Patterns

A **pattern** is a tried-and-true way to solve a common problem. Instead of
inventing a solution each time, you reach for a pattern that experienced
developers already trust. This lesson collects several small, practical patterns
you will see (and use) in real projects.

For each one we will answer two questions: **What problem does it solve?** and
**How do I use it?** with a small example.

## What You'll Learn

- **Debounce** and **throttle** (and when to use each)
- The **module pattern / IIFE** for privacy
- **Optional chaining** `?.` and **nullish coalescing** `??`
- **Short-circuit** evaluation tricks
- **Shallow vs deep copy** (and `structuredClone`)
- **Event delegation** (quick recap)
- **Memoization** (a simple cache)

---

## Debounce

**Problem:** Some events fire *way* too often. As a user types in a search box,
the `input` event fires on every keystroke. If you call an API on each one, you
flood the server.

**Idea:** Debounce says "wait until the user *stops* for a moment, then act." Each
new event resets the timer. Think of an elevator door: every new person resets the
"close" timer; the door only closes once people stop arriving.

```js
function debounce(fn, delay) {
  let timer; // remembered between calls thanks to closure
  return function (...args) {
    clearTimeout(timer);                 // cancel the previous wait
    timer = setTimeout(() => fn(...args), delay); // start a new wait
  };
}

// Usage:
const search = debounce((text) => {
  console.log("Searching for:", text);
}, 500);

search("h");
search("he");
search("hel"); // only this last one runs, 500ms after typing stops
// => Searching for: hel
```

**Use it for:** search-as-you-type, resizing the window, auto-saving a draft —
anywhere you want to act *after activity settles*.

---

## Throttle

**Problem:** Sometimes you do not want to wait for things to stop — you want to
run regularly, but **not too often**. Scroll events can fire dozens of times per
second.

**Idea:** Throttle says "run at most once every X milliseconds, no matter how many
times you are called." Think of a turnstile that only lets one person through every
few seconds.

```js
function throttle(fn, limit) {
  let waiting = false;
  return function (...args) {
    if (!waiting) {
      fn(...args);          // run now
      waiting = true;
      setTimeout(() => (waiting = false), limit); // open up again later
    }
  };
}

const onScroll = throttle(() => {
  console.log("Scroll position checked");
}, 1000);

// Even if scroll fires 100 times a second, this runs at most once per second.
```

**Debounce vs throttle in one line:** *Debounce* waits for quiet, then acts once.
*Throttle* acts on a steady schedule while activity continues. Use debounce for
"when they finish," throttle for "every so often."

---

## The Module Pattern / IIFE (Privacy)

**Problem:** You want some variables to be **private** — hidden from the rest of
the program — but still expose a few public functions.

**Idea:** Wrap your code in a function that runs immediately. Variables inside stay
private; you return only what you want to share. A function that runs itself right
away is called an **IIFE** (Immediately Invoked Function Expression).

```js
const counter = (function () {
  let count = 0; // private! cannot be touched from outside

  return {
    increment() {
      count++;
      return count;
    },
    reset() {
      count = 0;
    },
  };
})(); // the () at the end runs it immediately

console.log(counter.increment()); // => 1
console.log(counter.increment()); // => 2
counter.reset();
console.log(counter.increment()); // => 1
console.log(counter.count);       // => undefined (it is private!)
```

The outside world can only call `increment` and `reset`. The `count` variable is
safely locked away. (Modern code often uses ES Modules or class private fields
`#count` for the same goal, but this classic pattern still shows up.)

---

## Optional Chaining `?.`

**Problem:** Reaching into nested objects can crash if something in the middle is
missing.

```js
const user = { name: "Amy" };
// console.log(user.address.city);
// => TypeError: Cannot read properties of undefined (reading 'city')
```

**Idea:** `?.` checks each step. If something is `null` or `undefined`, it stops
and returns `undefined` instead of throwing an error.

```js
const user = { name: "Amy" };
console.log(user.address?.city); // => undefined (no crash!)

const user2 = { address: { city: "Paris" } };
console.log(user2.address?.city); // => Paris
```

It also works for calling methods that might not exist:

```js
const obj = {};
obj.doThing?.(); // does nothing instead of crashing
```

---

## Nullish Coalescing `??`

**Problem:** You want a default value, but only when the value is *truly missing*
(`null` or `undefined`) — not when it is `0` or `""`, which are valid.

The old trick `value || default` treats `0` and `""` as "missing," which is often
wrong:

```js
const count = 0;
console.log(count || 10); // => 10  (oops! 0 is valid but got replaced)
```

**Idea:** `??` only falls back for `null` or `undefined`:

```js
const count = 0;
console.log(count ?? 10);     // => 0   (kept, because 0 is a real value)

const name = null;
console.log(name ?? "Guest"); // => Guest
```

Pair `?.` and `??` for safe defaults:

```js
const city = user.address?.city ?? "Unknown";
```

---

## Short-Circuit Evaluation Tricks

`&&` and `||` stop as soon as the answer is known. You can use this for handy
shortcuts.

```js
// `&&` runs the right side only if the left is truthy.
const user = { name: "Sam" };
user.name && console.log("Hello", user.name); // => Hello Sam

// `||` gives the first truthy value (a quick default).
const input = "";
const display = input || "Anonymous";
console.log(display); // => Anonymous
```

These are concise, but do not overuse them — readability beats cleverness. For
real defaults, prefer `??` so `0` and `""` are respected.

---

## Shallow vs Deep Copy

**Problem:** Copying objects is trickier than it looks. A **shallow** copy copies
the top level, but nested objects are still *shared* with the original.

```js
const original = { name: "Amy", hobbies: ["reading"] };
const shallow = { ...original }; // spread = shallow copy

shallow.name = "Bob";          // top-level: independent
shallow.hobbies.push("coding"); // nested: SHARED!

console.log(original.name);    // => Amy (good, separate)
console.log(original.hobbies); // => ['reading', 'coding'] (changed! shared)
```

**Idea (deep copy):** A **deep** copy duplicates everything, all the way down, so
nothing is shared. The modern, easy way is `structuredClone`:

```js
const original = { name: "Amy", hobbies: ["reading"] };
const deep = structuredClone(original);

deep.hobbies.push("coding");
console.log(original.hobbies); // => ['reading'] (untouched!)
console.log(deep.hobbies);     // => ['reading', 'coding']
```

> An older trick was `JSON.parse(JSON.stringify(obj))`. It works for plain data
> but loses things like functions, `undefined`, and `Date` objects. Prefer
> `structuredClone` when available.

---

## Event Delegation (Quick Recap)

**Problem:** You have many similar elements (say, 100 list items) and want to
handle clicks on all of them. Adding 100 separate listeners is wasteful, and it
breaks for items added later.

**Idea:** Add **one** listener to the parent. Thanks to event bubbling, clicks on
children travel up to the parent, where you check what was actually clicked.

```js
const list = document.querySelector("#todo-list");

list.addEventListener("click", (event) => {
  // Did the click land on a list item?
  if (event.target.matches("li")) {
    console.log("You clicked:", event.target.textContent);
  }
});
```

One listener handles all items — even ones added in the future. (For the full
story on events and bubbling, see the Events lesson in Module 4.)

---

## Memoization (A Simple Cache)

**Problem:** Some functions are slow or do repeated work. If you call them with
the same input many times, you waste effort recomputing the same answer.

**Idea:** **Memoization** remembers past results. On the next call with the same
input, return the saved answer instantly. Think of a calculator that writes down
answers so it does not redo the math.

```js
function memoize(fn) {
  const cache = new Map(); // input -> result
  return function (n) {
    if (cache.has(n)) {
      return cache.get(n); // saved! return instantly
    }
    const result = fn(n);
    cache.set(n, result);  // remember for next time
    return result;
  };
}

// A "slow" function we want to speed up:
const slowSquare = (n) => {
  console.log("Computing...");
  return n * n;
};

const fastSquare = memoize(slowSquare);

console.log(fastSquare(4)); // logs "Computing...", => 16
console.log(fastSquare(4)); // no log this time,     => 16 (from cache)
```

The first call computes; the second uses the cache. Great for expensive
calculations called repeatedly with the same inputs.

---

## Common Mistakes

- **Mixing up debounce and throttle.** Debounce waits for quiet; throttle runs on
  a schedule. Pick based on the behavior you want.
- **Using `||` for defaults when `0` or `""` are valid.** Use `??` instead.
- **Assuming spread gives a deep copy.** `{...obj}` is shallow — nested objects are
  still shared. Use `structuredClone` for a true deep copy.
- **Adding a listener per element** when one delegated listener on the parent
  would do.
- **Memoizing impure functions.** Caching only makes sense for pure functions
  (same input → same output). Caching something with side effects causes bugs.

---

## Exercises

1. Write a debounced function that logs `"saved"` and attach it to an input's
   `keyup` event so it only logs once the user stops typing for 1 second.

2. Use optional chaining and nullish coalescing to safely read
   `settings.theme.color` from an object, defaulting to `"light"` when anything is
   missing.

3. Make a shallow copy and a deep copy of an object that contains a nested array.
   Change the nested array in each copy and show how the shallow copy affects the
   original but the deep copy does not.

4. Write a `memoize`-wrapped function that computes the factorial of a number.
   Call it twice with the same input and confirm the second call uses the cache.

---

## Key Takeaways

- **Debounce** waits for activity to stop; **throttle** limits how often code runs.
- The **IIFE / module pattern** keeps variables private and exposes a small API.
- `?.` (**optional chaining**) avoids crashes on missing nested values.
- `??` (**nullish coalescing**) gives defaults only for `null`/`undefined`.
- **Short-circuiting** with `&&`/`||` enables neat shortcuts — use sparingly.
- Spread is a **shallow** copy; use **`structuredClone`** for a deep copy.
- **Event delegation** uses one parent listener for many children.
- **Memoization** caches results of pure functions to skip repeat work.

You made it through Module 6! These patterns are the kind of thing that makes you
look like a seasoned developer. Next up: working like a pro.

---

[← Back to Functional Programming](./05-functional-programming.md) | [Next: npm and Package Management →](../07-professional-skills/01-npm-and-packages.md)
