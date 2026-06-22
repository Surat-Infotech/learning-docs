# Scope and Hoisting

Where can a variable be seen? Where does it disappear? This is what **scope** is
about. Scope is like a set of rooms in a house. Some things are visible only in
one room. Some things are visible in the whole house.

**Hoisting** is a related idea about *when* variables and functions become
available. It explains some surprising JavaScript behavior. Once you understand
both, a whole class of confusing bugs will make sense.

## What You'll Learn

- What scope means
- Global scope, function scope, and block scope
- How `let` and `const` differ from `var`
- The scope chain (inner code can see outer variables)
- What hoisting is, in simple terms
- The "temporal dead zone" for `let` and `const`
- Why `var` causes bugs

## What Is Scope?

**Scope** means the area of your code where a variable can be used. Outside its
scope, the variable does not exist.

Think of it like this:

- Your **name** is known everywhere in your house (global).
- A **secret** you tell only inside one room stays in that room (local).

## Global Scope

A variable declared outside of any function or block is in the **global scope**.
It can be used anywhere in your file.

```js
const appName = "MyApp"; // global

function showName() {
  console.log(appName); // can see the global variable
}

showName(); // => MyApp
console.log(appName); // => MyApp
```

Global variables are easy to use but can be dangerous. Any part of your code can
change them, which leads to bugs. Use them sparingly.

## Function Scope

A variable declared **inside** a function lives only inside that function. The
outside world cannot see it.

```js
function makeMessage() {
  const message = "Hello from inside"; // function scope
  console.log(message);
}

makeMessage(); // => Hello from inside

console.log(message); // => ReferenceError: message is not defined
```

The variable `message` was born when the function ran and died when it finished.
The outside code never sees it.

## Block Scope

A **block** is any code inside curly braces `{ }`. This includes `if` statements,
`for` loops, and even braces written on their own.

Variables made with `let` and `const` are **block-scoped**. They live only inside
the nearest `{ }`.

```js
if (true) {
  let secret = "hidden"; // lives only inside these braces
  console.log(secret);   // => hidden
}

console.log(secret); // => ReferenceError: secret is not defined
```

This is good. It keeps variables small and contained, so they do not leak out and
cause surprises.

## `let` / `const` vs `var`

This is one of the most important rules in JavaScript:

- `let` and `const` are **block-scoped** (limited to the nearest `{ }`).
- `var` is **function-scoped** (it ignores block braces and only respects
  function boundaries).

Watch what happens with a loop.

```js
// With "let": x stays inside the loop block. Good.
for (let i = 0; i < 3; i++) {
  // i exists only here
}
console.log(i); // => ReferenceError: i is not defined

// With "var": x leaks OUT of the loop. Surprising!
for (var j = 0; j < 3; j++) {
  // j is NOT limited to the loop
}
console.log(j); // => 3  (it escaped!)
```

The `var` version lets `j` escape the loop. This is exactly the kind of surprise
that causes bugs. Modern advice: **use `const` by default, `let` when you need to
reassign, and avoid `var`.**

(See [Variables](../01-fundamentals/01-variables.md) for more on choosing
between them.)

## The Scope Chain

What if a variable is not found in the current scope? JavaScript looks **outward**
into the surrounding scopes, one layer at a time, until it finds it. This chain of
lookups is called the **scope chain**.

The rule is one-directional: **inner code can see outer variables, but outer code
cannot see inner variables.**

```js
const planet = "Earth"; // outer (global)

function outer() {
  const country = "India"; // middle

  function inner() {
    const city = "Pune"; // innermost

    // inner can see ALL three
    console.log(city);   // => Pune
    console.log(country); // => India
    console.log(planet); // => Earth
  }

  inner();
  // out here we CANNOT see "city"
}

outer();
```

Think of it like nested boxes. A small box inside a big box can look outward into
the big box. But the big box cannot look inside the small box.

## Hoisting (The Simple Version)

