# LocalStorage

Normally, when you reload a web page, everything resets. Any data your page was holding disappears. But sometimes you want to *remember* things, like a user's name or their to-do list. **LocalStorage** lets you save small bits of data in the browser so they survive a page reload. This lesson shows you how.

## What You'll Learn

- What LocalStorage is and what it's good for
- How to save, read, remove, and clear data
- Why LocalStorage only stores text (strings)
- How to save objects and arrays using JSON
- A small example: saving a to-do list
- The limits of LocalStorage and when not to use it
- The difference between LocalStorage and sessionStorage

## What Is LocalStorage?

**LocalStorage** is a small storage space inside the user's browser. You can save data there, and it stays even after the user:

- Reloads the page.
- Closes the tab.
- Closes the whole browser and comes back later.

It's perfect for small things like settings, a theme choice (light or dark), or a simple to-do list. The data is saved on the user's own computer, only for your website.

## The Four Main Methods

LocalStorage works with **keys** and **values**, like labeled boxes. You give each piece of data a name (the key) and a value.

```js
// Save data: setItem(key, value)
localStorage.setItem("username", "Ava");

// Read data: getItem(key)
let name = localStorage.getItem("username");
console.log(name); // "Ava"

// Remove one item: removeItem(key)
localStorage.removeItem("username");

// Remove everything for this website: clear()
localStorage.clear();
```

That's the whole basic toolkit:

- **setItem(key, value)** — save something.
- **getItem(key)** — read something (returns `null` if not found).
- **removeItem(key)** — delete one thing.
- **clear()** — delete everything.

## Important: Only Strings Are Allowed

Here is the most important rule: **LocalStorage can only store text (strings).**

If you try to save a number, it becomes text. If you try to save an object or array directly, it gets turned into a useless `"[object Object]"`.

```js
// This does NOT work the way you want
let user = { name: "Ava", age: 25 };
localStorage.setItem("user", user);
console.log(localStorage.getItem("user")); // "[object Object]" — broken!
```

## Saving Objects and Arrays with JSON

To save objects and arrays, we first turn them into text using **JSON**. JSON is a text format for data.

- **JSON.stringify** — turns an object or array into a string (to save it).
- **JSON.parse** — turns that string back into an object or array (to use it).

```js
let user = { name: "Ava", age: 25 };

// Turn the object into text, then save it
localStorage.setItem("user", JSON.stringify(user));

// Read the text back, then turn it into an object
let saved = JSON.parse(localStorage.getItem("user"));

console.log(saved.name); // "Ava"
console.log(saved.age);  // 25
```

Remember the pair: **stringify to save, parse to read.**

## Example: Saving a To-Do List

Let's save and load a list of tasks. This shows the whole pattern in action.

```js
// Our list of tasks (an array)
let todos = ["Buy milk", "Walk the dog", "Study JavaScript"];

// SAVE the list to LocalStorage as text
localStorage.setItem("todos", JSON.stringify(todos));

// ... later, maybe after a page reload ...

// LOAD the list back
let savedText = localStorage.getItem("todos");

// If something was saved, turn it back into an array.
// If nothing was saved, start with an empty array.
let loadedTodos = savedText ? JSON.parse(savedText) : [];

console.log(loadedTodos); // ["Buy milk", "Walk the dog", "Study JavaScript"]
```

Notice the safety check: `getItem` returns `null` if nothing was saved, so we use an empty array as a backup. This stops errors when the list doesn't exist yet.

## Limits and When NOT to Use It

LocalStorage is handy, but it has rules:

- **Size limit.** You usually get about 5 megabytes per website. That's a lot of text but not enough for big files like images or videos.
- **Strings only.** As we saw, you must use JSON for objects and arrays.
- **Not secure.** Anyone using the computer can open DevTools and read it.

**Never store sensitive data in LocalStorage**, such as:

- Passwords.
- Credit card numbers.
- Secret tokens or private keys.

For large or important data, use a real database on a server instead. LocalStorage is for small, non-secret, convenient data.

## sessionStorage: The Short-Lived Cousin

There is a similar tool called **sessionStorage**. It works exactly the same way (same methods), with one key difference:

- **LocalStorage** keeps data until you delete it. It survives closing the browser.
- **sessionStorage** keeps data only until the tab is closed. Close the tab and it's gone.

```js
// Same methods, but this data disappears when the tab closes
sessionStorage.setItem("tempColor", "blue");
console.log(sessionStorage.getItem("tempColor")); // "blue"
```

Use sessionStorage for temporary, single-visit data. Use LocalStorage when you want data to stick around.

## Common Mistakes

- **Forgetting JSON.** Saving objects without `JSON.stringify` gives you `"[object Object]"`.
- **Forgetting to parse.** Reading without `JSON.parse` gives you a string, not an object.
- **Not handling `null`.** If the key doesn't exist, `getItem` returns `null`. Always plan for that.
- **Storing secrets.** Never put passwords or private data in LocalStorage.
- **Expecting it to work across different websites.** Each website has its own separate storage.

## Exercises

1. Save your name with `localStorage.setItem`, then reload the page and read it back with `getItem`.
2. Save an object (like `{ theme: "dark", fontSize: 16 }`) using `JSON.stringify`, then read it back with `JSON.parse` and log one of its properties.
3. Build a tiny to-do array, save it, then load it with a safety check that uses an empty array if nothing was saved.
4. Try the same save-and-reload test with `sessionStorage`, then close the tab and reopen it. Notice the data is gone.

## Key Takeaways

- **LocalStorage** saves small data in the browser that survives reloads and restarts.
- Use `setItem`, `getItem`, `removeItem`, and `clear`.
- LocalStorage stores **only strings**.
- Use **JSON.stringify** to save objects/arrays and **JSON.parse** to read them back.
- `getItem` returns `null` when the key is missing, so always have a backup value.
- It has a size limit (about 5 MB) and is **not secure**, so never store passwords or secrets.
- **sessionStorage** is the same but clears when the tab closes.

---

[← Back to Browser DevTools and Debugging](./03-devtools-and-debugging.md) | [Back to Course Index](../README.md) | [Next: Testing Basics →](./05-testing.md)
