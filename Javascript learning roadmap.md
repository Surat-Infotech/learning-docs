# JavaScript Learning Roadmap

**Technology:** JavaScript
**Experience Level:** Beginner
**Weekly Availability:** 30 hours
**Plan Length:** 8 weeks (~240 hours total)

---

## Concepts Checklist (Quick Reference)

### Phase 1 — Fundamentals

- [ ] How JavaScript runs (browser & Node) — you need to know where your code executes. **(Must-know)**
- [ ] Variables: `let`, `const`, `var` — the basic way to store data. **(Must-know)**
- [ ] Data types (string, number, boolean, null, undefined) — everything in JS is one of these. **(Must-know)**
- [ ] Operators & expressions — how to do math, compare, and combine values. **(Must-know)**
- [ ] Conditionals (`if`, `else`, `switch`) — make your code decide things. **(Must-know)**
- [ ] Loops (`for`, `while`, `for...of`) — repeat work without copy-pasting. **(Must-know)**
- [ ] Functions & parameters — reusable blocks of logic. **(Must-know)**
- [ ] Scope & hoisting — where variables live and why bugs happen. **(Must-know)**

### Phase 2 — Core

- [ ] Arrays & array methods (`map`, `filter`, `reduce`, `forEach`) — handle lists of data. **(Must-know)**
- [ ] Objects & object methods — model real-world things with key/value pairs. **(Must-know)**
- [ ] Strings & template literals — manipulate and build text. **(Must-know)**
- [ ] Arrow functions — shorter, modern function syntax. **(Must-know)**
- [ ] The DOM (selecting & changing elements) — make web pages interactive. **(Must-know)**
- [ ] Events & event listeners — respond to clicks, typing, etc. **(Must-know)**
- [ ] `this` keyword basics — understand context, a common confusion point. **(Should-know)**
- [ ] Error handling (`try`/`catch`) — handle things that go wrong gracefully. **(Should-know)**

### Phase 3 — Asynchronous & Modern JS

- [ ] Callbacks — the original async pattern; you'll see it everywhere. **(Should-know)**
- [ ] Promises — the modern way to handle async work. **(Must-know)**
- [ ] `async`/`await` — clean, readable async code. **(Must-know)**
- [ ] Fetch API & working with JSON — get data from servers/APIs. **(Must-know)**
- [ ] ES Modules (`import`/`export`) — split code into reusable files. **(Must-know)**
- [ ] Destructuring & spread/rest operators — write cleaner modern code. **(Should-know)**
- [ ] Closures — powerful concept asked about in interviews. **(Should-know)**

### Phase 4 — Professional Skills

- [ ] npm & package management — install and use external libraries. **(Must-know)**
- [ ] Git & GitHub basics — version control, expected on every team. **(Must-know)**
- [ ] Browser DevTools & debugging — find and fix bugs efficiently. **(Must-know)**
- [ ] LocalStorage — save data in the browser. **(Should-know)**
- [ ] Basic testing concepts — verify your code works. **(Should-know)**
- [ ] Clean code & naming conventions — write code others can read. **(Should-know)**
- [ ] A bit of TypeScript awareness — know what's coming next. **(Nice-to-have)**

---

## Learning Overview

**Estimated outcome:**
By the end of 8 weeks you will be able to build interactive web applications from scratch using vanilla JavaScript, fetch and display data from real APIs, structure code into modules, and confidently read and debug other people's JavaScript. You'll be ready to move into a framework like React.

**Key skills to be gained:**

- Writing clean, modern (ES6+) JavaScript
- Manipulating web pages with the DOM and handling user events
- Working with asynchronous code and real APIs
- Using Git, npm, and browser DevTools like a working developer
- Breaking problems into functions and modules

**Recommended learning approach:**

- Spend roughly **40% reading/watching** and **60% writing code**. You learn JS by typing it, not just reading it.
- Type out every example yourself — never copy-paste while learning.
- Build a small thing at the end of each week. Theory fades; projects stick.
- Use the browser console daily to test small ideas.
- When stuck, read the error message first, then search. Debugging is a core skill, not a failure.

---

## Roadmap

### Week 1 — Programming Fundamentals

