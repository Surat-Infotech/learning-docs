# The DOM

So far you have run JavaScript on its own. Now it is time for the fun part: making
web pages come alive! 🌐 Before you can change a page, you need to understand the
**DOM**. Do not worry — the name sounds scary, but the idea is simple. Let's go!

## What You'll Learn

- What the **DOM** is and why it matters
- What the **document** object is
- How a page is built like a **tree** of boxes (nodes and elements)
- What **parent** and **child** mean in this tree
- How JavaScript connects to a page using a `<script>` tag
- How to look at the DOM in your browser's **DevTools**
- A first peek at `document.querySelector`

## What Is the DOM?

**DOM** stands for **Document Object Model**. That is a mouthful, so let's break it
down.

When a browser opens a web page, it reads the **HTML** (the code that describes the
page). But the browser does not just show the HTML as text. Instead, it turns the
HTML into a set of **objects** that JavaScript can read and change.

> An **object** in JavaScript is a thing with properties (data) and abilities
> (actions). You learned about objects earlier in this course.

So the **DOM** is:

> The browser's living copy of your page, made out of objects that JavaScript can
> change.

Here is the key idea: **HTML is the starting point. The DOM is the live version.**
When JavaScript changes the DOM, the page on the screen changes too — instantly.

Analogy: Think of HTML as a written recipe on paper. The DOM is the actual meal
cooking on the stove. You can still add salt to the meal (change the DOM), and the
taste changes right away. The paper recipe does not change, but the meal does.

## The `document` Object

The browser gives you one special object that represents the **whole page**. It is
called `document`.

You do not create it. It is always there for you when your code runs in a browser.

```js
// `document` is the entry point to the whole page
console.log(document); // shows the full page structure
console.log(document.title); // shows the page's title text
```

Think of `document` as the **front door** to your page. Almost everything you do
with the DOM starts with `document`.

## The Tree Structure

A web page is shaped like a **tree**. Not a real tree — a family tree, turned upside
down.

Look at this small HTML page:

```html
<!DOCTYPE html>
<html>
  <head>
    <title>My Page</title>
  </head>
  <body>
    <h1>Hello</h1>
    <p>Welcome to my page.</p>
  </body>
</html>
```

The browser turns it into a tree like this:

```
document
└── html
    ├── head
    │   └── title  ("My Page")
    └── body
        ├── h1   ("Hello")
        └── p    ("Welcome to my page.")
```

Every box in this tree is called a **node**. A **node** is just one item in the
tree. Most of the nodes you care about are **elements**.

> An **element** is a node made from an HTML tag, like `<h1>`, `<p>`, or `<body>`.

### Parent and Child

The tree uses family words to describe how nodes connect:

- A **parent** is a node that contains other nodes.
- A **child** is a node inside another node.
- Nodes with the same parent are **siblings** (like brothers and sisters).

In the example above:

- `body` is the **parent** of `h1` and `p`.
- `h1` and `p` are **children** of `body`.
- `h1` and `p` are **siblings**.

Analogy: It is like folders on your computer. A folder (parent) can hold files and
other folders (children). Files in the same folder are siblings.

You do not need to memorize this now. Just remember: the page is a tree, and pieces
are connected as parents and children.

## How JavaScript Connects to a Page

HTML and JavaScript are two different things. HTML describes the page. JavaScript
adds behavior. To join them, you use a `<script>` tag inside your HTML.

A `<script>` tag tells the browser: "Run this JavaScript."

You can write the JavaScript right inside the tag:

```html
<body>
  <h1>Hello</h1>

  <script>
    // This JavaScript runs in the browser
    console.log("The page is loaded!");
  </script>
</body>
```

But the better way is to put your JavaScript in a separate file and link to it:

```html
<body>
  <h1>Hello</h1>

  <!-- Load JavaScript from a file named app.js -->
  <script src="app.js"></script>
</body>
```

> `src` means "source" — it points to the file the browser should load.

### Why Put the Script at the Bottom?

The browser reads HTML from top to bottom. If your script runs *before* the page
elements exist, it cannot find them. To avoid this, place your `<script>` tag at the
**bottom of the `<body>`**, just before `</body>`. By then, all the elements above
it are ready.

There is also a modern option: add the word `defer` to the tag. This tells the
browser to wait until the page is built before running the script.

```html
<!-- `defer` waits until the page is ready, even at the top -->
<script src="app.js" defer></script>
```

## Viewing the DOM in DevTools

Your browser has built-in tools called **DevTools** that let you look at the DOM.

To open them:

- Press **F12**, or
- Right-click anywhere on a page and choose **Inspect**.

Then click the **Elements** tab (in Chrome/Edge) or **Inspector** tab (in Firefox).
You will see the page's tree of elements. You can click the small arrows to open and
close parts of the tree, just like folders.

This is one of the most useful things a web developer does. When something looks
wrong on a page, you open DevTools and look at the DOM to understand what is really
there.

> Try it now: open any website, press **F12**, and explore the Elements tab.

## A First Peek at Selecting

Before the next lesson dives deep into changing the page, here is a tiny taste.

To work with an element, you first need to **select** it (grab it). The most common
tool is `document.querySelector`. It finds the **first** element that matches a
**CSS selector** you give it.

> A **CSS selector** is a short way to point at an element. For example, `"h1"`
> means "an `<h1>` element", and `".note"` means "an element with the class `note`".

```html
<h1>Hello</h1>
<p class="note">This is a note.</p>

<script>
  // Grab the first <h1> on the page
  const heading = document.querySelector("h1");
  console.log(heading.textContent); // => Hello

  // Grab the element with class "note"
  const note = document.querySelector(".note");
  console.log(note.textContent); // => This is a note.
</script>
```

Do not worry about all the details yet. The next lesson explains selecting and
changing elements fully. For now, just know: **you grab an element first, then you
can do things with it.**

## Common Mistakes

- **Mixing up HTML and the DOM.** HTML is the starting code. The DOM is the live
  version the browser builds and JavaScript can change.
- **Running a script before the page exists.** If your `<script>` runs before the
  elements load, it cannot find them. Put the script at the bottom of `<body>` or
  use `defer`.
- **Forgetting the `src` file path.** If `<script src="app.js">` points to the wrong
  place, the browser cannot load your code.
- **Thinking you must create `document`.** You do not. The browser always gives it
  to you for free.

## Exercises

1. Make a simple HTML file with a `<title>`, one `<h1>`, and two `<p>` paragraphs.
   Open it in a browser, press F12, and find each element in the Elements tab.
2. Draw the DOM tree for your file on paper. Label which nodes are parents, which are
   children, and which are siblings.
3. Add a `<script>` tag at the bottom of the `<body>` that logs `document.title` to
   the console. Open the console (F12) and check that the title appears.
4. Add `console.log(document.querySelector("h1").textContent)` to your script and
   confirm it prints your heading's text.

## Key Takeaways

- The **DOM** is the browser's live, object-based copy of your page that JavaScript
  can change.
- The **`document`** object is the front door to the whole page.
- A page is a **tree** of **nodes**; most useful nodes are **elements**.
- Nodes connect as **parent**, **child**, and **sibling**.
- A `<script>` tag connects JavaScript to a page; put it at the bottom of `<body>`
  or use `defer`.
- Use **DevTools** (F12) and the Elements tab to view the DOM.
- `document.querySelector` grabs the first element that matches a CSS selector.

---

[← Back to Course Index](../README.md) &nbsp;&nbsp;|&nbsp;&nbsp; [Next: Changing the Page (DOM Manipulation)](02-manipulating-the-dom.md)
