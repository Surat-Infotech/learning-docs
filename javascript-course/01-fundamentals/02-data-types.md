# Data Types

Data comes in different kinds. A piece of text is different from a number, and a
true/false answer is different from both. In JavaScript, the kind of a value is
called its **data type**. Knowing data types helps you understand what you can do
with a value and avoid surprising bugs.

## What You'll Learn

- The seven **primitive** types in JavaScript
- The five you will use most: string, number, boolean, null, undefined
- A short mention of the rare two: bigint and symbol
- What **objects** and **arrays** are (non-primitive types)
- How to check a type with `typeof`
- The difference between `null` and `undefined`
- The basics of numbers, strings, and booleans

## Two Big Groups of Data

JavaScript values fall into two groups:

1. **Primitive types** — simple, single values (text, a number, true/false, etc.).
2. **Non-primitive types** — collections of values, mainly **objects** and
   **arrays**.

There are **seven primitive types**:

- `string`
- `number`
- `boolean`
- `null`
- `undefined`
- `bigint`
- `symbol`

We will focus on the first five. The last two are rare for beginners, so we only
mention them briefly.

## Strings

A **string** is text. You write a string by wrapping characters in quotes. You
can use single quotes `'...'`, double quotes `"..."`, or backticks `` `...` ``.

```js
let firstName = "Maya";
let greeting = 'Hello';
let city = `Paris`;
```

Backticks are special: they let you put variables inside the text using
`${...}`. This is called a **template literal**.

```js
let name = "Sam";
let message = `Hi, ${name}!`;
console.log(message); // => Hi, Sam!
```

You can join strings together (you will see more of this in the operators lesson):

```js
let a = "foot";
let b = "ball";
console.log(a + b); // => football
```

A string has a `.length`, which is how many characters it contains:

```js
console.log("hello".length); // => 5
```

## Numbers

A **number** in JavaScript covers both whole numbers (integers) and decimals.
There is no separate "decimal type" — it is all just `number`.

```js
let age = 30; // integer
let price = 19.99; // decimal
let negative = -7; // negative number
```

### Special number values

Sometimes math gives you a strange result. JavaScript has two special numbers:

- **`NaN`** means "Not a Number." You get it when a math operation makes no sense.
- **`Infinity`** means a number too large to represent, like dividing by zero.

```js
console.log(10 / 0); // => Infinity
console.log("hello" * 2); // => NaN  (text times a number makes no sense)
```

Oddly, `NaN` is still of type `number`:

```js
console.log(typeof NaN); // => "number"
```

## Booleans

A **boolean** has only two possible values: `true` or `false`. Booleans answer
yes/no questions and are the heart of decision-making in code.

```js
let isLoggedIn = true;
let hasPaid = false;
console.log(5 > 3); // => true
console.log(2 > 9); // => false
```

## `null` and `undefined`

These two both mean "no value," but they are used in different ways.

- **`undefined`** means "this has not been given a value yet." JavaScript usually
  sets this automatically.
- **`null`** means "intentionally empty." A programmer sets this on purpose to say
  "there is nothing here right now."

A simple way to remember: `undefined` is JavaScript saying "I have nothing,"
while `null` is *you* saying "I am putting nothing here on purpose."

```js
let notSetYet; // declared but no value
console.log(notSetYet); // => undefined

let chosenColor = null; // we deliberately set it to "empty"
console.log(chosenColor); // => null
```

## Objects and Arrays (Non-Primitive)

When you need to group many values together, you use an **object** or an
**array**.

An **object** stores values as named pairs (a key and a value):

```js
let person = {
  name: "Ana",
  age: 28,
  isStudent: true,
};
console.log(person.name); // => Ana
```

An **array** stores a list of values in order, reached by their position
(starting at 0):

```js
let colors = ["red", "green", "blue"];
console.log(colors[0]); // => red
console.log(colors[2]); // => blue
```

You will study objects and arrays in depth in later modules. For now, just know
they are the main non-primitive types.

## Checking the Type With `typeof`

The `typeof` operator tells you the type of a value. It returns the type as a
string. This is very useful when you are unsure what a value is.

```js
console.log(typeof "hello"); // => "string"
console.log(typeof 42); // => "number"
console.log(typeof true); // => "boolean"
console.log(typeof undefined); // => "undefined"
console.log(typeof { a: 1 }); // => "object"
console.log(typeof [1, 2, 3]); // => "object"  (arrays report as "object")
```

### Two `typeof` quirks to know

```js
console.log(typeof null); // => "object"   (a famous old bug in JavaScript)
console.log(typeof function () {}); // => "function"
```

`typeof null` says `"object"` even though `null` is a primitive. This is a
historical mistake that was never fixed. Just remember it.

## The Rare Two: bigint and symbol

You will almost never need these as a beginner, but here is a one-line idea of
each so the names are not a mystery.

- **`bigint`** is for whole numbers that are too large for the normal `number`
  type. You write it with an `n` at the end.
- **`symbol`** creates a unique, hidden identifier, often used as special object
  keys in advanced code.

```js
let huge = 9007199254740993n; // bigint
console.log(typeof huge); // => "bigint"

let id = Symbol("id"); // symbol
console.log(typeof id); // => "symbol"
```

Do not worry if these feel abstract. You can come back to them much later.

## Common Mistakes

- **Thinking arrays are their own type.** `typeof [1, 2]` is `"object"`, not
  `"array"`. (There is another way to check for arrays you will learn later.)
- **Confusing `null` and `undefined`.** Use `undefined` to mean "not set yet" and
  `null` to mean "intentionally empty."
- **Expecting `typeof null` to be `"null"`.** It is `"object"` due to an old bug.
- **Treating `NaN` as text.** `NaN` is of type `number`, even though it means
  "Not a Number."
- **Mixing quote styles in one string.** Start and end a string with the same
  type of quote.

## Exercises

1. Create one variable of each of these types: string, number, boolean, `null`,
   and `undefined`. Print the `typeof` each one and check the output.
2. Make an object describing a book (with `title`, `pages`, and `inStock`). Print
   the type of the whole object and the type of each value inside it.
3. Try `typeof null` and `typeof [1, 2, 3]`. Write down the results and why they
   might surprise a beginner.
4. Use a template literal (backticks) to print a sentence that includes a name
   variable and an age variable.

## Key Takeaways

- JavaScript has seven primitive types; you will mostly use string, number,
  boolean, null, and undefined.
- Objects and arrays are non-primitive types for grouping many values.
- `typeof` returns the type of a value as a string.
- `undefined` means "not set yet"; `null` means "intentionally empty."
- Numbers include integers and decimals, plus special values `NaN` and `Infinity`.
- `typeof null` is `"object"` and arrays also report as `"object"` — both are
  quirks worth remembering.

---

[← Back to Course Index](../README.md) | [Next: Operators and Expressions →](03-operators-and-expressions.md)