**Hoisting** is JavaScript's behavior of moving declarations to the top of their
scope *before* the code runs. The word "hoist" means "lift up".

Here is what happens for each kind:

### `var` is hoisted as `undefined`

A `var` variable is lifted to the top, but its **value** is not. So before the
line where you set it, it exists but equals `undefined`.

```js
console.log(x); // => undefined  (not an error!)
var x = 5;
console.log(x); // => 5
```

JavaScript treats it roughly like this:

```js
var x;          // declaration hoisted to top (value is undefined)
console.log(x); // => undefined
x = 5;          // assignment stays where it was
console.log(x); // => 5
```

### Function declarations are fully hoisted

A whole **function declaration** is lifted up. This means you can call it *before*
you write it in the file.

```js
sayHi(); // => Hi!  (works even though it is defined below)

function sayHi() {
  console.log("Hi!");
}
```

This does **not** work for function expressions stored in variables, because the
variable rules apply:

```js
sayBye(); // => TypeError: sayBye is not a function

var sayBye = function () {
  console.log("Bye!");
};
```

### `let` and `const`: the Temporal Dead Zone

`let` and `const` are also hoisted, but you **cannot use them** before their line.
The gap between the top of the block and the line where the variable is declared
is called the **temporal dead zone** (TDZ). Using the variable in that gap throws
an error.

```js
console.log(y); // => ReferenceError: Cannot access 'y' before initialization
let y = 10;
```

"Temporal" means "about time". The variable is in a dead zone in *time* until its
declaration line runs. This is actually helpful: it stops you from using a
variable too early by accident, instead of silently giving you `undefined`.

## Why `var` Causes Bugs

Putting it together, `var` is risky because:

1. **It leaks out of blocks.** It ignores `if` and `for` braces.
2. **It is `undefined` before its line** instead of throwing a clear error, so
   typos and ordering mistakes hide silently.
3. **It can be re-declared** without warning, which can overwrite earlier code.

```js
var count = 1;
var count = 2; // no error! easy to do by accident
console.log(count); // => 2

let total = 1;
// let total = 2; // SyntaxError: clearer, safer
```

Because of all this, modern JavaScript uses `let` and `const`. Treat `var` as
old code you might *read* but should not *write*.

## Common Mistakes

- **Using a variable outside its block.** A `let`/`const` from inside `{ }` does
  not exist outside.
- **Expecting outer code to see inner variables.** Scope only flows inward to
  outward when *reading*; outer code cannot reach inside.
- **Calling a function expression before it is defined.** Only full function
  declarations are hoisted, not variables holding functions.
- **Relying on `var` hoisting.** Getting `undefined` instead of an error makes
  bugs hard to find.
- **Re-declaring with `var` by accident.** It silently overwrites.

## Exercises

1. Predict the output, then run it:
   ```js
   console.log(a);
   var a = 1;
   console.log(b);
   let b = 2;
   ```
   Explain why one line prints `undefined` and the other throws an error.
2. Rewrite a `for` loop that uses `var` to use `let` instead. Try to log the loop
   variable after the loop and note the difference.
3. Write three nested functions where the innermost one reads a variable from the
   outermost one. Confirm the scope chain works.
4. Create a block with `{ }` (not an `if` or loop). Declare a `const` inside it
   and try to use it outside. What happens?

## Key Takeaways

- **Scope** is where a variable can be used.
- There are three scopes: **global**, **function**, and **block**.
- `let` and `const` are **block-scoped**; `var` is **function-scoped**.
- The **scope chain** lets inner code read outer variables, but not the reverse.
- **Hoisting** lifts declarations to the top of their scope.
- `var` is hoisted as `undefined`; function declarations are fully hoisted;
  `let`/`const` sit in the **temporal dead zone** until their line.
- Prefer `const` and `let`. Avoid `var` in new code.

---

[← Back to Course Index](../README.md) | [Next: Closures →](04-closures.md)
