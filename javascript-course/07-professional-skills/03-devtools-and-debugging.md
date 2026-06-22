# Browser DevTools and Debugging

Every web browser has a built-in set of tools for developers. They let you peek inside your web page, see errors, and find out why something is broken. These are called **DevTools**. Learning them turns confusing bugs into solvable problems. Don't worry, bugs happen to everyone, even experts.

## What You'll Learn

- How to open DevTools
- How to use the Console for logging and reading errors
- Useful console methods
- What the Elements tab shows
- How to watch network requests
- How to set breakpoints and step through code
- The `debugger` statement
- How to read a stack trace
- A calm, simple debugging strategy

## Opening DevTools

To open DevTools in most browsers:

- Press **F12**, or
- Right-click anywhere on a page and choose **Inspect**, or
- Use the keyboard shortcut: **Ctrl + Shift + I** (Windows) or **Cmd + Option + I** (Mac).

A panel will appear with several tabs. Let's go through the most useful ones.

## The Console

The **Console** is where JavaScript talks to you. It shows messages you print and any errors the browser finds.

You print messages with `console.log`:

```js
// Print a message to the Console
console.log("Hello from my code");

// Print the value of a variable to check it
let total = 42;
console.log("The total is:", total);
```

When something goes wrong, errors appear here in red. **Always read the Console first** when something doesn't work. The error message usually tells you what and where the problem is.

You can also type JavaScript directly into the Console and press Enter to test things quickly.

## Useful Console Methods

`console.log` is the most common, but there are more helpful ones:

```js
// Normal message
console.log("Starting the app");

// Show data as a neat table (great for arrays of objects)
const users = [
  { name: "Ava", age: 25 },
  { name: "Ben", age: 30 },
];
console.table(users);

// Show an error message (appears in red)
console.error("Something went wrong!");

// Show a warning message (appears in yellow)
console.warn("This feature is old and may be removed");

// Show all properties of an object in an expandable view
console.dir(document.body);
```

`console.table` is especially nice for seeing lists of data clearly.

## The Elements Tab

The **Elements** tab shows the HTML and CSS of your page. You can:

- See the structure of the page (the HTML).
- Click any element to highlight it on the page.
- Change text and styles temporarily to test ideas.

Changes you make here are *temporary*. They disappear when you reload the page. It's a safe place to experiment with how things look.

## The Network Tab

The **Network** tab shows every request your page makes, like loading images or fetching data from a server.

This is very useful when working with `fetch` (getting data from the internet). You can:

- See if a request succeeded or failed.
- Check the status code (200 means success, 404 means not found).
- Click a request to see the data it returned.

If your data isn't showing up on the page, the Network tab tells you whether the request even worked. Tip: reload the page with the Network tab open so you can see the requests happen.

## The Sources Tab and Breakpoints

The **Sources** tab shows your JavaScript files. Here you can use the most powerful debugging tool: the **breakpoint**.

A **breakpoint** is a marker that pauses your code at a specific line. When the code pauses, you can look at the values of all your variables at that exact moment.

To set one:

1. Open the Sources tab.
2. Find your JavaScript file.
3. Click the line number where you want to pause.

When your code reaches that line, it freezes, and you can inspect everything.

## Stepping Through Code

Once your code is paused at a breakpoint, you can move through it line by line using the control buttons:

- **Step over** — run the current line and move to the next one.
- **Step into** — go inside a function call to see what happens there.
- **Resume** — continue running until the next breakpoint (or the end).

Stepping lets you watch your program think, one step at a time. This is the best way to understand exactly where things go wrong.

## The debugger Statement

You can also pause code from inside your code using the `debugger` keyword:

```js
function calculateTotal(price, tax) {
  // Code will pause here when DevTools is open
  debugger;
  return price + tax;
}
```

When DevTools is open and the code reaches `debugger`, it pauses just like a breakpoint. Remember to remove it when you're done.

## Reading a Stack Trace

When an error happens, the Console often shows a **stack trace**. This is a list of the functions that were running when the error occurred, in order. Here's a simple example:

```bash
TypeError: Cannot read properties of undefined (reading 'name')
    at showUser (app.js:12)
    at loadPage (app.js:5)
    at app.js:1
```

How to read it:

- The **top line** is the actual error message. Read it carefully.
- The **next lines** show where it happened. `app.js:12` means file `app.js`, line `12`.
- Start from the top and look at *your* files first.

The error above means the code tried to read `.name` from something that doesn't exist yet (it's `undefined`).

## A Simple Debugging Strategy

When you hit a bug, stay calm and follow these steps:

1. **Read the error.** The message and stack trace usually point right at the problem.
2. **Isolate it.** Find the smallest piece of code that causes the bug.
3. **Test small.** Add `console.log` lines to check values, or use a breakpoint.
4. **Change one thing at a time.** Then check if it helped. This keeps things clear.
5. **Search the error message** online if you're stuck. Someone has likely had the same issue.

Debugging is a normal, daily part of coding. Each bug you solve makes you better.

## Common Mistakes

- **Ignoring the Console.** The answer is often sitting right there in red.
- **Leaving `debugger` or `console.log` in finished code.** Clean them up.
- **Changing many things at once.** Then you don't know what fixed (or broke) it.
- **Panicking at red errors.** Errors are helpful guides, not failures.
- **Forgetting DevTools must be open** for `debugger` to pause the code.

## Exercises

1. Open any website, open the Console, and type `console.log("Hello")`. Then create an array of objects and try `console.table` on it.
2. Write a small script with a deliberate error (like reading a property of `undefined`) and read the stack trace it produces.
3. Set a breakpoint inside a function in the Sources tab, then use "step over" to move through it line by line.
4. Open the Network tab, reload a page, and find one request. Check its status code.

## Key Takeaways

- Open DevTools with **F12** or right-click and Inspect.
- The **Console** shows your logs and errors. Read it first.
- Use `console.table`, `console.error`, and `console.warn` for clearer output.
- The **Elements** tab shows HTML/CSS; changes there are temporary.
- The **Network** tab shows requests, great for checking `fetch`.
- **Breakpoints** and the `debugger` statement pause code so you can inspect it.
- Read the **stack trace** from the top, focusing on your files.
- Debug calmly: read, isolate, test small, change one thing at a time.

---

[← Back to Git and GitHub Basics](./02-git-and-github.md) | [Back to Course Index](../README.md) | [Next: LocalStorage →](./04-localstorage.md)
