# ES Modules: `import` / `export`

When your program grows, putting all your code in one giant file becomes a
nightmare. You scroll forever, you lose track of things, and small changes feel
scary. **Modules** are the fix. A module is just a file. ES Modules let one file
share its code with another file in a clean, controlled way.

Think of modules like rooms in a house. The kitchen has the cooking tools. The
garage has the car tools. You do not throw everything into one big pile in the
living room. Each room has a job, and you walk between rooms to get what you need.

## What You'll Learn

- Why we split code into separate files
- **Named exports** and how to import them
- **Default exports** and how to import them
- Renaming imports with `as`
- Importing everything with `* as`
- Module scope (each file is its own private world)
- Using modules in the **browser** and in **Node.js**
- A small multi-file example you can build
- A quick mention of the older **CommonJS** `require`

---

## Why Split Code Into Files?

Imagine writing a 3,000-line file. Some problems show up fast:

- It is hard to find anything.
- Two people cannot easily work on it at once.
- Reusing one small function in another project means copying lots of junk.
- A tiny typo can break the whole thing.

Splitting code into small files gives each file **one clear job**. You can reuse
files, test them on their own, and read them without getting lost.

The rule of modules is simple: a file keeps everything private **unless** it
chooses to `export` it. Another file can then `import` what was exported.

---

## Named Exports

A **named export** shares a value under a specific name. You can have many named
exports in one file.

```js
// file: math.js

// Export each value with the `export` keyword.
export const PI = 3.14159;

export function add(a, b) {
  return a + b;
}

export function subtract(a, b) {
  return a - b;
}
```

To use them, you **import by name** using curly braces `{ }`:

```js
// file: main.js
import { add, subtract, PI } from "./math.js";

console.log(add(2, 3));      // => 5
console.log(subtract(10, 4)); // => 6
console.log(PI);             // => 3.14159
```

The names inside `{ }` must match the exported names exactly. The `"./math.js"`
part is the **path** to the file. The `./` means "in the same folder."

> Tip: You can also write one export list at the bottom of a file:
> `export { add, subtract, PI };`

---

## Default Export

A file can have **one default export**. Use it when the file is really "about"
one main thing.

```js
// file: greeter.js
export default function greet(name) {
  return "Hello, " + name + "!";
}
```

When importing a default export, you do **not** use curly braces. And you can
pick **any name** you want for it:

```js
// file: main.js
import greet from "./greeter.js"; // no curly braces

console.log(greet("Sam")); // => Hello, Sam!
```

You can mix one default export with named exports in the same file:

```js
// file: user.js
export default class User {}      // the main thing
export const MAX_USERS = 100;     // an extra named export
```

```js
import User, { MAX_USERS } from "./user.js";
```

---

## Renaming Imports With `as`

Sometimes a name clashes with one you already use, or the name is unclear. Use
`as` to rename it while importing:

```js
import { add as addNumbers } from "./math.js";

console.log(addNumbers(1, 1)); // => 2
```

You can also rename while exporting:

```js
function internalAdd(a, b) {
  return a + b;
}
export { internalAdd as add };
```

---

## Importing Everything With `* as`

If you want all the named exports of a file bundled into one object, use
`* as someName`:

```js
import * as math from "./math.js";

console.log(math.add(2, 2)); // => 4
console.log(math.PI);        // => 3.14159
```

Here `math` is an object, and each export becomes a property on it. This is handy
when a file has many exports and you want to keep them grouped.

---

## Module Scope: Each File Is Its Own World

This is a key idea. **Variables inside a module are private to that module.**
They do not leak out to other files, and they do not pollute the global space.

```js
// file: secret.js
const password = "12345"; // private to this file
export const hint = "It is very common.";
```

```js
// file: main.js
import { hint } from "./secret.js";

console.log(hint);     // => It is very common.
console.log(password); // => ReferenceError: password is not defined
```

You can only reach what was exported. Everything else stays hidden. This makes
modules safe and predictable.

