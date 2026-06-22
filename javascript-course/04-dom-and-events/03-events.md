# Events and Event Listeners

Pages get truly interactive when they **react** to the user. A click, a key press, a
form sent — these are all **events**. In this lesson you will learn how to listen for
events and run code when they happen. This is where your pages start to feel alive! 🎬

## What You'll Learn

- What an **event** is
- How to listen for events with `addEventListener`
- The **event object** and `event.target`
- Common events: `click`, `input`, `change`, `submit`, `keydown`, `mouseover`
- How to stop listening with `removeEventListener`
- **Event bubbling** and **delegation**, explained simply
- How to stop default behavior with `event.preventDefault`

## What Is an Event?

An **event** is something that happens on the page — usually something the user does.

Examples of events:

- The user **clicks** a button.
- The user **types** in a text box.
- The user **submits** a form.
- The user **moves** the mouse over an image.

Your job as a developer is to **listen** for these events and decide what should
happen.

Analogy: Think of a doorbell. The "event" is someone pressing the button. The
"listener" is you, waiting to hear it. When the bell rings, you react — you open the
door. In code, you set up a listener, and when the event happens, your code runs.

## `addEventListener`

The main tool for listening to events is `addEventListener`. You call it on an
element. It takes two things:

1. The **name** of the event (like `"click"`).
2. A **function** to run when the event happens. This function is called the
   **handler** or **callback**.

```html
<button id="myBtn">Click me</button>

<script>
  const button = document.getElementById("myBtn");

  // Listen for clicks on the button
  button.addEventListener("click", function () {
    console.log("Button was clicked!");
  });
</script>
```

Every time you click the button, the function runs and the message appears. You can
also use an arrow function, which is shorter:

```js
button.addEventListener("click", () => {
  console.log("Clicked!");
});
```

> Note: do **not** put `()` after the function name when you pass it. You pass the
> function itself, not its result. `addEventListener("click", sayHi)` is correct;
> `addEventListener("click", sayHi())` is a common mistake (it runs `sayHi`
> immediately instead of waiting for the click).

## The Event Object

When an event happens, the browser gives your handler a special object full of
details about what happened. By tradition, we name it `event` (or just `e`).

```js
button.addEventListener("click", function (event) {
  console.log(event); // the full event object
  console.log(event.type); // => click
});
```

### `event.target`

The most useful property is `event.target`. It is the **element that the event
happened on** — the thing the user actually touched.

```html
<button id="myBtn">Click me</button>

<script>
  const button = document.getElementById("myBtn");

  button.addEventListener("click", function (event) {
    console.log(event.target); // the <button> element
    console.log(event.target.textContent); // => Click me
    event.target.textContent = "Clicked!"; // change its text
  });
</script>
```

`event.target` is very handy when many elements share one listener (you will see why
in the **delegation** section below).

## Common Events

There are many events, but these are the ones you will use most often.

### `click`

Fires when the user clicks an element.

```js
button.addEventListener("click", () => {
  console.log("clicked");
});
```

### `input`

Fires **every time** the value of a text box changes — on each key press. Great for
live updates.

```html
<input id="name" placeholder="Type your name" />
<p id="output"></p>

<script>
  const input = document.getElementById("name");
  const output = document.getElementById("output");

  input.addEventListener("input", (event) => {
    output.textContent = "Hello, " + event.target.value;
  });
</script>
```

### `change`

Fires when an input's value changes **and the user moves away** (or for a checkbox,
when it is toggled). It fires less often than `input`.

```js
input.addEventListener("change", (event) => {
  console.log("Final value:", event.target.value);
});
```

### `submit`

Fires when a **form** is submitted (usually by clicking a submit button or pressing
Enter). You attach this to the `<form>`, not the button. (Full forms come in the next
lesson.)

```js
form.addEventListener("submit", (event) => {
  console.log("Form submitted");
});
```

### `keydown`

Fires when the user presses a key on the keyboard. The event object tells you which
key.

```js
document.addEventListener("keydown", (event) => {
  console.log("You pressed:", event.key);
  if (event.key === "Enter") {
    console.log("That was Enter!");
  }
});
```

### `mouseover`

Fires when the mouse moves **onto** an element. (Its partner, `mouseout`, fires when
the mouse leaves.)

```js
const box = document.getElementById("box");

box.addEventListener("mouseover", () => {
  box.style.backgroundColor = "lightblue";
});

box.addEventListener("mouseout", () => {
  box.style.backgroundColor = "white";
});
```

## `removeEventListener`

Sometimes you want to **stop** listening. Use `removeEventListener`. But there is a
catch: you must pass the **same function** you added. That means you cannot use an
anonymous (unnamed) function — you need a named one.

```js
function handleClick() {
  console.log("clicked once");
}

// Start listening
button.addEventListener("click", handleClick);

// Later, stop listening
button.removeEventListener("click", handleClick);
```

A common use is a listener that should run only **once**:

