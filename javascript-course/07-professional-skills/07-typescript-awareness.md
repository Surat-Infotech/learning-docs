# A Taste of TypeScript

As you grow as a developer, you'll keep hearing about **TypeScript**. It's everywhere in modern projects. You don't need to learn it right now, but it's good to know what it is and why people love it. This lesson is a light, friendly introduction. Just awareness, no pressure.

## What You'll Learn

- What TypeScript is
- Why developers use it
- Basic type annotations
- A first look at interfaces and types
- How to add types to function parameters and return values
- That TypeScript turns into regular JavaScript
- That it's okay to learn this later

## What Is TypeScript?

**TypeScript** is JavaScript with **types** added. That's the short version.

A **type** describes what kind of value something is, such as text, a number, or true/false. In plain JavaScript, a variable can hold anything, and JavaScript won't warn you if you make a mistake. TypeScript lets you say "this variable should always be a number," and it checks that for you.

So: **TypeScript = JavaScript + types**. All valid JavaScript is already valid TypeScript, so you're not starting from scratch.

## Why Do People Use It?

TypeScript has two big benefits:

1. **Catch errors early.** It spots mistakes *while you write code*, before you even run it. For example, it warns you if you try to do math on a piece of text.
2. **Better autocomplete.** Because your editor knows the types, it can suggest exactly what you can do with each value. This makes coding faster and reduces guessing.

Here's a quick taste of the difference. This bug would silently break in JavaScript:

```js
// JavaScript: no warning, but this is a mistake
let age = "25";        // age is text, not a number
let nextYear = age + 1; // "251" instead of 26 — oops!
```

```ts
// TypeScript: catches the problem before you run it
let age: number = "25"; // Error! "25" is text, not a number
```

TypeScript points at the problem right away, saving you from a confusing bug later.

## Basic Type Annotations

A **type annotation** is how you tell TypeScript what type something should be. You write a colon and the type after the variable name.

```ts
// Text values use "string"
let firstName: string = "Ava";

// Number values use "number"
let age: number = 25;

// True/false values use "boolean"
let isStudent: boolean = true;

// An array of numbers
let scores: number[] = [90, 85, 100];

// An array of strings
let names: string[] = ["Ava", "Ben"];
```

The part after the colon (`: string`, `: number`) is the type. TypeScript now makes sure you only put the right kind of value there.

## Describing Objects with Interfaces and Types

For objects, you can describe their shape using an **interface** or a **type**. Both let you say what properties an object must have. Here's an interface:

```ts
// Describe what a "User" object looks like
interface User {
  name: string;
  age: number;
  isMember: boolean;
}

// This object must match the shape above
let user: User = {
  name: "Ava",
  age: 25,
  isMember: true,
};
```

You can do the same thing with a `type`:

```ts
// Another way to describe an object's shape
type Product = {
  title: string;
  price: number;
};

let item: Product = {
  title: "Notebook",
  price: 5,
};
```

For beginners, `interface` and `type` are very similar. Use either one. Now if you forget a property or use the wrong kind of value, TypeScript tells you.

## Function Parameter and Return Types

You can also add types to function inputs (parameters) and outputs (the return value).

```ts
// (a: number, b: number) means both inputs must be numbers
// : number after the parentheses means it returns a number
function add(a: number, b: number): number {
  return a + b;
}

add(2, 3);     // works fine, returns 5
add(2, "3");   // Error! "3" is text, not a number
```

This is powerful. TypeScript makes sure you call your functions correctly. If a function expects a number and you pass text, you'll know immediately.

## TypeScript Compiles to JavaScript

Browsers and Node don't understand TypeScript directly. So TypeScript gets **compiled** (converted) into plain JavaScript before it runs. This step simply removes the types and produces normal JavaScript.

```ts
// TypeScript that you write
let message: string = "Hello";
```

```js
// The JavaScript it becomes (types are removed)
let message = "Hello";
```

So the types are like helpful labels that guide you while writing. Once compiled, you get ordinary JavaScript that runs anywhere.

## You Don't Need It Yet

Here's the reassuring part: **you don't need TypeScript right now.** Focus on getting comfortable with JavaScript first. Everything you learn in JavaScript carries straight over.

But TypeScript is very common in real jobs and modern projects. So when you feel solid with JavaScript, learning TypeScript will be a natural and rewarding next step. For now, it's enough to know it exists and what it does.

## Common Mistakes

- **Thinking TypeScript is a totally different language.** It's just JavaScript with types.
- **Rushing into it too early.** Get comfortable with JavaScript first.
- **Forgetting it must be compiled.** The browser runs the JavaScript output, not the TypeScript.
- **Overcomplicating types at first.** Start with the simple ones: string, number, boolean.

## Exercises

1. Write a TypeScript variable for your name (a `string`) and your age (a `number`).
2. Create an `interface` called `Book` with a `title` (string) and `pages` (number), then make an object that matches it.
3. Write a function `greet(name: string): string` that returns a greeting message.
4. In words, explain what happens to the types when TypeScript is compiled to JavaScript.

## Key Takeaways

- **TypeScript** is JavaScript plus **types**.
- It helps you **catch errors early** and gives **better autocomplete**.
- Add types with a colon, like `let age: number = 25`.
- Use **interfaces** or **types** to describe the shape of objects.
- You can type function **parameters** and **return values**.
- TypeScript **compiles to plain JavaScript** before it runs.
- You don't need it yet, but it's a great next step after JavaScript.

---

[← Back to Clean Code and Naming](./06-clean-code.md) | [Back to Course Index](../README.md)