Goal: Understand the building blocks of any program.

- [ ] Set up your environment (VS Code, browser, Node.js)
- [ ] Learn variables: `let`, `const`, and why to avoid `var`
- [ ] Learn data types and how to check them with `typeof`
- [ ] Practice operators and comparisons
- [ ] Master conditionals (`if`/`else`, `switch`)
- [ ] Master loops (`for`, `while`, `for...of`)
- **Exercise — Console Warm-Up Scripts**
  - **What to build:** Three tiny scripts that run in the browser console or Node.
    1. **FizzBuzz** — print numbers 1 to 100, but print "Fizz" for multiples of 3, "Buzz" for multiples of 5, and "FizzBuzz" for multiples of both.
    2. **Number checker** — given a number, log whether it is positive/negative/zero and even/odd.
    3. **Simple calculator** — a function that takes two numbers and an operator (`"+"`, `"-"`, `"*"`, `"/"`) and returns the result using `if` or `switch`.
  - **Requirements:** Use `const`/`let` correctly, use at least one loop and one conditional, and handle divide-by-zero in the calculator.
  - **Stretch goal:** Make the calculator reject invalid operators with a clear message.
  - **What you'll learn:** Translating plain-English logic into loops and conditionals — the foundation of everything else.
  - **Time:** ~3–4 hours.

### Week 2 — Functions, Arrays & Objects

Goal: Organize data and logic.

- [ ] Write functions with parameters and return values
- [ ] Understand scope and hoisting
- [ ] Learn arrays and core methods: `push`, `pop`, `map`, `filter`, `forEach`
- [ ] Learn objects: properties, methods, dot vs bracket access
- [ ] Combine arrays of objects (very common in real apps)
- **Exercise — To-Do List Engine (logic only, no UI)**
  - **What to build:** A set of functions that manage a list of task objects. Each task looks like `{ id: 1, text: "Buy milk", done: false }`. Store tasks in an array.
  - **Requirements (write each as a function):**
    - `addTask(text)` — adds a new task with a unique id and `done: false`.
    - `removeTask(id)` — removes a task by id (use `filter`).
    - `toggleTask(id)` — flips `done` true/false (use `map`).
    - `listTasks()` — prints all tasks nicely.
    - `countRemaining()` — returns how many tasks are not done.
  - **Test it by:** Calling the functions in order and logging the array after each step to confirm it changes correctly.
  - **Stretch goal:** Add `clearCompleted()` that removes all done tasks.
  - **What you'll learn:** Modeling real data with arrays of objects and transforming it with `map`/`filter` — the single most common pattern in real apps.
  - **Time:** ~5–6 hours.

### Week 3 — The DOM & Events

Goal: Make web pages come alive.

- [ ] Select elements (`querySelector`, `getElementById`)
- [ ] Change text, styles, and classes
- [ ] Create and remove elements dynamically
- [ ] Add event listeners (`click`, `input`, `submit`)
- [ ] Read values from forms and inputs
- **Mini-project — Interactive To-Do App**
  - **What to build:** A working to-do app in the browser. Reuse your Week 2 functions and connect them to a real interface.
  - **Requirements:**
    - An input box and an "Add" button to create tasks.
    - Tasks render as a list on the page (build each `<li>` dynamically).
    - Clicking a task toggles it done (add a line-through style via a CSS class).
    - Each task has a "Delete" button that removes it.
    - Show a live count of remaining tasks.
    - Submitting an empty input should do nothing (validate it).
  - **Tech:** One HTML file, one CSS file, one JS file. Use `querySelector`, `addEventListener`, and `createElement`.
  - **Stretch goals:** Add an "All / Active / Completed" filter; clear the input after adding.
  - **What you'll learn:** Connecting data (your array) to the screen (the DOM) and reacting to user actions — the core of front-end work.
  - **Time:** ~7–8 hours.

### Week 4 — Modern JavaScript (ES6+)

Goal: Write code the way professionals do today.

