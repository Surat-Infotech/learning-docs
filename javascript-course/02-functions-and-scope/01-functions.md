# Functions

Imagine you have a coffee machine. You press a button, and it gives you coffee.
You do not need to know *how* it works inside. You just use it, again and again.

A **function** in JavaScript is like that coffee machine. It is a reusable block
of logic. You write the steps once, give it a name, and then run it whenever you
want. This saves you from writing the same code over and over.

## What You'll Learn

- What a function is and why we use them
- How to write a function declaration
- The difference between parameters and arguments
- How to return a value from a function
- How to call (run) a function
- Default parameters
- Functions that return nothing (`undefined`)
- Function expressions and functions as values
- A simple calculator example
- Pure vs impure functions (in plain words)

## What Is a Function?

A function is a named block of code that does one job. You write it once. You
can run it many times. Running a function is called **calling** or **invoking**
it.

Why use functions?

- **Reuse:** Write the logic once, use it everywhere.
- **Clean code:** Hide messy details behind a clear name.
- **Fewer bugs:** Fix the code in one place instead of ten places.

## Function Declarations

The most common way to make a function is a **function declaration**. It uses
the `function` keyword.

```js
// "function" keyword, then a name, then (), then { the body }
function sayHello() {
  console.log("Hello!");
}

// Now we CALL the function using its name and ()
sayHello(); // => Hello!
sayHello(); // => Hello!
sayHello(); // => Hello!
```

Notice the parentheses `()` at the end. That is what actually runs the function.
Writing `sayHello` without `()` does **not** run it.

```js
sayHello;   // does nothing useful (just points to the function)
sayHello(); // => Hello!  (this RUNS it)
```

## Parameters vs Arguments

Functions become powerful when you give them input. The input slots are called
**parameters**. The real values you pass in are called **arguments**.

Think of a parameter as an empty box with a label. An argument is the thing you
put inside the box.

```js
// "name" is a PARAMETER (the label / empty box)
function greet(name) {
  console.log("Hello, " + name + "!");
}

// "Vinay" is an ARGUMENT (the real value we put in)
greet("Vinay"); // => Hello, Vinay!
greet("Asha");  // => Hello, Asha!
```

A function can take many parameters. Separate them with commas.

```js
function introduce(name, age) {
  console.log(name + " is " + age + " years old.");
}

introduce("Vinay", 25); // => Vinay is 25 years old.
```

**Easy way to remember:**
- **P**arameters are in the function definition (**P**lan).
- **A**rguments are the **A**ctual values you send when calling.

## Return Values

So far our functions only printed text. But often we want a function to *give
back* a result so we can use it later. We do this with the `return` keyword.

```js
function add(a, b) {
  return a + b; // send the result back to whoever called us
}

let sum = add(2, 3); // the returned value is stored in "sum"
console.log(sum);    // => 5

// You can use the result right away too
console.log(add(10, 20)); // => 30
```

Important: `return` also **stops** the function. Any code after `return` does not
run.

```js
function test() {
  return "first";
  console.log("This never runs"); // unreachable code
}

console.log(test()); // => first
```

## Default Parameters

Sometimes the caller forgets to pass a value. You can give a parameter a
**default value** to use when nothing is passed.

```js
// If "name" is not given, it defaults to "friend"
function welcome(name = "friend") {
  console.log("Welcome, " + name + "!");
}

welcome("Vinay"); // => Welcome, Vinay!
welcome();        // => Welcome, friend!
```

Without a default, a missing argument becomes `undefined`:

```js
function welcomeBad(name) {
  console.log("Welcome, " + name + "!");
}

welcomeBad(); // => Welcome, undefined!
```

## Functions That Return Nothing

If a function does not have a `return` statement (or has `return` with no value),
it returns `undefined`. The word `undefined` means "no value at all".

```js
function logMessage(text) {
  console.log(text);
  // no return statement here
}

let result = logMessage("Hi"); // => Hi
console.log(result);           // => undefined
```

