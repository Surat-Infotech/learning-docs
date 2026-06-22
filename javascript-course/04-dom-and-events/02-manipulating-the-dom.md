# Changing the Page (DOM Manipulation)

Now for the magic! ✨ In this lesson you will learn how to **change** a web page with
JavaScript: change text, change colors, add new elements, and remove old ones. This
is called **DOM manipulation**. "Manipulation" just means "changing things". Let's
make pages do what we want!

## What You'll Learn

- How to **select** elements three ways
- How to read and change **text** (`textContent`, `innerText`)
- How to change **HTML** (`innerHTML`) — and a safety warning
- How to change **styles** with `.style`
- How to change **classes** with `classList`
- How to read and change **attributes** and **properties**
- How to **create** new elements
- How to **remove** elements
- A full mini example that builds a list

## Selecting Elements

Before you can change an element, you must **select** it (grab it). There are a few
ways. Here are the three you will use most.

### `getElementById`

Finds **one** element by its `id`. An `id` is a unique name you give an element.

```html
<h1 id="title">Hello</h1>

<script>
  const heading = document.getElementById("title");
  console.log(heading.textContent); // => Hello
</script>
```

Note: you do **not** write the `#` symbol here, just the id name.

### `querySelector`

Finds the **first** element that matches a **CSS selector**. This is the most
flexible tool.

```html
<p class="note">First note</p>
<p class="note">Second note</p>

<script>
  // Here you DO use # for id and . for class, like in CSS
  const firstNote = document.querySelector(".note");
  console.log(firstNote.textContent); // => First note
</script>
```

Quick reminder of selectors:

- `"p"` → the first `<p>` tag
- `".note"` → the first element with class `note`
- `"#title"` → the element with id `title`

### `querySelectorAll`

Finds **all** elements that match, and gives you a list of them.

```html
<p class="note">First note</p>
<p class="note">Second note</p>

<script>
  const notes = document.querySelectorAll(".note");
  console.log(notes.length); // => 2

  // Loop over each one
  notes.forEach((note) => {
    console.log(note.textContent);
  });
  // => First note
  // => Second note
</script>
```

> The list from `querySelectorAll` is called a **NodeList**. It is like an array.
> You can loop it with `forEach`. If you need full array methods, turn it into a real
> array with `Array.from(notes)`.

## Reading and Changing Text

Once you have an element, you can read or change its text.

### `textContent`

`textContent` is the **text inside** an element. You can read it or set it.

```html
<h1 id="title">Hello</h1>

<script>
  const heading = document.getElementById("title");

  // Read the text
  console.log(heading.textContent); // => Hello

  // Change the text
  heading.textContent = "Goodbye";
  // The page now shows: Goodbye
</script>
```

### `innerText`

`innerText` is very similar to `textContent`. The small difference: `innerText`
cares about how the page *looks* — it skips text that is hidden by CSS, and it
respects line breaks the user sees.

For most beginner work, **use `textContent`**. It is simpler and faster.

```js
heading.innerText = "Visible text only";
```

## Changing HTML

Sometimes you do not just want plain text — you want to add real HTML, like a bold
word or a link. For that, use `innerHTML`.

```html
<div id="box"></div>

<script>
  const box = document.getElementById("box");

  // This adds real HTML, not just text
  box.innerHTML = "<strong>Important!</strong>";
  // The page shows the word "Important!" in bold
</script>
```

### ⚠️ A Safety Warning About `innerHTML`

`innerHTML` is powerful, but it can be **dangerous**. If you put text from a user
(like a comment or a search box) into `innerHTML`, a bad person could sneak in code
that runs on your page. This attack is called **XSS** (cross-site scripting).

**The rule:**

- Use `textContent` when you are inserting **plain text** (this is safe).
- Only use `innerHTML` with HTML that **you** wrote and trust.
- Never put **untrusted user input** straight into `innerHTML`.

```js
// BAD: user input straight into innerHTML — dangerous!
box.innerHTML = userTypedText;

// GOOD: user input as plain text — safe
box.textContent = userTypedText;
```

## Changing Styles

You can change how an element looks using `.style`. This lets you set CSS properties
from JavaScript.

```html
<p id="message">Hello</p>

<script>
  const message = document.getElementById("message");

  message.style.color = "red"; // text turns red
  message.style.fontSize = "24px"; // text gets bigger
  message.style.backgroundColor = "yellow"; // yellow background
</script>
```

Notice the names! In CSS you write `font-size` and `background-color` (with
dashes). In JavaScript you write `fontSize` and `backgroundColor` (no dashes, second
word capitalized). This style is called **camelCase**.

> Tip: For more than one or two style changes, it is cleaner to use **classes**
> (next section) instead of setting many `.style` properties one by one.

## Changing Classes

A **class** is a label you put on an element. CSS uses classes to apply styles. From
JavaScript you can add, remove, or toggle classes using `classList`.

Imagine this CSS already exists:

```html
<style>
  .highlight {
    background-color: yellow;
  }
  .hidden {
    display: none;
  }
</style>

<p id="msg">Watch me</p>
```

Now control the classes:

```js
const msg = document.getElementById("msg");

// Add a class
msg.classList.add("highlight"); // now yellow

// Remove a class
msg.classList.remove("highlight"); // back to normal

// Toggle: add if missing, remove if present
msg.classList.toggle("hidden"); // hide it
msg.classList.toggle("hidden"); // show it again

// Check if a class is there (true or false)
console.log(msg.classList.contains("highlight")); // => false
```

`classList.toggle` is great for things like a "show more" button or a dark-mode
switch — one method flips the class on and off.

## Changing Attributes and Properties