- [ ] Arrow functions
- [ ] Template literals
- [ ] Destructuring (arrays & objects)
- [ ] Spread and rest operators
- [ ] `reduce` and chaining array methods
- [ ] Closures (take your time here)
- **Exercise — Refactor To Modern JS**
  - **What to build:** Take your Week 3 to-do app and rewrite it using ES6+ features. The behavior stays the same; the code gets cleaner.
  - **Requirements:**
    - Convert all functions to **arrow functions** where appropriate.
    - Build every piece of HTML text using **template literals** instead of string concatenation.
    - Use **destructuring** when reading task properties (e.g. `const { id, text, done } = task`).
    - Use the **spread operator** to add tasks immutably (`[...tasks, newTask]`) instead of `push`.
    - Add a stats line using **`reduce`** (e.g. total tasks vs completed).
  - **Bonus concept:** Write one small example of a **closure** (e.g. a `makeIdGenerator()` function that returns a function giving the next id each call) and use it in the app.
  - **What you'll learn:** Writing code the way professional teams actually write it today, plus closures (a common interview topic).
  - **Time:** ~6 hours.

### Week 5 — Asynchronous JavaScript

Goal: Work with time and external data.

- [ ] Understand the call stack and event loop (high level)
- [ ] Callbacks and why they get messy
- [ ] Promises: `.then()`, `.catch()`
- [ ] `async`/`await`
- [ ] Error handling with `try`/`catch`
- **Exercise — Fake Data Loader**
  - **What to build:** Simulate loading data from a server before you touch real APIs, so you understand async on its own.
  - **Requirements:**
    1. Write a function `getUser(id)` that returns a **Promise**. Inside, use `setTimeout` (e.g. 1.5 seconds) to resolve with a fake user object like `{ id, name: "User " + id }`.
    2. Make it **reject** if the id is negative, with an error message.
    3. Call it first using `.then()` / `.catch()` and log the result.
    4. Then rewrite the same call using **`async`/`await`** inside a `try`/`catch` block.
    5. Show a "Loading…" message before the data arrives and replace it after.
  - **Stretch goal:** Load three users in sequence with `await`, then load them all at once with `Promise.all` and compare.
  - **What you'll learn:** Why async exists, how Promises work, and how `async`/`await` makes them readable — essential before fetching real data.
  - **Time:** ~6 hours.

### Week 6 — APIs & Fetching Data

Goal: Connect your app to the real world.

- [ ] Understand JSON
- [ ] Use the `fetch` API
- [ ] Handle loading and error states
- [ ] Display fetched data in the DOM
- [ ] Save data with LocalStorage
- **Mini-project — Search App With a Real API**
  - **What to build:** A small app that takes a search term, fetches real data from a free public API, and displays the results. Pick one:
    - **Movie search** using OMDb API (free key) or TMDB.
    - **Weather lookup** using Open-Meteo (no key needed) or OpenWeatherMap (free key).
    - **GitHub user search** using the public GitHub API (no key needed) — easiest to start.
  - **Requirements:**
    - An input + button (or live search as you type).
    - Use `fetch` with `async`/`await` to call the API.
    - Show a **"Loading…"** state while waiting.
    - Show a friendly **error message** if the request fails or returns nothing (wrap in `try`/`catch`).
    - Render results as cards in the DOM (image, title, and a couple of details).
    - Save the user's **last search** to LocalStorage and restore it on page reload.
  - **Stretch goals:** Add a "recent searches" list from LocalStorage; debounce live search so you don't call the API on every keystroke.
  - **What you'll learn:** The complete real-world data flow — request, wait, handle errors, display, and persist. This is what most front-end jobs do daily.
  - **Time:** ~8–10 hours.

### Week 7 — Tooling & Code Organization

Goal: Work like you're on a team.

