# Conditionals: `if`, `else`, `switch`

Real programs make decisions. "If the user is logged in, show their name.
Otherwise, show a login button." This kind of decision-making is done with
**conditionals**. A conditional runs some code only when a certain condition is
true. Think of it as a fork in the road: your program picks a path based on the
answer to a yes/no question.

## What You'll Learn

- The `if`, `else if`, and `else` statements
- Using comparison and logical operators inside conditions
- Nesting conditionals inside each other
- The ternary operator as a shortcut for simple choices
- The `switch` statement with `break` and `default`
- When to choose `switch` over `if`
- The common bug of forgetting `break`

## The `if` Statement

An `if` statement runs a block of code only when its condition is `true`. The
condition goes in parentheses `()`, and the code to run goes in curly braces `{}`.

```js
let age = 20;

if (age >= 18) {
  console.log("You are an adult."); // => You are an adult.
}
```

If the condition is `false`, the block is skipped completely.

```js
let age = 12;

if (age >= 18) {
  console.log("You are an adult."); // this line does NOT run
}
```

## `else` — the Other Path

`else` gives a block to run when the `if` condition is `false`. The program always
picks exactly one path.

```js
let age = 12;

if (age >= 18) {
  console.log("You are an adult.");
} else {
  console.log("You are a minor."); // => You are a minor.
}
```

## `else if` — More Than Two Paths

When you have several possibilities, chain them with `else if`. JavaScript checks
each condition from top to bottom and runs the **first one that is true**, then
skips the rest.

```js
let score = 75;

if (score >= 90) {
  console.log("Grade: A");
} else if (score >= 70) {
  console.log("Grade: B"); // => Grade: B
} else if (score >= 50) {
  console.log("Grade: C");
} else {
  console.log("Grade: F");
}
```

Even though `75 >= 50` is also true, only the **first** matching block (`B`) runs.
Order matters: put the most specific or highest conditions first.

## Using Operators in Conditions

A condition is just an expression that produces `true` or `false`. You can use the
comparison and logical operators you already learned.

```js
let temperature = 30;
let isSunny = true;

// Logical AND: both must be true
if (temperature > 25 && isSunny) {
  console.log("Great beach day!"); // => Great beach day!
}

// Logical OR: at least one must be true
let isWeekend = false;
if (isWeekend || temperature > 28) {
  console.log("Time to relax."); // => Time to relax.
}

// Logical NOT: flips the value
let isRaining = false;
if (!isRaining) {
  console.log("No umbrella needed."); // => No umbrella needed.
}
```

Remember to use `===` for equality checks:

```js
let role = "admin";
if (role === "admin") {
  console.log("Welcome, admin."); // => Welcome, admin.
}
```

## Nesting Conditionals

You can put an `if` inside another `if`. This is called **nesting**. Use it when a
second decision only makes sense after the first one is true.

```js
let isLoggedIn = true;
let isAdmin = true;

if (isLoggedIn) {
  if (isAdmin) {
    console.log("Show admin panel."); // => Show admin panel.
  } else {
    console.log("Show normal dashboard.");
  }
}
```

Nesting works, but too much of it becomes hard to read. Often you can combine
conditions with `&&` instead:

```js
if (isLoggedIn && isAdmin) {
  console.log("Show admin panel."); // cleaner, same result
}
```

## The Ternary Operator as a Shortcut

For a simple two-way choice, the ternary operator is a short version of
`if/else`. Its shape is `condition ? valueIfTrue : valueIfFalse`.

```js
let age = 20;

// Long way:
let status;
if (age >= 18) {
  status = "adult";
} else {
  status = "minor";
}

// Short way (does the same thing):
let status2 = age >= 18 ? "adult" : "minor";
console.log(status2); // => adult
```

Use the ternary for short, simple choices. For complex logic with many branches,
a full `if/else if` is clearer.

## The `switch` Statement

When you compare **one value** against **many specific options**, a `switch` can
be cleaner than a long `if/else if` chain. It checks the value against each
`case` and runs the matching block.

```js
let day = "Tue";

switch (day) {
  case "Mon":
    console.log("Start of the week.");
    break;
  case "Tue":
    console.log("Taco Tuesday!"); // => Taco Tuesday!
    break;
  case "Fri":
    console.log("Almost weekend.");
    break;
  default:
    console.log("Just another day.");
}
```

Key parts:

- **`switch (value)`** — the value to compare.
- **`case "Tue":`** — a specific option to match (uses strict `===` comparison).
- **`break`** — stops the switch once a match runs (very important — see below).
- **`default`** — runs when no case matches. It is like the final `else`.

### The Common Bug: Forgetting `break`

If you forget `break`, JavaScript keeps running the next cases too, even if they do
not match. This is called **fall-through**, and it is a frequent beginner bug.

```js
let fruit = "apple";

switch (fruit) {
  case "apple":
    console.log("Apple"); // => Apple
  // no break here — execution falls through!
  case "banana":
    console.log("Banana"); // => Banana  (runs even though fruit is "apple")
  default:
    console.log("Unknown"); // => Unknown (also runs!)
}
```

Almost always, you want a `break` at the end of each case to stop this.

(There is an *intentional* use of fall-through to group cases, but as a beginner,
add `break` every time unless you have a clear reason not to.)

## When to Use `switch` vs `if`

- Use **`switch`** when you compare **one variable** to several exact values
  (like a menu choice, a day name, or a status code).
- Use **`if/else if`** when you need ranges or complex conditions (like
  `score >= 70`), or when conditions use different variables and logical
  operators.

```js
// Ranges → use if/else if (switch cannot easily do ranges)
if (score >= 90) {
  /* ... */
} else if (score >= 70) {
  /* ... */
}

// Exact matches of one value → switch reads nicely
switch (statusCode) {
  case 200:
    console.log("OK");
    break;
  case 404:
    console.log("Not found");
    break;
}
```

## Common Mistakes

- **Forgetting `break` in a `switch`.** Without it, later cases run too
  (fall-through).
- **Using `=` instead of `===` in a condition.** `if (x = 5)` assigns `5` and is
  always truthy; you meant `if (x === 5)`.
- **Wrong order in `else if` chains.** The first true condition wins, so put
  specific or higher conditions first.
- **Over-nesting.** Deeply nested `if`s are hard to read; combine with `&&` when
  possible.
- **Using a ternary for complex logic.** Long ternaries are hard to read; use a
  full `if/else` instead.

## Exercises

1. Write an `if/else if/else` chain that prints a message for a `temperature`:
   "freezing" below 0, "cold" below 15, "warm" below 28, otherwise "hot".
2. Rewrite this `if/else` as a ternary: if `isMember` is `true`, set
   `price = 8`, else set `price = 10`.
3. Write a `switch` on a `command` variable that handles `"start"`, `"stop"`, and
   `"pause"`, with a `default` for unknown commands. Make sure each case has a
   `break`.
4. Take the switch from exercise 3, remove one `break`, and observe how the output
   changes. Write down what fall-through did.

## Key Takeaways

- Conditionals let your program choose different paths based on a condition.
- `if` runs code when a condition is true; `else if` and `else` handle other
  cases.
- Conditions use comparison (`===`, `>`, etc.) and logical (`&&`, `||`, `!`)
  operators.
- The ternary `condition ? a : b` is a shortcut for simple two-way choices.
- `switch` compares one value to many exact cases; always add `break` and use
  `default` for the fallback.
- Forgetting `break` causes fall-through, a very common bug.

---

[← Back to Course Index](../README.md) | [Next: Loops →](06-loops.md)