This is normal. Many functions exist just to *do* something (like print to the
screen), not to give back a value.

## Function Expressions

There is another way to create a function: store it inside a variable. This is
called a **function expression**.

```js
// The function has no name. We store it in the variable "square".
const square = function (n) {
  return n * n;
};

console.log(square(4)); // => 16
```

This works because in JavaScript, a function is a **value**, just like a number
or a string. You can put it in a variable.

## Functions as Values

Because functions are values, you can:

- Store them in variables
- Pass them to other functions
- Return them from functions

```js
// Pass a function into another function
function doTwice(action) {
  action();
  action();
}

function bark() {
  console.log("Woof!");
}

doTwice(bark);
// => Woof!
// => Woof!
```

Here we passed `bark` (without `()`) so we hand over the function itself, not its
result. A function passed to another function is often called a **callback**.

## A Simple Calculator Example

Let's put it all together with a small calculator.

```js
function calculate(a, b, operation = "add") {
  if (operation === "add") {
    return a + b;
  }
  if (operation === "subtract") {
    return a - b;
  }
  if (operation === "multiply") {
    return a * b;
  }
  if (operation === "divide") {
    return a / b;
  }
  return "Unknown operation";
}

console.log(calculate(6, 2));             // => 8  (default is "add")
console.log(calculate(6, 2, "subtract")); // => 4
console.log(calculate(6, 2, "multiply")); // => 12
console.log(calculate(6, 2, "divide"));   // => 3
console.log(calculate(6, 2, "power"));    // => Unknown operation
```

## Pure vs Impure Functions (Briefly)

A **pure function** is the most reliable kind of function. It has two rules:

1. Same input always gives the same output.
2. It does not change anything outside itself.

```js
// PURE: only uses its inputs, only returns a value
function addPure(a, b) {
  return a + b;
}

addPure(2, 3); // always 5, every single time
```

An **impure function** changes something outside, or depends on outside state.

```js
let total = 0;

// IMPURE: it changes "total" (something outside the function)
function addToTotal(n) {
  total = total + n;
}

addToTotal(5);
console.log(total); // => 5
```

Pure functions are easier to test and trust. You will not always use pure
functions, but it is good to prefer them when you can. We go deeper in
[Functional Programming Ideas](../06-advanced-javascript/05-functional-programming.md).

## Common Mistakes

- **Forgetting the `()` when calling.** `sayHello` does nothing; `sayHello()`
  runs it.
- **Confusing `return` with `console.log`.** `console.log` only *prints* to the
  screen. `return` *gives back* a value you can use later. They are not the same.
- **Putting code after `return`.** Code after a `return` never runs.
- **Mixing up parameters and arguments.** Parameters are the labels in the
  definition; arguments are the real values you pass.
- **Expecting a value from a function with no `return`.** It gives back
  `undefined`.

## Exercises

1. Write a function `multiply(a, b)` that returns the product of two numbers.
   Call it with `4` and `5` and print the result.
2. Write a function `greetUser(name)` that returns the string
   `"Hi, <name>!"`. Give `name` a default value of `"guest"`. Test it with and
   without an argument.
3. Write a function `isEven(n)` that returns `true` if a number is even and
   `false` if it is odd. (Hint: a number is even if `n % 2 === 0`.)
4. Create a function expression stored in a variable called `cube` that returns
   a number multiplied by itself three times. Print `cube(3)`.

## Key Takeaways

- A function is a reusable, named block of logic.
- Use `function name() { ... }` to declare a function, and `name()` to call it.
- **Parameters** are the input labels; **arguments** are the real values.
- `return` sends a value back and stops the function.
- A function with no `return` gives back `undefined`.
- A **function expression** stores a function in a variable.
- Functions are **values**: you can store, pass, and return them.
- **Pure** functions always give the same output and change nothing outside.

---

[← Back to Course Index](../README.md) | [Next: Arrow Functions →](02-arrow-functions.md)
