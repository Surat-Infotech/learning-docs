# JavaScript Course — 8-Week Weekly Roadmap (30 hours/week)

This roadmap turns the [JavaScript Course](README.md) into a clear weekly plan.

- **Time budget:** 30 hours per week
- **Total length:** 8 weeks (~240 hours)
- **Split:** About **40% reading/watching** and **60% writing code** (so ~12 hours
  reading and ~18 hours coding each week).

> **How to read this plan.** Each week lists the lessons to study and a small
> project to build at the end. Type every example yourself, do the lesson
> exercises, then build the weekly project. If a week feels heavy, it is fine to
> slow down — understanding beats speed.

**Weekly rhythm (suggested):** 5 study days × ~5 hours, or 6 days × 5 hours.
Keep one short day for review and rest.

---

## Week-at-a-Glance

| Week | Focus                | Lessons                         | Weekly Project                  |
| ---- | -------------------- | ------------------------------- | ------------------------------- |
| 1    | Setup + Fundamentals | Module 0 + Module 1             | Console Warm-Up Scripts         |
| 2    | Functions & Scope    | Module 2                        | Mini Function Library           |
| 3    | Data Structures      | Module 3                        | To-Do List Engine (logic only)  |
| 4    | DOM & Events         | Module 4                        | Interactive To-Do App           |
| 5    | Async Foundations    | Module 5 (lessons 1–4)          | Fake Data Loader                |
| 6    | APIs & Data          | Module 5 (lessons 5–6) + review | Search App with a Real API      |
| 7    | Advanced JavaScript  | Module 6                        | Refactor + Modularize           |
| 8    | Pro Skills + Final   | Module 7                        | Final Project (Expense Tracker) |

---

## Week 1 — Setup and Fundamentals

**Goal:** Get your tools ready and learn the basic building blocks of code.

**Lessons (Module 0 + Module 1):**

- [How JavaScript Works](00-getting-started/01-how-javascript-works.md)
- [Setup and Running Your First Code](00-getting-started/02-setup-and-running-code.md)
- [Variables: `let`, `const`, `var`](01-fundamentals/01-variables.md)
- [Data Types](01-fundamentals/02-data-types.md)
- [Operators and Expressions](01-fundamentals/03-operators-and-expressions.md)
- [Type Conversion and Coercion](01-fundamentals/04-type-conversion-and-coercion.md)
- [Conditionals (`if`, `else`, `switch`)](01-fundamentals/05-conditionals.md)
- [Loops (`for`, `while`, `for...of`)](01-fundamentals/06-loops.md)

**Weekly project — Console Warm-Up Scripts** (~3–4 hours, rest of day 6 for review):

1. **FizzBuzz** — print 1 to 100, but "Fizz" for multiples of 3, "Buzz" for
   multiples of 5, "FizzBuzz" for both.
2. **Number checker** — log whether a number is positive/negative/zero and even/odd.
3. **Simple calculator** — a function taking two numbers and an operator
   (`+ - * /`) returning the result; handle divide-by-zero.

✅ **End-of-week check:** You can use `const`/`let` correctly and translate plain
English logic into loops and conditionals.

---

## Week 2 — Functions and Scope

**Goal:** Write reusable logic and understand where variables live.

**Lessons (Module 2):**

- [Functions](02-functions-and-scope/01-functions.md)
- [Arrow Functions](02-functions-and-scope/02-arrow-functions.md)
- [Scope and Hoisting](02-functions-and-scope/03-scope-and-hoisting.md)
- [Closures](02-functions-and-scope/04-closures.md)
- [The `this` Keyword](02-functions-and-scope/05-the-this-keyword.md)

**Weekly project — Mini Function Library** (~3 hours):

Write a small file of useful, reusable functions, each tested in the console:

- `add`, `subtract`, `multiply`, `divide` (arrow functions)
- `isEven(n)`, `max(a, b)`, `clamp(value, min, max)`
- `makeCounter()` — a **closure** that returns a function giving the next number each call.
- `greet(name = "friend")` — uses a default parameter.

✅ **End-of-week check:** You can write functions confidently and explain what a
closure is with a simple example.

---

## Week 3 — Data Structures

**Goal:** Store and transform collections of data.

**Lessons (Module 3):**

- [Arrays](03-data-structures/01-arrays.md)
- [Array Methods (`map`, `filter`, `reduce`)](03-data-structures/02-array-methods.md)
- [Objects](03-data-structures/03-objects.md)
- [Strings and Template Literals](03-data-structures/04-strings.md)
- [Sets and Maps](03-data-structures/05-sets-and-maps.md)
- [Destructuring, Spread, and Rest](03-data-structures/06-destructuring-spread-rest.md)

**Weekly project — To-Do List Engine (logic only, no UI)** (~5 hours):

Each task looks like `{ id: 1, text: "Buy milk", done: false }`, stored in an array.

- `addTask(text)` — adds a task with a unique id and `done: false`.
- `removeTask(id)` — removes a task (use `filter`).
- `toggleTask(id)` — flips `done` (use `map`).
- `listTasks()` — prints all tasks nicely.
- `countRemaining()` — returns how many tasks are not done.
- **Stretch:** `clearCompleted()` removes all done tasks.

✅ **End-of-week check:** You can model data with arrays of objects and transform
it with `map`, `filter`, and `reduce`.

---

## Week 4 — The DOM and Events

**Goal:** Make web pages interactive.

**Lessons (Module 4):**

- [The DOM](04-dom-and-events/01-the-dom.md)
- [Changing the Page (DOM Manipulation)](04-dom-and-events/02-manipulating-the-dom.md)
- [Events and Event Listeners](04-dom-and-events/03-events.md)
- [Forms and User Input](04-dom-and-events/04-forms-and-inputs.md)

**Weekly project — Interactive To-Do App** (~8 hours):

Connect your Week 3 to-do logic to a real page.

- Input box + "Add" button to create tasks.
- Tasks render as a list (build each `<li>` dynamically).
- Clicking a task toggles it done (line-through via a CSS class).
- Each task has a "Delete" button.
- Show a live count of remaining tasks.
- Empty input does nothing (validate it).
- **Stretch:** All / Active / Completed filter.

✅ **End-of-week check:** You can connect your data (an array) to the screen (the
DOM) and respond to user actions.

---

## Week 5 — Asynchronous Foundations

**Goal:** Understand how JavaScript handles things that take time.

**Lessons (Module 5, lessons 1–4):**

- [The Call Stack and Event Loop](05-asynchronous-javascript/01-call-stack-and-event-loop.md)
- [Callbacks](05-asynchronous-javascript/02-callbacks.md)
- [Promises](05-asynchronous-javascript/03-promises.md)
- [`async` / `await`](05-asynchronous-javascript/04-async-await.md)

**Weekly project — Fake Data Loader** (~5 hours):

1. `getUser(id)` returns a **Promise** using `setTimeout` (~1.5s) to resolve with
   `{ id, name: "User " + id }`.
2. **Reject** if the id is negative.
3. Call it with `.then()` / `.catch()`.
4. Rewrite the call with **`async`/`await`** inside `try`/`catch`.
5. Show "Loading…" before the data arrives.
6. **Stretch:** Load three users in sequence with `await`, then all at once with `Promise.all`.

✅ **End-of-week check:** You understand why async exists and how `async`/`await`
makes Promises readable.

---

## Week 6 — APIs and Real Data

**Goal:** Connect your app to the real world.

**Lessons (Module 5, lessons 5–6) + review:**

- [Fetch and JSON](05-asynchronous-javascript/05-fetch-and-json.md)
- [Error Handling](05-asynchronous-javascript/06-error-handling.md)
- Review: [Array Methods](03-data-structures/02-array-methods.md) and
  [DOM Manipulation](04-dom-and-events/02-manipulating-the-dom.md) (you will use both).