An **attribute** is the extra info written inside an HTML tag, like `href` in
`<a href="...">` or `src` in `<img src="...">`.

### `getAttribute` and `setAttribute`

```html
<a id="link" href="https://example.com">Visit</a>

<script>
  const link = document.getElementById("link");

  // Read an attribute
  console.log(link.getAttribute("href")); // => https://example.com

  // Change an attribute
  link.setAttribute("href", "https://google.com");
</script>
```

### Properties (the shortcut way)

Many common attributes also work as **properties** — values you read or set directly
with a dot. This is often easier.

```html
<input id="name" value="Sam" />
<img id="photo" src="cat.jpg" />
<a id="site" href="https://example.com">Site</a>

<script>
  // .value — the text inside an input box
  const nameInput = document.getElementById("name");
  console.log(nameInput.value); // => Sam
  nameInput.value = "Alex"; // changes the input box

  // .src — the image file
  const photo = document.getElementById("photo");
  photo.src = "dog.jpg";

  // .href — a link's address
  const site = document.getElementById("site");
  site.href = "https://google.com";
</script>
```

> Rule of thumb: for `value`, `src`, and `href`, the dot-property way
> (`element.value`) is usually the simplest. For unusual attributes, use
> `getAttribute` / `setAttribute`.

## Creating New Elements

You can build brand-new elements with JavaScript and add them to the page. This is
how apps show new content without reloading.

It takes two steps:

1. **Create** the element with `document.createElement`.
2. **Add** it to the page with `append` or `appendChild`.

```html
<ul id="list"></ul>

<script>
  const list = document.getElementById("list");

  // 1. Create a new <li> element
  const item = document.createElement("li");

  // 2. Give it some text
  item.textContent = "Milk";

  // 3. Add it inside the <ul>
  list.append(item);
  // The page now shows a list with "Milk"
</script>
```

### `append` vs `appendChild`

Both add an element as the last child of a parent. They are almost the same:

- `appendChild(node)` — the older way. Adds one element node.
- `append(...)` — the newer way. Can add elements **and** plain text, and can add
  several at once.

```js
list.appendChild(item); // works fine
list.append(item); // also works, and is more flexible
list.append("Just some text"); // append can add text too
```

For new code, `append` is a great default.

## Removing Elements

To remove an element, call `.remove()` on it.

```html
<p id="goodbye">Remove me</p>

<script>
  const p = document.getElementById("goodbye");
  p.remove(); // the paragraph disappears from the page
</script>
```

That is it — one method, and it is gone.

## Full Mini Example: Build a List

Let's put it all together. This example takes a list of fruits and builds an HTML
list from it, then styles one item.

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Fruit List</title>
    <style>
      .special {
        font-weight: bold;
        color: green;
      }
    </style>
  </head>
  <body>
    <h1 id="heading">My Fruits</h1>
    <ul id="fruit-list"></ul>

    <script>
      // Our data
      const fruits = ["Apple", "Banana", "Cherry"];

      // Grab the empty list
      const list = document.getElementById("fruit-list");

      // Build one <li> per fruit
      fruits.forEach((fruit) => {
        const item = document.createElement("li"); // create
        item.textContent = fruit; // set text
        list.append(item); // add to page
      });

      // Style the first item using a class
      const firstItem = list.querySelector("li");
      firstItem.classList.add("special"); // Apple becomes bold and green

      // Change the heading
      const heading = document.getElementById("heading");
      heading.textContent = "My Favorite Fruits";
    </script>
  </body>
</html>
```

When this page loads, JavaScript builds the whole list from the `fruits` array,
makes "Apple" stand out, and updates the heading — all without you writing the
`<li>` tags by hand. That is the power of DOM manipulation. 🎉

## Common Mistakes

- **Using `#` and `.` in the wrong place.** `getElementById("title")` takes no `#`.
  `querySelector("#title")` needs the `#`.
- **Selecting an element before it exists.** Run your script at the bottom of
  `<body>` or use `defer`, or `querySelector` returns `null`.
- **Putting user input into `innerHTML`.** This is unsafe. Use `textContent` for
  plain text.
- **CSS names with dashes in `.style`.** Use `fontSize`, not `font-size`.
- **Forgetting to add a created element.** `createElement` builds an element, but it
  is not on the page until you `append` it.
- **Trying array methods on a NodeList.** Use `forEach`, or `Array.from(list)` first.

## Exercises

1. Make an HTML page with a button and a paragraph. Using JavaScript, change the
   paragraph's `textContent` and its `.style.color`. (You will learn clicks next
   lesson — for now just run the change directly.)
2. Create a `<ul>` and use `createElement` and `append` in a loop to add five list
   items from an array of your favorite movies.
3. Add a class called `done` (that puts a line through text using CSS
   `text-decoration: line-through`) and use `classList.toggle` to turn it on and off
   on one list item.
4. Make an `<img>` and use JavaScript to change its `.src` to a different image file.

## Key Takeaways

- Select elements with `getElementById`, `querySelector`, or `querySelectorAll`.
- Use `textContent` for plain text (safe); use `innerHTML` only for trusted HTML.
- Change looks with `.style` (camelCase names) or, better, with `classList`
  (`add`, `remove`, `toggle`, `contains`).
- Read/change attributes with `getAttribute` / `setAttribute`, or use properties
  like `.value`, `.src`, `.href`.
- Build elements with `createElement`, add them with `append` / `appendChild`.
- Remove elements with `.remove()`.

---

[← Back to The DOM](01-the-dom.md) &nbsp;&nbsp;|&nbsp;&nbsp; [Next: Events and Event Listeners](03-events.md)
