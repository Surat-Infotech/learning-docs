# Variables: `let`, `const`, `var`

Every program needs to remember things. A variable is how JavaScript remembers
a piece of data so you can use it later. Think of a variable as a **labeled box**:
you put something inside the box, write a name on the front, and later you can
open the box by its name to see what is inside.

## What You'll Learn

- What a variable is and why we need them
- How to create (declare) a variable with `let`, `const`, and `var`
- How to put a value into a variable (assignment)
- The differences between `let`, `const`, and `var`
- Why you should prefer `const`, then `let`, and avoid `var`
- The rules and good habits for naming variables
- The difference between reassigning and redeclaring
- What `const` really means when you use it with objects and arrays

## What Is a Variable?

A **variable** is a named place to store a value. The value can be a number, some
text, or many other things you will learn about later.

Imagine you have a box. You write `score` on the label, and you put the number
`10` inside. Later, when you say `score`, JavaScript looks in the box and finds
`10`.

```js
let score = 10; // a box named "score" holds the value 10
console.log(score); // => 10
```

Here, `console.log(...)` is a built-in tool that prints a value to the screen so
you can see it.

## Declaring a Variable

**Declaring** means creating the variable (making the box). In JavaScript you
declare a variable with one of three keywords: `let`, `const`, or `var`.

```js
let age = 25; // create a box named "age" with the value 25
const name = "Maya"; // create a box named "name" with the value "Maya"
var city = "Paris"; // the old way (we will avoid this)
```

A **keyword** is a special word that JavaScript already understands.

## Assignment

**Assignment** means putting a value into the box. The equals sign `=` is the
assignment operator. It does *not* mean "equal" like in math. It means
"take the value on the right and put it into the box on the left."

```js
let count = 5; // declare AND assign at the same time
count = 8; // assign a new value to the existing box
console.log(count); // => 8
```

You can also declare first and assign later:

```js
let message; // declare an empty box
console.log(message); // => undefined  (the box is empty)
message = "Hello"; // now put a value in it
console.log(message); // => Hello
```

When a box has nothing in it yet, JavaScript shows `undefined`. You will learn
more about `undefined` in the next lesson.

## The Difference Between `let`, `const`, and `var`

### `const` — a value that never changes

`const` means **constant**. Once you put a value in the box, you cannot put a
different value in it. If you try, you get an error.

```js
const pi = 3.14;
pi = 3.15; // => TypeError: Assignment to constant variable.
```

A `const` variable must get a value right away when you declare it:

```js
const greeting; // => SyntaxError: Missing initializer in const declaration
```

### `let` — a value that can change

`let` means you plan to change the value later. This is fine:

```js
let temperature = 20;
temperature = 25; // works perfectly
console.log(temperature); // => 25
```

### `var` — the old way

`var` is the oldest way to make a variable. It works, but it behaves in
confusing ways that cause bugs. Modern JavaScript uses `let` and `const` instead.

```js
var oldStyle = "please avoid me";
```

## Why Prefer `const`, Then `let`, and Avoid `var`

A simple rule to follow every time:

1. **Use `const` by default.** Most values never need to change. Using `const`
   tells anyone reading your code "this will stay the same." It also protects you
   from accidentally changing something.
2. **Use `let` only when you know the value will change** (for example, a counter
   that goes up).
3. **Avoid `var`.** It has tricky behavior with where it can be seen and when it
   exists. You do not need it.

Think of it like this: lock everything with `const` first, and only unlock a box
with `let` when you truly need to change what is inside.

## Naming Rules and Conventions

### Rules (you MUST follow these or the code breaks)

- A name can contain letters, numbers, `_` (underscore), and `$` (dollar sign).
- A name **cannot start with a number**.
- A name **cannot contain spaces**.
- A name **cannot be a keyword** like `let`, `const`, or `if`.

```js
let userName = "ok"; // good
let user_name = "ok"; // allowed
let $price = 10; // allowed
let 1stPlace = "bad"; // => SyntaxError (starts with a number)
let user name = "bad"; // => SyntaxError (has a space)
```

### Conventions (good habits, not enforced by JavaScript)

- Use **camelCase**: start with a lowercase letter, then capitalize the first
  letter of each new word. No spaces.
- Choose **clear, descriptive names**. `numberOfStudents` is much better than `n`.

```js
let firstName = "Sam"; // camelCase: easy to read
let totalPrice = 99; // clear meaning
let x = 99; // works, but tells the reader nothing
```

## Reassigning vs Redeclaring

These two words sound similar but mean different things.

- **Reassigning** = putting a new value into a box that already exists.
- **Redeclaring** = trying to create a box with a name that already exists.

```js
let speed = 10;
speed = 20; // reassigning: allowed with let
```

You cannot redeclare with `let` in the same area:

```js
let color = "red";
let color = "blue"; // => SyntaxError: 'color' has already been declared
```

(`var` lets you redeclare, which is one reason it causes bugs — you might
overwrite a variable by accident without noticing.)

## `const` With Objects and Arrays

This part surprises many beginners, so read it slowly.

When you use `const` with an **object** or an **array**, the rule is:
**the box is locked, but the contents inside can still change.**

An object or array is not stored directly in the box. Instead, the box holds a
*link* (called a reference) that points to the data. `const` locks the link, not
the data it points to.

Think of it like a house. `const` glues the house *address* onto the box. You can
never change the address. But you can still go inside the house and move the
furniture around.

```js
const user = { name: "Ana", age: 30 };
user.age = 31; // allowed: we changed the contents
console.log(user); // => { name: "Ana", age: 31 }

user = { name: "Bob" }; // => TypeError: changing the address is NOT allowed
```

Same idea with arrays:

```js
const colors = ["red", "green"];
colors.push("blue"); // allowed: add an item to the inside
console.log(colors); // => ["red", "green", "blue"]

colors = ["yellow"]; // => TypeError: cannot point the box somewhere new
```

So `const` does **not** mean "the value can never change." It means
"this name will always point to the same object or array."

## Common Mistakes

- **Using `=` and thinking it means "equals."** `=` puts a value into a box. To
  check if two things are equal, you use `===` (you will learn this soon).
- **Trying to reassign a `const`.** If a value needs to change, use `let`.
- **Forgetting to give `const` a value.** A `const` must get its value on the
  same line you declare it.
- **Expecting `const` to freeze an object.** It only locks the binding, not the
  contents inside the object or array.
- **Using unclear names** like `a`, `b`, `data2`. Future you will not remember
  what they mean.

## Exercises

1. Declare a `const` called `country` with the value of your country's name. Then
   try to reassign it to another country and read the error message you get.
2. Declare a `let` called `points` starting at `0`. Reassign it to `50`, then to
   `100`, printing the value after each change.
3. Create a `const` array called `fruits` with two fruits inside. Add a third
   fruit using `.push(...)`, then print the array. Explain to yourself why this is
   allowed even though `fruits` is a `const`.
4. Try declaring two variables with the same name using `let`. Read the error and
   write down in plain English what it means.

## Key Takeaways

- A variable is a named box that stores a value.
- Declare with `const`, `let`, or `var`; assign a value with `=`.
- `const` cannot be reassigned; `let` can; `var` is the old way to avoid.
- Prefer `const`, use `let` only when the value must change, and skip `var`.
- Follow naming rules, and use clear camelCase names.
- Reassigning changes the value in an existing box; redeclaring tries to make a
  new box with the same name (not allowed with `let`).
- With `const`, the binding is fixed but the contents of an object or array can
  still change.

---

[← Back to Course Index](../README.md) | [Next: Data Types →](02-data-types.md)
