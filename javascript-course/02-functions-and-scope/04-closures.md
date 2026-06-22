# Closures

Closures sound scary. They are not. A closure is just a function that
**remembers** the variables from the place where it was created, even after that
place is gone.

Think of it like a backpack. When a function is created, it packs a backpack with
the variables around it. Later, wherever that function goes, it still carries the
backpack and can use what is inside.

This is one of the most loved (and most asked about in interviews) topics in
JavaScript. We will build it up slowly, one small step at a time.

## What You'll Learn

- What a closure is in plain words
- How a function "remembers" outer variables
- A step-by-step counter example (`makeCounter`)
- An ID generator example
- Practical uses: data privacy and function factories
- Why this is a common interview question

## Step 1: A Function Inside a Function

First, remember from [Scope and Hoisting](03-scope-and-hoisting.md) that an inner
function can see the variables of the function around it.

```js
function outer() {
  const message = "Hello";

  function inner() {
    console.log(message); // inner can see "message" from outer
  }

  inner();
}

outer(); // => Hello
```

Nothing magic yet. Now let's add the twist.

## Step 2: Returning the Inner Function

What if `outer` does not call `inner`, but instead **returns** it? Then we can
run `inner` later, outside of `outer`.

```js
function outer() {
  const message = "Hello";

  function inner() {
    console.log(message);
  }

  return inner; // give back the function itself (no parentheses)
}

const myFunction = outer(); // outer() has finished running now
myFunction();               // => Hello
```

Here is the amazing part. By the time we call `myFunction()`, the `outer`
function has already finished. Normally its variables would be gone. But
`message` is still there!

That is a **closure**. The returned function kept a "backpack" containing
`message`.

## What Is a Closure? (Plain Words)

A **closure** is a function bundled together with the variables it was created
near. The function "closes over" those variables, so it can keep using them later,
even after the outer function is done.

In one sentence:

> A closure is a function that remembers the variables from where it was born.

## Step 3: The Counter Example (`makeCounter`)

This is the classic example. We make a function that creates counters. Each
counter remembers its own count.

```js
function makeCounter() {
  let count = 0; // this variable is "remembered"

  return function () {
    count = count + 1; // change the remembered variable
    return count;
  };
}

const counter = makeCounter();

console.log(counter()); // => 1
console.log(counter()); // => 2
console.log(counter()); // => 3
```

Each time we call `counter()`, it reaches into its backpack, finds `count`, adds
one, and remembers the new value for next time. The `count` variable lives on
between calls.

Even better: each counter is **separate**. They do not share their `count`.

```js
const a = makeCounter();
const b = makeCounter();

console.log(a()); // => 1
console.log(a()); // => 2
console.log(b()); // => 1   (b has its own count!)
console.log(a()); // => 3
```

Counter `a` and counter `b` each got their own backpack. This is very powerful.

## Step 4: An ID Generator

Closures are great for generating unique IDs. The function remembers the last ID
it gave out.

```js
function makeIdGenerator() {
  let lastId = 0;

  return function () {
    lastId = lastId + 1;
    return "id-" + lastId;
  };
}

const nextId = makeIdGenerator();

console.log(nextId()); // => id-1
console.log(nextId()); // => id-2
console.log(nextId()); // => id-3
```

No global variable needed. The `lastId` is safely hidden inside the closure.

## Practical Use 1: Data Privacy

Variables inside a closure are **private**. Outside code cannot touch them
directly. This is a clean way to protect data.

```js
function createBankAccount(startingBalance) {
  let balance = startingBalance; // private! no one outside can change it directly

  return {
    deposit(amount) {
      balance = balance + amount;
      return balance;
    },
    withdraw(amount) {
      if (amount > balance) {
        return "Not enough money";
      }
      balance = balance - amount;
      return balance;
    },
    getBalance() {
      return balance;
    },
  };
}

const account = createBankAccount(100);

console.log(account.deposit(50));   // => 150
console.log(account.withdraw(30));  // => 120
console.log(account.getBalance());  // => 120

// You cannot reach "balance" directly:
console.log(account.balance); // => undefined
```

The only way to change `balance` is through the functions we provided. This is a
safe, controlled design.

## Practical Use 2: Function Factories

A **function factory** is a function that builds and returns other functions,
customized with remembered values.

```js
function makeMultiplier(factor) {
  return function (number) {
    return number * factor; // "factor" is remembered
  };
}

const double = makeMultiplier(2);
const triple = makeMultiplier(3);

console.log(double(5)); // => 10
console.log(triple(5)); // => 15
```

`double` remembers `factor = 2`. `triple` remembers `factor = 3`. Same factory,
different remembered values.

## Why This Is an Interview Favorite

Closures come up in interviews a lot because they test whether you really
understand scope and how functions hold on to variables. If someone asks you,
you can answer with the one-line definition:

> A closure is a function that remembers the variables from where it was created.

Then show the `makeCounter` example. That answer covers it well.

## Common Mistakes

- **Calling the inner function instead of returning it.** To make a closure you
  usually `return inner;` (no parentheses), not `return inner();`.
- **Expecting separate calls to share state.** Each call to the factory
  (`makeCounter()`) creates a *new, independent* closure with its own variables.
- **Thinking the remembered variable is a copy.** It is the *same live variable*.
  Changing it through the closure changes it for that closure permanently.
- **Trying to read private variables from outside.** That is the whole point:
  you cannot. Use the returned functions instead.

## Exercises

1. Write a `makeGreeter(greeting)` function. It should return a function that
   takes a `name` and returns `"<greeting>, <name>!"`. Create a `hello` greeter
   and a `hi` greeter and test both.
2. Build a `makeCounter` that can also count *down*. Return an object with two
   functions: `increment` and `decrement`, both sharing the same private `count`.
3. Write a function `once(fn)` that returns a new function. The new function runs
   `fn` only the first time it is called and ignores later calls. (Hint: remember
   a boolean like `hasRun` in the closure.)
4. Explain in your own words, in two sentences, what a closure is. No code.

## Key Takeaways

- A **closure** is a function that remembers the variables from where it was
  created.
- It works because the inner function keeps a "backpack" of outer variables, even
  after the outer function finishes.
- Each call to a factory function creates a new, independent closure.
- Closures power **data privacy** (hidden variables) and **function factories**.
- The classic example is `makeCounter`, where `count` lives on between calls.
- This is a common interview topic, so learn the one-line definition.

---

[← Back to Course Index](../README.md) | [Next: The `this` Keyword →](05-the-this-keyword.md)
