# The HTML Document Structure

Every web page you have ever seen is built from HTML. Think of HTML as the skeleton of a page: it holds everything in place and gives each piece a name. In this lesson you will learn the basic building blocks of HTML and the standard "shape" that every page starts with.

## What You'll Learn

- What an HTML **element** is, and its three parts
- The difference between **tags**, **elements**, and **attributes**
- What **void** (self-closing) elements are
- The standard boilerplate that starts every HTML page
- What goes in the `<head>` versus the `<body>`
- How to **nest** elements neatly with indentation
- How to write **comments** in HTML

## What Is an HTML Element?

An **element** is a single piece of a web page, like a heading, a paragraph, or an image. Most elements have three parts:

1. An **opening tag** — tells the browser "a thing starts here."
2. The **content** — the text or other elements inside.
3. A **closing tag** — tells the browser "the thing ends here."

A **tag** is the keyword wrapped in angle brackets, like `<p>`. The closing tag looks the same but has a forward slash, like `</p>`.

```html
<p>Hello, world!</p>
```

Here:

- `<p>` is the opening tag.
- `Hello, world!` is the content.
- `</p>` is the closing tag.

Together, all three parts make one **element**. The `p` stands for "paragraph."

Think of it like a sandwich: the opening tag is the top slice of bread, the closing tag is the bottom slice, and the content is the filling in the middle.

## Tags vs Elements vs Attributes

These three words are easy to mix up, so let's be clear:

- A **tag** is just the bit in angle brackets: `<p>` or `</p>`.
- An **element** is the whole thing: opening tag + content + closing tag.
- An **attribute** is extra information you add inside the opening tag to change how the element behaves.

Here is an element with an attribute:

```html
<p lang="en">Hello, world!</p>
```

The attribute here is `lang="en"`. It has two parts:

- The **name** (`lang`), which says what the attribute controls (here, the language).
- The **value** (`"en"`), which is the setting you choose (here, English).

Attributes always live inside the opening tag, and the value is wrapped in quotation marks. You can have more than one attribute, separated by spaces.

## Void (Self-Closing) Elements

Some elements have no content to wrap, so they have no closing tag. These are called **void elements** (or self-closing elements). They do their job all on their own.

A common example is the line break:

```html
<br>
```

Another is the image element, which we will study later:

```html
<img src="cat.jpg" alt="A sleeping cat">
```

Notice there is no `</br>` or `</img>`. The element is complete with just one tag. You may sometimes see them written with a slash, like `<br />`. Both styles work in modern HTML, so don't worry if you see either one.

## The Boilerplate: Every Page Starts the Same Way

Almost every HTML page begins with the same standard structure. We call this the **boilerplate** — a starting template you reuse every time. Here is the full minimal document:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>My First Page</title>
  </head>
  <body>
    <h1>Hello, world!</h1>
    <p>This is my first web page.</p>
  </body>
</html>
```

If you save this as a file ending in `.html` and open it in a browser, you will see a big heading that says "Hello, world!" and a small line of text below it. Let's go through every line.

### `<!DOCTYPE html>`

This is the very first line of every page. It tells the browser "this is a modern HTML document." It is not really an element — think of it as a label on the box. Always put it at the top, and never change it.

### `<html lang="en">`

The `<html>` element wraps **everything** else on the page. It is the root, like the trunk of a tree. The `lang="en"` attribute tells the browser and screen readers that the page is written in English. This helps with accessibility and search engines.

### `<head>` — The Behind-the-Scenes Section

The `<head>` holds **metadata**: information *about* the page that the visitor does not see directly on the screen. Think of it as the label inside a shirt collar — important details that aren't part of the visible design.

Inside the `<head>` we have:

- **`<meta charset="UTF-8">`** — sets the **character encoding** to UTF-8. This makes sure letters, symbols, and emojis from every language show up correctly. Without it, some characters can turn into strange boxes or question marks.
- **`<meta name="viewport" content="width=device-width, initial-scale=1.0">`** — tells phones and tablets to size the page to fit their screen instead of zooming out. This is essential for pages that look good on mobile.
- **`<title>My First Page</title>`** — sets the text shown in the browser tab and in search results. This is one of the few `<head>` items that the user *does* see, just not in the page body.

### `<body>` — The Visible Content

The `<body>` holds everything the visitor actually sees: headings, paragraphs, images, links, buttons, and more. Whatever you want on the screen goes here.

```html
<body>
  <h1>Hello, world!</h1>
  <p>This is my first web page.</p>
</body>
```

If the `<head>` is the kitchen where preparation happens, the `<body>` is the dining table where the food is served.

## Nesting and Indentation

When one element sits inside another, we say it is **nested**. In our boilerplate, the `<h1>` and `<p>` are nested inside the `<body>`, which is nested inside the `<html>`.

A golden rule: elements must close in the reverse order they opened, like closing a set of nested boxes from the inside out.

```html
<body>
  <p>This is <strong>correct</strong> nesting.</p>
</body>
```

This is wrong, because the tags overlap instead of nesting cleanly:

```html
<p>This is <strong>wrong nesting.</p></strong>
```

To keep your code readable, use **indentation**: add a couple of spaces in front of each nested element. The browser ignores this spacing, but it makes the structure obvious to humans, like an outline.

## Comments

A **comment** is a note you write for yourself or other developers. The browser ignores it completely — it never appears on the page. Comments start with `<!--` and end with `-->`.

```html
<!-- This is a comment. The browser skips it. -->
<p>This paragraph is visible.</p>
```

Use comments to explain tricky parts or to temporarily "switch off" a chunk of HTML without deleting it.

## Common Mistakes

- Forgetting the closing tag, like writing `<p>` but never `</p>`.
- Leaving out `<!DOCTYPE html>` at the top of the file.
- Putting visible content (like a paragraph) inside the `<head>` instead of the `<body>`.
- Overlapping tags instead of nesting them properly.
- Forgetting the quotation marks around an attribute value.
- Saving the file without a `.html` ending, so the browser shows raw code.

## Exercises

1. Type out the full boilerplate from memory, then check it against this lesson. Save it as `index.html` and open it in your browser.
2. Change the `<title>` to your own name and confirm it appears in the browser tab.
3. Add a second paragraph inside the `<body>` describing your favorite hobby.
4. Add a comment above one of your paragraphs explaining what it is for.
5. On purpose, remove a closing `</p>` tag and reload the page. Notice what changes, then put it back.

## Key Takeaways

- An **element** has an opening tag, content, and a closing tag.
- **Void elements** like `<br>` and `<img>` have no closing tag.
- **Attributes** add extra information inside the opening tag, as `name="value"`.
- Every page starts with `<!DOCTYPE html>`, then `<html>`, `<head>`, and `<body>`.
- The `<head>` holds invisible metadata; the `<body>` holds visible content.
- Nest elements neatly and use indentation and comments to stay organized.

---

[← Back to Course Index](../README.md) | [Next: Text, Headings, and Paragraphs →](02-text-and-headings.md)
