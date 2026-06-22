# The `this` Keyword

`this` is one of the most confusing words in JavaScript. Many experienced
developers still pause to think about it. So if it confuses you, you are normal.
We will go slowly and use simple analogies.

Here is the big secret up front:

> `this` does not depend on *where* a function is written. It depends on *how* the
> function is called.

Read that again. It is the key to everything. The value of `this` is decided at
**call time**, not at write time.

## What You'll Learn

- What `this` actually means
- `this` in the global scope
- `this` in a regular function
- `this` inside an object method
- `this` in arrow functions (it inherits from around it)
- Common `this` bugs (losing `this` in callbacks)
- `call`, `apply`, and `bind` (briefly)

## An Analogy First

Imagine the word "here". If *you* say "I am here", "here" means your location. If
*I* say "I am here", "here" means my location. The word "here" changes meaning
depending on who says it.

`this` is like the word "here". It changes meaning depending on **who calls the
function**.

## `this` in the Global Scope

When you use `this` outside any function, it points to the global object.

```js
console.log(this);
// In a browser:    => Window (the global object)
// In Node modules: => {} (an empty object) or module.exports
```

You rarely need `this` at the top level. The interesting cases are inside
functions.

## `this` in a Regular Function

When you call a plain function by itself, `this` is not tied to any object. In
**strict mode** (which modern code and modules use), `this` is `undefined`.

```js
"use strict";

function showThis() {
  console.log(this);
}

showThis(); // => undefined (in strict mode)
```

Without strict mode, `this` would fall back to the global object. Either way, a
lonely function call gives you no useful `this`. The value only becomes useful
when a function is called *as part of an object*.

## `this` in an Object Method

A function stored inside an object is called a **method**. When you call a method
using `object.method()`, `this` points to the object on the left of the dot.

```js
const user = {
  name: "Vinay",
  greet() {
    console.log("Hi, I am " + this.name);
  },
};

user.greet(); // => Hi, I am Vinay
```

Here `this` is `user`, because we called `user.greet()`. The thing before the dot
becomes `this`. This is the most common and most useful case.

```js
const dog = {
  name: "Rex",
  speak() {
    console.log(this.name + " says woof");
  },
};

dog.speak(); // => Rex says woof
```

**Remember:** look to the left of the dot when the function is called. That is
usually your `this`.

## `this` in Arrow Functions

Arrow functions are different. **An arrow function does not have its own `this`.**
Instead, it borrows `this` from the code around it (its surrounding scope). It
does not care how it is called.

This can help or hurt. First, a place where it *hurts*:

```js
const user = {
  name: "Vinay",
  greet: () => {
    // Arrow borrows "this" from OUTSIDE the object, not from "user"
    console.log("Hi, I am " + this.name);
  },
};

user.greet(); // => Hi, I am undefined
```

The arrow did not get `user` as `this`. So **avoid arrow functions for object
methods.** Use a normal method instead (like the earlier example).

Now a place where the arrow *helps*: inside another method, where we want to keep
the outer `this`. We will see this next.

## The Common Bug: Losing `this` in Callbacks

This is the most famous `this` problem. It happens when a method passes another
function (a callback) to something like `setTimeout` or an array method.

```js
const user = {
  name: "Vinay",
  greetLater() {
    setTimeout(function () {
      // PROBLEM: this regular function has its OWN this (undefined / global)
      console.log("Hi, I am " + this.name);
    }, 1000);
  },
};

user.greetLater(); // => Hi, I am undefined
```

What went wrong? The inner function is a plain function. `setTimeout` calls it on
its own, not as `user.something()`. So `this` is lost.

**Fix: use an arrow function.** Because arrows borrow `this` from around them, the
inner arrow keeps the `this` of `greetLater`, which is `user`.

```js
const user = {
  name: "Vinay",
  greetLater() {
    setTimeout(() => {
      // Arrow borrows "this" from greetLater, which is "user"
      console.log("Hi, I am " + this.name);
    }, 1000);
  },
};

user.greetLater(); // => Hi, I am Vinay
```

This is the number one real-world reason people love arrow functions. Keep this
example in mind.

## `call`, `apply`, and `bind` (Briefly)

Sometimes you want to choose `this` yourself. JavaScript gives three tools for
that. They all live on functions.

First, a sample function and two objects:

```js
function introduce(greeting) {
  console.log(greeting + ", I am " + this.name);
}

const person1 = { name: "Vinay" };
const person2 = { name: "Asha" };
```

### `call` — run now, with a chosen `this`

`call` runs the function right away. The first argument becomes `this`. The rest
are normal arguments.

```js
introduce.call(person1, "Hello"); // => Hello, I am Vinay
introduce.call(person2, "Hi");    // => Hi, I am Asha
```

### `apply` — same as `call`, but arguments in an array

`apply` is almost identical to `call`. The only difference: you pass the
arguments as an **array**.

```js
introduce.apply(person1, ["Hey"]); // => Hey, I am Vinay
```

Easy memory trick: **a**pply uses an **a**rray.

### `bind` — make a new function with a fixed `this`

`bind` does **not** run the function now. It returns a **new function** that will
always use the `this` you gave, no matter how it is later called.

```js
const introduceVinay = introduce.bind(person1);

introduceVinay("Welcome"); // => Welcome, I am Vinay

// Even passed to setTimeout, "this" stays as person1
setTimeout(introduceVinay, 500, "Later"); // => Later, I am Vinay
```

`bind` is another classic fix for the "lost `this` in a callback" problem, used
instead of arrow functions.

## Common Mistakes

- **Thinking `this` depends on where a function is written.** It depends on *how*
  it is called.
- **Using an arrow function as an object method.** Arrows do not get the object
  as `this`. Use a normal method.
- **Passing a method as a callback and losing `this`.** Fix it with an arrow
  function or `.bind()`.
- **Confusing `call` and `apply`.** Both set `this` and run now; `apply` takes an
  array of arguments, `call` takes them one by one.
- **Forgetting `bind` returns a new function.** It does not run anything by
  itself.

## Exercises

1. Create an object `car` with a property `brand` and a method `describe()` that
   logs `"This car is a <brand>"`. Call it and confirm `this` works.
2. Take the `car` object and rewrite `describe` as an arrow function. Observe how
   `this.brand` becomes `undefined`, and explain why.
3. Write a method `startCountdown()` on an object that uses `setTimeout` to log
   the object's name after 1 second. Make it work using an arrow function inside.
4. Write a function `sayHi()` that logs `"Hi, " + this.name`. Use `call`,
   `apply`, and `bind` to run it with an object `{ name: "Asha" }`.

## Key Takeaways

- `this` depends on **how a function is called**, not where it is written.
- In the global scope, `this` is the global object (or `{}` in Node modules).
- In a plain function (strict mode), `this` is `undefined`.
- In an **object method** called with the dot, `this` is the object before the
  dot.
- **Arrow functions** have no `this`; they borrow it from their surroundings.
- A common bug is **losing `this` in callbacks**; fix it with an arrow function
  or `.bind()`.
- `call` and `apply` run a function with a chosen `this` (apply uses an array);
  `bind` returns a new function with a locked-in `this`.

---

[← Back to Course Index](../README.md)