- [ ] Git basics: `init`, `add`, `commit`, `push`, branches
- [ ] Push a project to GitHub
- [ ] npm: `init`, installing packages, `package.json`
- [ ] ES Modules: `import` / `export`
- [ ] Browser DevTools: breakpoints, console, network tab
- **Exercise — Modularize & Publish**
  - **What to build:** Take your Week 6 search app and turn it into a properly organized, version-controlled project on GitHub.
  - **Requirements:**
    - Run `npm init` to create a `package.json`.
    - Split your code into **ES Modules**, for example:
      - `api.js` — only the fetch/data functions (`export` them).
      - `ui.js` — only the DOM rendering functions.
      - `storage.js` — LocalStorage save/load helpers.
      - `main.js` — imports the others and wires everything together.
    - Initialize Git (`git init`), make small, meaningful commits as you go.
    - Create a repo on GitHub and `push` your code.
    - Write a **README.md** explaining what the app does, how to run it, and a screenshot.
    - Create one feature **branch**, make a change, and merge it back.
  - **Stretch goal:** Install and use one small npm package (e.g. a date formatter) to see how dependencies work.
  - **What you'll learn:** How real codebases are organized and shared on a team — modules, npm, and Git are expected on day one of any job.
  - **Time:** ~6–7 hours.

### Week 8 — Consolidation & Final Project

Goal: Prove you can build independently.

- [ ] Review weak areas from earlier weeks
- [ ] Learn basic clean-code and naming habits
- [ ] Read a short intro to testing and to TypeScript (awareness only)
- [ ] Build and ship the final project (see below)
- **Focus:** Working independently, debugging on your own, and shipping something complete.

---

## Projects & Milestones

### Suggested Mini-Projects (build as you go)

| Week | Project                    | Skills Practiced           |
| ---- | -------------------------- | -------------------------- |
| 2    | To-do list logic (no UI)   | Functions, arrays, objects |
| 3    | Interactive to-do app      | DOM, events                |
| 4    | Refactored to-do app       | ES6+ syntax                |
| 6    | Weather / movie search app | Fetch, APIs, async         |
| 7    | Modularized GitHub project | Git, npm, modules          |

### Final Project Recommendation — Expense Tracker

Build **one complete app** that combines everything you've learned. The recommended project is an **Expense Tracker**, but a Recipe Manager, Budget Planner, or Habit Tracker would work equally well.

**Core features (must have):**

- Add an expense with a description, amount, and category.
- Edit and delete existing expenses.
- Show a running **total** and a **breakdown by category** (use `reduce`).
- **Persist** all data in LocalStorage so it survives a page reload.
- Filter expenses by category or date.
- Validate input (no empty descriptions, no non-numeric amounts).

**API feature (must have):**

- Fetch **live currency exchange rates** from a free API (e.g. exchangerate.host — no key needed) and let the user view their total in a second currency. Handle loading and error states.

**Code & delivery requirements:**

- Organized into **ES Modules** (e.g. `storage.js`, `api.js`, `ui.js`, `calculations.js`, `main.js`).
- Clean ES6+ code with clear, consistent naming.
- A Git history with meaningful commits.
- Hosted on **GitHub** with a README (description, features, how to run, screenshot).
- **Bonus:** deploy it free on GitHub Pages or Netlify so you have a live link to share.

**Stretch goals:** simple chart of spending by category; export data as JSON; dark mode saved to LocalStorage.

**Suggested timeline within Week 8:** 1 day planning + UI sketch, 2 days core features, 1 day API + persistence, 1 day cleanup, README, and deploy.

**Time:** ~20–25 hours.

### Readiness Checkpoints

- [ ] **End of Week 2:** Can write functions and manipulate arrays/objects without help.
- [ ] **End of Week 4:** Can build a small interactive page and read modern JS comfortably.
- [ ] **End of Week 6:** Can fetch data from an API and display it.
- [ ] **End of Week 8:** Can plan, build, debug, and publish a full app independently.

---

## Success Criteria

By the end of this roadmap, you should be able to **independently**:

**Build:**

- Interactive web pages that respond to user input
- Apps that fetch, display, and store real data
- A complete multi-file project hosted on GitHub

**Explain:**

- The difference between `let`, `const`, and `var`, and when to use each
- How the DOM works and how events are handled
- What Promises and `async`/`await` do and why async code matters
- What a closure is and a basic example of one

**Implement:**

- Array transformations using `map`, `filter`, and `reduce`
- API calls with proper loading and error handling
- Clean, readable code split into reusable functions and modules
- A Git workflow: commit, branch, and push to GitHub

If you can do the above without copying from a tutorial, you're ready to start learning a framework like **React**.