```js
function greetOnce() {
  console.log("Hello!");
  button.removeEventListener("click", greetOnce); // remove itself
}

button.addEventListener("click", greetOnce);
```

> Shortcut: `addEventListener("click", greetOnce, { once: true })` does the same
> thing — it removes itself automatically after one run.

## Event Bubbling and Delegation

This idea sounds advanced, but it is actually helpful. Let's go slowly.

### Bubbling

When you click an element, the event does not stop there. It **bubbles up** to that
element's parent, then the parent's parent, all the way up.

Think of bubbles rising in a glass of soda — they start at the bottom and float
upward. A click on a `<li>` also reaches the `<ul>` around it, then the `<body>`,
and so on.

```html
<ul id="list">
  <li>Apple</li>
  <li>Banana</li>
</ul>

<script>
  const list = document.getElementById("list");

  // Click any <li> — the click bubbles up to the <ul>
  list.addEventListener("click", (event) => {
    console.log("List heard a click on:", event.target.textContent);
  });
</script>
```

Click "Apple", and even though the listener is on the `<ul>`, it still runs — because
the click bubbled up. `event.target` tells you it was the "Apple" `<li>`.

### Delegation

**Event delegation** uses bubbling to your advantage. Instead of adding a listener to
**every** child, you add **one** listener to the **parent**. Then you check
`event.target` to see which child was clicked.

Why is this great? Imagine a list with 100 items, or a list where items are added
later. Adding 100 listeners is wasteful, and new items would have no listener.
Delegation solves both problems with a single listener.

```html
<ul id="todo-list">
  <li>Buy milk</li>
  <li>Walk dog</li>
  <li>Read book</li>
</ul>

<script>
  const todoList = document.getElementById("todo-list");

  // ONE listener on the parent handles ALL items, even future ones
  todoList.addEventListener("click", (event) => {
    // Make sure an <li> was clicked, not the empty space
    if (event.target.tagName === "LI") {
      event.target.classList.toggle("done"); // cross it off
    }
  });
</script>
```

Now clicking any item toggles a `done` class — and if you add new `<li>` items later
with JavaScript, they work too, with no extra code. That is the power of delegation.

> `event.target.tagName` gives the tag name in capital letters, like `"LI"` or
> `"BUTTON"`. We check it so we only react to the right elements.

## Preventing Default Behavior

Some elements have a **built-in** action:

- Clicking a link goes to a new page.
- Submitting a form reloads the page.

Sometimes you want to **stop** that default action and do your own thing instead. Use
`event.preventDefault()`.

```html
<a id="link" href="https://example.com">Visit (but not really)</a>

<script>
  const link = document.getElementById("link");

  link.addEventListener("click", (event) => {
    event.preventDefault(); // stop the page from navigating
    console.log("Link click was blocked!");
  });
</script>
```

The most common use is forms. By default, submitting a form reloads the page, which
wipes out your JavaScript. `preventDefault` stops that so your code can handle the
data. You will use this a lot in the next lesson.

```js
form.addEventListener("submit", (event) => {
  event.preventDefault(); // stop the reload
  console.log("Now I handle the form myself");
});
```

## Common Mistakes

- **Calling the function instead of passing it.** Use
  `addEventListener("click", doThing)`, not `doThing()` — the `()` runs it right
  away.
- **Listening before the element exists.** Run your script after the page loads
  (bottom of `<body>` or `defer`), or the element is `null`.
- **Removing a listener with a different function.** `removeEventListener` needs the
  exact same function reference; anonymous functions cannot be removed.
- **Forgetting `preventDefault` on forms.** Without it, the page reloads and your
  code seems to "do nothing".
- **Adding a listener per item when delegation would be simpler.** For lists, one
  listener on the parent is usually cleaner.

## Exercises

1. Make a button that, when clicked, changes its own text to "You clicked me!" using
   `event.target`.
2. Make a text input and a paragraph. Use the `input` event so the paragraph shows
   what the user types, live, as they type.
3. Build a `<ul>` with several `<li>` items and use **event delegation** (one
   listener on the `<ul>`) so clicking any item toggles a `done` class.
4. Make a link that uses `event.preventDefault()` so clicking it logs a message
   instead of going to another page.

## Key Takeaways

- An **event** is something that happens, usually a user action like a click or key
  press.
- `addEventListener(eventName, handler)` runs your function when the event happens.
- The **event object** holds details; `event.target` is the element that was acted
  on.
- Common events: `click`, `input`, `change`, `submit`, `keydown`, `mouseover`.
- `removeEventListener` stops listening, but needs the same named function.
- Events **bubble** up to parents; **delegation** uses one parent listener for many
  children.
- `event.preventDefault()` stops built-in actions like link navigation and form
  reloads.

---

[← Back to Changing the Page](02-manipulating-the-dom.md) &nbsp;&nbsp;|&nbsp;&nbsp; [Next: Forms and User Input](04-forms-and-inputs.md)