**Weekly project — Search App with a Real API** (~10 hours):

Pick one: GitHub user search (easiest, no key), movie search (OMDb/TMDB), or
weather (Open-Meteo, no key).

- Input + button (or live search as you type).
- Use `fetch` with `async`/`await`.
- Show a **"Loading…"** state while waiting.
- Show a friendly **error message** on failure (`try`/`catch`).
- Render results as cards (image, title, a couple of details).
- **Stretch:** Save the last search to LocalStorage; debounce live search.

✅ **End-of-week check:** You can fetch data from an API, handle loading/errors,
and display the results.

---

## Week 7 — Advanced JavaScript

**Goal:** Learn the concepts that take you from beginner to intermediate.

**Lessons (Module 6):**

- [ES Modules (`import` / `export`)](06-advanced-javascript/01-es-modules.md)
- [Objects, Classes, and Prototypes (OOP)](06-advanced-javascript/02-oop-classes-prototypes.md)
- [Iterators and Generators](06-advanced-javascript/03-iterators-and-generators.md)
- [Regular Expressions](06-advanced-javascript/04-regular-expressions.md)
- [Functional Programming Ideas](06-advanced-javascript/05-functional-programming.md)
- [Useful Advanced Patterns](06-advanced-javascript/06-advanced-patterns.md)

**Weekly project — Refactor and Modularize** (~4 hours):

Take your Week 6 search app and improve it:

- Split it into **ES Modules**: `api.js`, `ui.js`, `storage.js`, `main.js`.
- Add a `debounce` helper (from Advanced Patterns) to your live search.
- Use **optional chaining** (`?.`) when reading API fields that might be missing.
- **Stretch:** Model a result as a small **class** with a method (e.g. `User.displayName()`).

✅ **End-of-week check:** You can split code into modules and read advanced JS
(classes, prototypes, generators) without fear.

---

## Week 8 — Professional Skills and Final Project

**Goal:** Work like a real developer and ship something complete.

**Lessons (Module 7):**

- [npm and Package Management](07-professional-skills/01-npm-and-packages.md)
- [Git and GitHub Basics](07-professional-skills/02-git-and-github.md)
- [Browser DevTools and Debugging](07-professional-skills/03-devtools-and-debugging.md)
- [LocalStorage](07-professional-skills/04-localstorage.md)
- [Testing Basics](07-professional-skills/05-testing.md)
- [Clean Code and Naming](07-professional-skills/06-clean-code.md)
- [A Taste of TypeScript](07-professional-skills/07-typescript-awareness.md)

**Final project — Expense Tracker** (~17–20 hours):

Build one complete app that combines everything.

- Add an expense with description, amount, and category.
- Edit and delete expenses.
- Show a running **total** and a **breakdown by category** (use `reduce`).
- **Persist** all data in LocalStorage so it survives reload.
- Filter by category or date; validate input.
- **API feature:** fetch live currency rates (e.g. exchangerate.host) and show
  the total in a second currency, with loading/error states.
- **Delivery:** organize into ES Modules, write clean code, commit to Git, push
  to GitHub with a README, and (bonus) deploy on GitHub Pages or Netlify.

✅ **End-of-week check:** You can plan, build, debug, and publish a full app on
your own.

---

## Readiness Checkpoints

- [ ] **End of Week 2:** Write functions and use closures without help.
- [ ] **End of Week 3:** Transform arrays/objects with `map`/`filter`/`reduce`.
- [ ] **End of Week 4:** Build a small interactive page.
- [ ] **End of Week 6:** Fetch and display data from a real API.
- [ ] **End of Week 8:** Plan, build, debug, and publish a full app independently.

When you can do all of the above without copying from a tutorial, you are ready
to learn a framework like **React**.

---

[← Back to Course Index](README.md)
