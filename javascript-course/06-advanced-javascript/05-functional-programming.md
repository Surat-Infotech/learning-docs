# Functional Programming Ideas

**Functional programming** (FP) is a *style* of writing code. It is not a new
language or a special tool — it is a set of habits that make your code more
predictable and easier to trust. You have already used some of these ideas
(`map`, `filter`, `reduce`). This lesson connects the dots and gives names to the
ideas.

Do not think of FP as "advanced math." Think of it as "small, honest functions
that do exactly what they say." That is the whole spirit.

## What You'll Learn

- **Pure functions** and why they are great
- **Immutability** (not changing data in place)
- Avoiding **side effects**
- **First-class functions** (functions are values)
- **Higher-order functions** (functions that take or return functions)
- Using **`map` / `filter` / `reduce`** as functional tools
- **Function composition** (chaining small functions)
- **Currying** (one argument at a time)
- Why these ideas make code easier to test and reason about

---

## Pure Functions

A **pure function** follows two rules:

1. **Same input, same output.** Give it the same arguments and it always returns
   the same result.
2. **No side effects.** It does not change anything outside itself (no editing
   global variables, no writing to the page, no logging — just compute and return).

Think of a pure function like a **good vending machine**: press B4, you always get
the same snack, and nothing else in the world changes.

```js
// Pure: depends only on its inputs, changes nothing outside.
function add(a, b) {
  return a + b;
}
console.log(add(2, 3)); // => 5 (always, forever)
```

```js
// Impure: depends on outside state and changes it.
let total = 0;
function addToTotal(n) {
  total += n;       // changes something outside!
  return total;
}
addToTotal(5); // => 5
addToTotal(5); // => 10  (same input, different output — not pure)
```

Pure functions are easy to test because you do not need to set up the world
around them. You just check: given X, do I get Y?

---

## Immutability

**Immutability** means you do not change existing data. Instead, you make a
**new** copy with the change. The original stays untouched.

Why care? When data can change behind your back, bugs hide. If data never changes
once created, you can trust it.

```js
// Mutating (changing the original) — avoid this in FP
const nums = [1, 2, 3];
nums.push(4); // original array is now changed
console.log(nums); // => [1, 2, 3, 4]
```

```js
// Immutable — make a new array instead
const nums = [1, 2, 3];
const more = [...nums, 4]; // copy + add
console.log(nums); // => [1, 2, 3]  (untouched!)
console.log(more); // => [1, 2, 3, 4]
```

Same idea for objects:

```js
const user = { name: "Amy", age: 30 };
const older = { ...user, age: 31 }; // copy, override age

console.log(user.age);  // => 30 (original unchanged)
console.log(older.age); // => 31
```

---

## Avoiding Side Effects

A **side effect** is anything a function does to the outside world besides
returning a value: changing a global variable, editing the DOM, saving a file,
printing to the console.

Side effects are not evil — programs need them to be useful! The FP goal is to
**keep them out of your calculation code** and push them to the edges. That way
the "thinking" part of your code is pure and easy to test, while the "doing" part
is small and separate.

```js
// Calculation: pure
function calculateTotal(prices) {
  return prices.reduce((sum, p) => sum + p, 0);
}

// Side effect: kept separate
const prices = [10, 20, 30];
const total = calculateTotal(prices); // pure work
console.log("Total:", total);         // the side effect lives here
// => Total: 60
```

---

## First-Class Functions

In JavaScript, **functions are values**, just like numbers and strings. This is
called functions being **first-class**. You can:

- Store a function in a variable.
- Put functions in arrays or objects.
- Pass a function to another function.
- Return a function from a function.

```js
const greet = function (name) {
  return "Hi " + name;
};

const actions = [greet]; // functions in an array
console.log(actions[0]("Sam")); // => Hi Sam
```

This single feature is what makes everything else in this lesson possible.

---

## Higher-Order Functions

A **higher-order function** is a function that either **takes a function** as an
argument, **returns a function**, or both. The name sounds scary, but you have
used these already.

```js
// Takes a function as an argument:
function repeat(times, action) {
  for (let i = 0; i < times; i++) {
    action(i);
  }
}

repeat(3, (i) => console.log("Run", i));
// => Run 0
// => Run 1
// => Run 2
```

```js
// Returns a function:
function multiplier(factor) {
  return function (n) {
    return n * factor;
  };
}

const double = multiplier(2);
const triple = multiplier(3);
console.log(double(5)); // => 10
console.log(triple(5)); // => 15
```

`multiplier` builds and hands back a brand-new function. This is a tiny example of
how flexible first-class functions are.

---

## `map`, `filter`, `reduce` as Functional Tools

These three array methods are the everyday tools of functional JavaScript. They
each take a function and return a **new** result without changing the original
array — perfectly in the FP spirit.

