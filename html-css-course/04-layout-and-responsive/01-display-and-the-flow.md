# Display and the Normal Flow

Before you can build layouts, you need to understand how the browser places elements on the page by default. This default behavior is called the **normal flow**. Once you know how the flow works, every layout tool you learn later will make more sense.

## What You'll Learn

- What the "normal flow" is and why it matters
- The difference between **block**, **inline**, and **inline-block** elements
- How to change an element's behavior with the `display` property
- How block elements stack and inline elements sit side by side
- How width and height behave for each display type
- How to hide things with `display: none` and `visibility: hidden`
- That `flex` and `grid` are also `display` values (coming in later lessons)

## What Is the Normal Flow?

When a web page loads, the browser reads your HTML from top to bottom and places each element one after another. This default arrangement is called the **normal flow** (sometimes just "the flow" or "document flow").

Think of it like stacking books and laying out playing cards:

- Some elements stack on top of each other, like books in a pile.
- Other elements sit next to each other in a line, like cards dealt across a table.

Which behavior an element uses depends on its **display type**. The two most common types are **block** and **inline**.

## Block Elements

A **block element** takes up the full width available, no matter how little content it holds. Each block element starts on a new line, so block elements stack vertically like a pile of boxes.

Common block elements include `<div>`, `<p>`, `<h1>` through `<h6>`, `<section>`, and `<ul>`.

```html
<p>I am the first paragraph.</p>
<p>I am the second paragraph.</p>
<p>I am the third paragraph.</p>
```

Even though the text is short, each paragraph stretches across the whole page and sits on its own line:

```text
[ I am the first paragraph.            ]
[ I am the second paragraph.           ]
[ I am the third paragraph.            ]
```

### Width and Height on Block Elements

Block elements accept `width` and `height`. If you do not set a width, they fill the available space. If you do set one, they stay at that width and the leftover space stays empty.

```css
.box {
  background: lightblue;
  width: 200px;
  height: 80px;
}
```

```html
<div class="box">A block with a fixed size</div>
```

## Inline Elements

An **inline element** is only as wide as its content. Inline elements do **not** start on a new line. Instead, they sit next to each other in a row, wrapping to the next line only when they run out of horizontal space, just like words in a sentence.

Common inline elements include `<span>`, `<a>`, `<strong>`, `<em>`, and `<img>`.

```html
<p>
  Visit our <a href="#">home page</a> or read the
  <strong>important notice</strong> below.
</p>
```

The link and the bold text flow inside the line of text. They do not push other content onto a new line.

### Width and Height on Inline Elements

This is the key surprise for beginners: **inline elements ignore `width` and `height`.** You can write them, but the browser will not apply them.

```css
.tag {
  background: gold;
  width: 300px;   /* ignored */
  height: 100px;  /* ignored */
}
```

```html
<span class="tag">I stay the size of my text</span>
```

The `<span>` only grows to fit "I stay the size of my text". Top and bottom margins are also ignored on inline elements, which often confuses people. If you need width and height, you need a different display type.

## Inline-Block Elements

What if you want the best of both worlds? You want elements to sit side by side **and** respect width and height. That is exactly what **inline-block** is for.

An **inline-block element** flows in a line like an inline element, but it accepts `width`, `height`, and all margins like a block element.

```css
.chip {
  display: inline-block;
  width: 120px;
  height: 40px;
  background: salmon;
  text-align: center;
}
```

```html
<span class="chip">One</span>
<span class="chip">Two</span>
<span class="chip">Three</span>
```

The three chips line up next to each other, but each is exactly 120px wide and 40px tall:

```text
[  One   ] [  Two   ] [ Three  ]
```

This was a popular way to build button rows and small cards before flexbox arrived.

## The `display` Property

You are not stuck with an element's default type. The `display` property lets you change how any element behaves.

```css
/* Make a span act like a block */
.banner {
  display: block;
}

/* Make a list item sit in a line */
.menu-item {
  display: inline;
}

/* Flow in a line but accept size */
.card {
  display: inline-block;
}
```

A common real example is turning list items into a horizontal menu:

```css
nav li {
  display: inline-block;
  margin-right: 16px;
}
```

```html
<nav>
  <ul>
    <li>Home</li>
    <li>About</li>
    <li>Contact</li>
  </ul>
</nav>
```

The list items, which are normally block, now sit side by side.

## Hiding Elements: `display: none` vs `visibility: hidden`

There are two common ways to hide something, and they behave differently.

**`display: none`** removes the element completely. It takes up no space at all. The page acts as if the element does not exist.

```css
.hidden {
  display: none;
}
```

**`visibility: hidden`** makes the element invisible but keeps its space. There is now an empty gap where the element used to be.

```css
.invisible {
  visibility: hidden;
}
```

Here is the difference side by side:

```html
<p>Line one</p>
<p style="display: none;">Line two (display none)</p>
<p>Line three</p>
```

With `display: none`, line one and line three sit right next to each other.

```html
<p>Line one</p>
<p style="visibility: hidden;">Line two (visibility hidden)</p>
<p>Line three</p>
```

With `visibility: hidden`, there is a blank gap between line one and line three where line two still reserves its space.

Use `display: none` when you want the element fully gone. Use `visibility: hidden` when you want to keep the layout from shifting.

## A Peek Ahead: `flex` and `grid`

The `display` property has two more powerful values you will meet soon: **`flex`** and **`grid`**.

```css
.row {
  display: flex;   /* turns on Flexbox (next lessons) */
}

.gallery {
  display: grid;   /* turns on CSS Grid (next lessons) */
}
```

These turn an element into a **layout container** and unlock modern tools for arranging children in rows, columns, and full two-dimensional grids. You do not need them yet. Just know that they are also `display` values, so everything you learned here is the foundation they build on.

## Common Mistakes

- Setting `width` or `height` on an inline element and wondering why nothing changes. Switch it to `inline-block` or `block` first.
- Applying top and bottom margins to inline elements, which ignore them.
- Using `visibility: hidden` when you meant to fully remove an element, leaving an unexpected empty gap.
- Forgetting that block elements stretch to full width, then being surprised they push other content onto new lines.
- Mixing up `display: none` (gone, no space) with `visibility: hidden` (invisible, still takes space).

## Exercises

1. Create three `<div>` elements with text. Confirm they stack vertically. Then add `display: inline-block` and a fixed `width` to each and watch them line up in a row.
2. Make a `<span>` with a background color. Try to give it `width: 200px` and `height: 60px`. Note that nothing changes. Now add `display: inline-block` and see the size apply.
3. Build a simple horizontal navigation menu from a `<ul>` of links by setting `display: inline-block` on the list items.
4. Put three paragraphs in a row. Hide the middle one with `display: none`. Then change it to `visibility: hidden` and describe the difference you see.
5. Take a block `<div>`, give it `width: 300px`, and add a background color. Notice the empty space to its right that the block no longer fills.

## Key Takeaways

- The **normal flow** is the default top-to-bottom, left-to-right placement of elements.
- **Block** elements stack vertically, fill the available width, and accept width and height.
- **Inline** elements sit in a line and ignore width, height, and vertical margins.
- **Inline-block** elements flow in a line but accept width and height.
- The `display` property changes an element's behavior between these types.
- `display: none` removes an element and its space; `visibility: hidden` hides it but keeps the space.
- `flex` and `grid` are also `display` values that power modern layouts, coming up next.

---

[← Back to Course Index](../README.md) | [Next: Positioning →](02-positioning.md)