---

## Using Modules in the Browser

In a web page, tell the browser a script is a module with
`type="module"`:

```html
<!-- index.html -->
<script type="module" src="./main.js"></script>
```

Or write the module code inline:

```html
<script type="module">
  import { add } from "./math.js";
  console.log(add(5, 5)); // => 10
</script>
```

Two things to know:

- Module files must usually be served from a web server (like a local dev
  server), not opened directly as `file://`, or the browser may block them.
- Module code automatically runs in **strict mode** and is **deferred** (it runs
  after the HTML is parsed).

---

## Using Modules in Node.js

Node supports ES Modules too. The easiest way is to either:

- Name your files with the `.mjs` extension, **or**
- Add `"type": "module"` to your project's `package.json`.

```js
// file: main.mjs
import { add } from "./math.mjs";
console.log(add(3, 4)); // => 7
```

Run it with:

```bash
node main.mjs
```

---

## A Small Multi-File Example

Let's build a tiny calculator across two files.

```js
// file: math.js
export function add(a, b) {
  return a + b;
}

export function multiply(a, b) {
  return a * b;
}

export const TAX_RATE = 0.1;
```

```js
// file: main.js
import { add, multiply, TAX_RATE } from "./math.js";

const subtotal = add(20, 30);          // 50
const tax = multiply(subtotal, TAX_RATE); // 5
const total = add(subtotal, tax);      // 55

console.log("Subtotal:", subtotal); // => Subtotal: 50
console.log("Tax:", tax);           // => Tax: 5
console.log("Total:", total);       // => Total: 55
```

`math.js` does the math. `main.js` is the boss that puts it together. Each file
is short and easy to read.

---

## A Quick Word on CommonJS (`require`)

Before ES Modules, Node used a different system called **CommonJS**. You may see
it in older code or tutorials. It looks like this:

```js
// old CommonJS style
const math = require("./math.js");
module.exports = { add, subtract };
```

It does a similar job, but the syntax is different. For new code, prefer ES
Modules (`import` / `export`). Just recognize `require` and `module.exports` when
you see them in older projects.

---

## Common Mistakes

- **Forgetting the file extension in the browser.** Browsers need the full path,
  including `.js`: `from "./math.js"`, not `from "./math"`.
- **Curly braces on a default import.** Named imports use `{ }`; default imports
  do not. `import greet from "./greeter.js"` is correct.
- **Misspelled export names.** `import { ad }` when the export is `add` fails.
  The names must match exactly.
- **Opening HTML as a file.** If modules silently fail in the browser, you likely
  need a local server instead of double-clicking the file.
- **Expecting private variables to be importable.** If you did not `export` it,
  you cannot `import` it. That is by design.

---

## Exercises

1. Create a file `strings.js` that exports two named functions: `shout(text)`
   (returns the text in uppercase) and `whisper(text)` (returns it in lowercase).
   Import both into a `main.js` and test them.

2. Add a **default export** to `strings.js`: a function `reverse(text)` that
   reverses a string. Import it without curly braces and try it out.

3. In `main.js`, import everything from `strings.js` using `* as text`. Call the
   functions through the `text` object.

4. Make a private constant in `strings.js` that you do **not** export. Try to
   import it in `main.js` and observe the error. Explain in a comment why it fails.

---

## Key Takeaways

- A **module is a file**. It hides everything unless it `export`s it.
- **Named exports** use `{ }` and must match names; **default exports** do not use
  `{ }` and can be renamed freely.
- Use `as` to rename, and `* as name` to grab everything as one object.
- Each module has its **own private scope** — no accidental leaks.
- In the browser use `<script type="module">`; in Node use `.mjs` or
  `"type": "module"`.
- The older **CommonJS** style (`require` / `module.exports`) still exists in
  older code, but prefer ES Modules going forward.

---

[← Back to Course Index](../README.md) | [Next: Objects, Classes, and Prototypes (OOP) →](./02-oop-classes-prototypes.md)
