# Arrow Functions

Arrow functions are a shorter, newer way to write functions. They were added in
modern JavaScript (ES6). Think of them as a quicker way to say the same thing.
Instead of writing the long `function` keyword every time, you use a small arrow
`=>`.

You will see arrow functions everywhere in modern code, especially when working
with arrays. So it is important to feel comfortable with them.

## What You'll Learn

- The arrow function syntax
- How to convert a normal function into an arrow function
- Implicit return (a shortcut for one-line functions)
- Arrow functions with zero, one, or many parameters
- When arrow functions are most handy
- A light note on how arrow functions treat `this`

## The Arrow Function Syntax

A normal function looks like this:

```js
const add = function (a, b) {
  return a + b;
};
```

The same thing as an arrow function:

```js
const add = (a, b) => {
  return a + b;
};

console.log(add(2, 3)); // => 5
```

What changed?

- We removed the word `function`.
- We added `=>` (an arrow) after the parentheses.

That is it. The body inside `{ }` works exactly the same.

## Converting a Normal Function to an Arrow Function

Here is the step-by-step recipe.

```js
// Step 1: Start with a function expression
const greet = function (name) {
  return "Hi, " + name;
};

// Step 2: Remove "function", add "=>" after the ()
const greet2 = (name) => {
  return "Hi, " + name;
};

console.log(greet2("Vinay")); // => Hi, Vinay
```

## Implicit Return (One-Line Shortcut)

If your arrow function just returns one thing, you can make it even shorter. You
can remove the `{ }` braces **and** the `return` word. This is called an
**implicit return** ("implicit" means "happens automatically").

```js
// Long version with braces and return
const double = (n) => {
  return n * 2;
};

// Short version: no braces, no "return"
const doubleShort = (n) => n * 2;

console.log(doubleShort(5)); // => 10
```

Rule: if there are no `{ }` braces, the value after the arrow is returned
automatically.

Be careful: the moment you add `{ }`, you must write `return` yourself.

```js
const bad = (n) => { n * 2 };  // returns undefined! No "return" inside braces.
const good = (n) => { return n * 2 };

console.log(bad(5));  // => undefined
console.log(good(5)); // => 10
```

## Zero, One, or Many Parameters

The parentheses change a little depending on how many parameters you have.

**Zero parameters:** you must keep the empty parentheses `()`.

```js
const sayHi = () => "Hello!";
console.log(sayHi()); // => Hello!
```

**One parameter:** the parentheses are optional. Both work.

```js
const square = (n) => n * n; // with parentheses
const cube = n => n * n * n; // without parentheses (also fine)

console.log(square(4)); // => 16
console.log(cube(2));   // => 8
```

**Many parameters:** you must use parentheses.

```js
const addThree = (a, b, c) => a + b + c;
console.log(addThree(1, 2, 3)); // => 6
```

A small tip: many people always use parentheses, even for one parameter, to keep
the style the same everywhere. Both are correct.

## When Arrow Functions Are Handy

Arrow functions shine when you pass a small function into another function. These
small passed-in functions are called **callbacks**.

Array methods are the most common place you will see this. (We cover these in
detail in [Array Methods](../03-data-structures/02-array-methods.md).)

```js
const numbers = [1, 2, 3, 4];

// "map" runs the arrow function on each item and builds a new array
const doubled = numbers.map((n) => n * 2);
console.log(doubled); // => [2, 4, 6, 8]

// "filter" keeps only items where the arrow function returns true
const evens = numbers.filter((n) => n % 2 === 0);
console.log(evens); // => [2, 4]
```

See how short and clean that is? Writing the full `function` keyword each time
would be much longer.

Arrow functions are also great with timers:

```js
setTimeout(() => {
  console.log("3 seconds passed!");
}, 3000);
```

## A Note on `this`

There is one important difference between arrow functions and normal functions:
they treat the special keyword `this` differently.

In short: **an arrow function does not have its own `this`.** It uses the `this`
from the code around it. This is sometimes very helpful and sometimes confusing.

Do not worry about the details now. We explain `this` slowly and carefully in the
next lessons. Just remember this one sentence for later:

> Arrow functions borrow `this` from their surroundings.

Read more in [The `this` Keyword](05-the-this-keyword.md).

## Common Mistakes

- **Forgetting that braces need `return`.** `n => { n * 2 }` returns `undefined`.
  Use `n => n * 2` or `n => { return n * 2 }`.
- **Dropping the empty `()` for zero parameters.** You still need `()` when there
  are no parameters: `() => "Hi"`.
- **Using an arrow function as an object method when you need `this`.** Because
  arrow functions do not have their own `this`, they are usually a bad choice for
  object methods. (More on this in the `this` lesson.)
- **Trying to return an object without parentheses.** `() => { name: "Vinay" }`
  fails because the `{` looks like a function body. Wrap the object in `()`:
  `() => ({ name: "Vinay" })`.

## Exercises

1. Convert this function into an arrow function with an implicit return:
   ```js
   function triple(n) {
     return n * 3;
   }
   ```
2. Write an arrow function `greet` that takes a `name` and returns
   `"Hello, <name>"`. Use the implicit return style.
3. Given `const prices = [10, 20, 30];`, use `map` with an arrow function to
   create a new array where every price is increased by 5.
4. Write a zero-parameter arrow function called `randomDie` that returns a random
   whole number from 1 to 6. (Hint: `Math.floor(Math.random() * 6) + 1`.)

## Key Takeaways

- Arrow functions are a shorter way to write functions using `=>`.
- Remove `function`, add `=>` after the parentheses.
- No braces means the value is returned automatically (implicit return).
- With braces, you must write `return` yourself.
- Zero parameters need empty `()`; one parameter can skip the `()`; many need
  `()`.
- They are perfect for callbacks and array methods like `map` and `filter`.
- Arrow functions treat `this` differently: they borrow it from their
  surroundings.

---

[← Back to Course Index](../README.md) | [Next: Scope and Hoisting →](03-scope-and-hoisting.md)