```js
const numbers = [1, 2, 3, 4, 5];

// map: transform each item -> new array
const doubled = numbers.map((n) => n * 2);
console.log(doubled); // => [2, 4, 6, 8, 10]

// filter: keep items that pass a test -> new array
const evens = numbers.filter((n) => n % 2 === 0);
console.log(evens); // => [2, 4]

// reduce: boil the array down to one value
const sum = numbers.reduce((total, n) => total + n, 0);
console.log(sum); // => 15

console.log(numbers); // => [1, 2, 3, 4, 5] (original untouched!)
```

You can chain them, reading left to right like a sentence:

```js
const result = numbers
  .filter((n) => n % 2 === 1) // keep odds: [1, 3, 5]
  .map((n) => n * 10);        // times ten: [10, 30, 50]
console.log(result); // => [10, 30, 50]
```

> For a deeper dive on these, see
> [Array Methods (`map`, `filter`, `reduce`)](../03-data-structures/02-array-methods.md).

---

## Function Composition

**Composition** means building a big operation by chaining small functions, where
the output of one becomes the input of the next. Like an assembly line: each
station does one small job, and the product flows through.

```js
const trim = (s) => s.trim();
const lower = (s) => s.toLowerCase();
const exclaim = (s) => s + "!";

// Do them one after another:
const shout = (s) => exclaim(lower(trim(s)));

console.log(shout("  HELLO  ")); // => hello!
```

A small helper can make composition cleaner:

```js
// compose runs functions right-to-left
const compose = (...fns) => (x) => fns.reduceRight((acc, fn) => fn(acc), x);

const shout = compose(exclaim, lower, trim);
console.log(shout("  HELLO  ")); // => hello!
```

Each piece stays tiny and testable, and you assemble them like LEGO bricks.

---

## Currying

**Currying** means turning a function that takes several arguments into a chain of
functions that each take **one** argument. It lets you "pre-fill" some arguments
now and supply the rest later.

```js
// Normal function
function add(a, b) {
  return a + b;
}

// Curried version
function curriedAdd(a) {
  return function (b) {
    return a + b;
  };
}

console.log(curriedAdd(2)(3)); // => 5

// Pre-fill the first argument to make a new specialized function:
const add10 = curriedAdd(10);
console.log(add10(5)); // => 15
console.log(add10(1)); // => 11
```

Arrow functions make currying short:

```js
const multiply = (a) => (b) => a * b;
const byThree = multiply(3);
console.log(byThree(4)); // => 12
```

Currying is handy when you keep calling a function with one of the same arguments
over and over — you bake that argument in once.

---

## Why These Ideas Help

- **Easier to test.** Pure functions need no setup; just check input → output.
- **Easier to reason about.** If data never changes secretly, you can trust it.
- **Fewer bugs.** Side effects in one place are easier to track than scattered
  everywhere.
- **More reusable.** Small, single-job functions snap together in new ways.

You do not have to go "100% functional." Even sprinkling these habits into normal
code makes it noticeably cleaner.

---

## Common Mistakes

- **Mutating data accidentally.** `array.sort()` and `array.push()` change the
  original. Copy first (`[...array].sort()`) if you want to stay immutable.
- **Hiding side effects inside "calculation" functions.** Keep `console.log`, DOM
  edits, and saves out of your pure logic.
- **Overusing currying/composition** until code is hard to read. Use them where
  they genuinely help, not to show off.
- **Thinking `map`/`filter` change the array.** They return *new* arrays; the
  original stays the same.
- **Forgetting the starting value in `reduce`.** Pass the initial accumulator
  (like `0` or `[]`) to avoid surprises.

---

## Exercises

1. Write a **pure** function `toFahrenheit(celsius)` and prove it is pure by
   calling it twice with the same input and getting the same output.

2. Take an array `[5, 1, 4, 2, 3]` and produce a **new** sorted array without
   changing the original. Print both to show the original is untouched.

3. Write a higher-order function `makeGreeter(greeting)` that returns a function.
   `makeGreeter("Hello")("Amy")` should give `"Hello, Amy"`.

4. Use `map`, `filter`, and `reduce` together: from `[1,2,3,4,5,6]`, keep the even
   numbers, double them, and sum the result. (Expected: 24.)

---

## Key Takeaways

- **Pure functions**: same input → same output, no side effects.
- **Immutability**: make new copies instead of changing data in place.
- Keep **side effects** at the edges, away from your calculation code.
- Functions are **first-class** values, which powers **higher-order functions**.
- `map` / `filter` / `reduce` are your everyday functional tools.
- **Composition** chains small functions; **currying** feeds arguments one at a
  time.
- These habits make code easier to test, reuse, and trust.

---

[← Back to Regular Expressions](./04-regular-expressions.md) | [Next: Useful Advanced Patterns →](./06-advanced-patterns.md)
