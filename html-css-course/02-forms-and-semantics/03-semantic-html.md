# Semantic HTML

Not all HTML tags are equal. Some tags carry meaning about *what* a piece of content is, while others are just empty boxes. Using meaningful tags is called writing **semantic HTML**, and it makes your pages clearer to browsers, search engines, and people using assistive tools. In this lesson you will learn the main semantic tags and how to build a well-structured page.

## What You'll Learn

- What "semantic" means and why it matters
- The main layout tags: `<header>`, `<nav>`, `<main>`, `<footer>`
- Content tags: `<section>`, `<article>`, `<aside>`
- How to show images with captions using `<figure>` and `<figcaption>`
- How to mark dates and times with `<time>`
- The difference between meaningless `<div>`/`<span>` and semantic tags
- What a document outline is and how to build a full semantic page

## What Does "Semantic" Mean?

The word **semantic** means "related to meaning". A **semantic** tag is one whose name tells you what kind of content it holds. For example, a `<nav>` tag holds navigation links, and a `<footer>` tag holds footer content. The tag *describes its content*.

Compare that to a `<div>`, which means nothing at all. It is just a generic box. A computer reading `<div>` learns nothing about what is inside.

### Why Semantic HTML Matters

Using the right tag helps in three big ways.

- **Accessibility**: People who cannot see the screen use **screen readers**, tools that read the page aloud. Semantic tags let these tools say things like "navigation" or "main content" so users can jump around quickly. A page made only of `<div>`s sounds like a flat, shapeless wall of text.
- **SEO**: **SEO** stands for Search Engine Optimization, which means helping search engines like Google understand your page. Semantic tags give search engines clear signals about what each part of the page is, which can help your page show up in results.
- **Readability**: Other developers (and future you) can read semantic HTML and instantly understand the layout. `<nav>` is obviously the menu; `<div class="nav">` requires guessing.

In short, semantic HTML adds meaning that humans and machines can both use.

## The Main Layout Tags

These tags describe the big regions of a typical web page.

### `<header>`

The **`<header>`** holds introductory content at the top of a page or a section. It often contains a logo, a site title, and sometimes the navigation menu.

```html
<header>
  <h1>Maria's Bakery</h1>
  <p>Fresh bread, baked daily</p>
</header>
```

### `<nav>`

The **`<nav>`** holds the main navigation links, the menu that lets users move around the site.

```html
<nav>
  <a href="/">Home</a>
  <a href="/menu">Menu</a>
  <a href="/contact">Contact</a>
</nav>
```

You do not need to wrap *every* group of links in `<nav>`, just the major ones like the main menu.

### `<main>`

The **`<main>`** holds the central, most important content of the page. This is the unique part, the reason the page exists. There should be only **one** `<main>` per page.

```html
<main>
  <h2>Today's Specials</h2>
  <p>Sourdough, croissants, and cinnamon rolls.</p>
</main>
```

### `<footer>`

The **`<footer>`** holds closing content at the bottom, such as copyright text, contact details, or extra links.

```html
<footer>
  <p>&copy; 2026 Maria's Bakery</p>
</footer>
```

## Content Tags

These tags describe blocks of content *inside* the layout.

### `<section>`

A **`<section>`** groups related content under a common theme, usually with its own heading. Think of it as a chapter in a book.

```html
<section>
  <h2>Our Story</h2>
  <p>We opened our doors in 1998...</p>
</section>
```

### `<article>`

An **`<article>`** is a self-contained piece of content that would still make sense on its own. A blog post, a news story, or a single product card are all good `<article>`s. A handy test: if you could copy it and paste it somewhere else and it would still make sense, it is probably an article.

```html
<article>
  <h2>The History of Sourdough</h2>
  <p>Sourdough is one of the oldest forms of bread...</p>
</article>
```

The difference between `<section>` and `<article>`: a section is a thematic *part* of something bigger, while an article is a complete, standalone item.

### `<aside>`

An **`<aside>`** holds content that is related but not essential, like a sidebar, a tip box, or related links. It is "to the side" of the main content.

```html
<aside>
  <h3>Did you know?</h3>
  <p>Bread was being baked over 14,000 years ago.</p>
</aside>
```

## Figures and Captions

When you have an image, chart, or diagram that needs a caption, use **`<figure>`** to wrap it and **`<figcaption>`** for the caption text.

```html
<figure>
  <img src="croissant.jpg" alt="A golden, flaky croissant">
  <figcaption>Our signature butter croissant.</figcaption>
</figure>
```

This connects the picture and its caption as one unit, which is clearer than just placing a paragraph below an image.

## Marking Time with `<time>`

The **`<time>`** tag marks a date or time in a way machines can read. You show friendly text to people, and provide a precise version in the **`datetime`** attribute for computers.

```html
<p>The shop opens on <time datetime="2026-07-01">July 1st</time>.</p>
```

The user reads "July 1st", while a computer reads the exact, unambiguous `2026-07-01`. This helps calendars and search engines understand the date correctly.

## `<div>` and `<span>`: Boxes With No Meaning

So what about `<div>` and `<span>`? They are still useful, but they carry **no meaning**.

- **`<div>`** is a generic block-level box. It starts on a new line and is used to group things, usually for styling.
- **`<span>`** is a generic inline box. It sits within a line of text and is used to style a small piece of it.

```html
<div class="card">
  <span class="price">$3.50</span>
</div>
```

The rule of thumb: **reach for a semantic tag first**. Only use `<div>` or `<span>` when no semantic tag fits, usually when you just need a wrapper for styling. They are tools for *layout and style*, not for *meaning*.

## The Document Outline

A **document outline** is the structure of your page as understood from its headings and semantic regions, like a table of contents. Screen readers and search engines build this outline to understand your page.

Two habits keep your outline clean:

1. Use **headings in order**: `<h1>` for the page title, then `<h2>` for major sections, then `<h3>` underneath those. Do not skip levels.
2. Wrap content in the right semantic regions so the outline reflects the real structure.

A good outline is like a well-organized table of contents. A bad one (all `<div>`s, jumbled headings) is like a book with no chapter titles.

## A Full Semantic Page Skeleton

Here is how all these tags fit together into one complete page.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Maria's Bakery</title>
</head>
<body>

  <header>
    <h1>Maria's Bakery</h1>
    <nav>
      <a href="/">Home</a>
      <a href="/menu">Menu</a>
      <a href="/contact">Contact</a>
    </nav>
  </header>

  <main>
    <section>
      <h2>Welcome</h2>
      <p>Fresh bread and pastries, baked every morning.</p>
    </section>

    <article>
      <h2>The History of Sourdough</h2>
      <p>Sourdough is one of the oldest forms of bread...</p>
      <figure>
        <img src="sourdough.jpg" alt="A round loaf of sourdough bread">
        <figcaption>A freshly baked sourdough loaf.</figcaption>
      </figure>
      <p>Published on <time datetime="2026-06-22">June 22, 2026</time>.</p>
    </article>

    <aside>
      <h3>Opening Hours</h3>
      <p>Open daily from 7am to 6pm.</p>
    </aside>
  </main>

  <footer>
    <p>&copy; 2026 Maria's Bakery. All rights reserved.</p>
  </footer>

</body>
</html>
```

Notice how you can read this page's structure at a glance, even without seeing it in a browser. That is the power of semantic HTML.

## Common Mistakes

- Using `<div>` for everything when a semantic tag would say what the content is.
- Having more than one `<main>` on a page. There should be exactly one.
- Skipping heading levels, like jumping from `<h1>` straight to `<h4>`.
- Wrapping the whole page in `<section>` tags that have no heading or theme.
- Confusing `<article>` (standalone) with `<section>` (a themed part of a bigger whole).
- Using `<figure>` for images that have no caption (a plain `<img>` is fine there).

## Exercises

1. Take a plain page built only from `<div>`s and rewrite it using `<header>`, `<nav>`, `<main>`, and `<footer>`.
2. Write a blog post using an `<article>` with an `<h2>` heading, two paragraphs, and a `<figure>` with a `<figcaption>`.
3. Add an `<aside>` next to your main content containing a short "tip of the day".
4. Mark a publication date on your article using `<time>` with a correct `datetime` attribute.
5. Look at any web page you like and try to name which semantic region each part would be (header, nav, main, aside, footer).

## Key Takeaways

- "Semantic" means the tag's name describes its content's meaning.
- Semantic HTML improves accessibility, SEO, and readability.
- Use `<header>`, `<nav>`, `<main>` (only one), and `<footer>` for page regions.
- Use `<section>` for themed groups, `<article>` for standalone items, and `<aside>` for side content.
- Use `<figure>`/`<figcaption>` for captioned media and `<time>` for machine-readable dates.
- `<div>` and `<span>` carry no meaning; use them only for styling when no semantic tag fits.
- Order your headings and regions to give the page a clean document outline.

---

[← Back to Course Index](../README.md) | [Next: Accessibility Basics →](04-accessibility-basics.md)
