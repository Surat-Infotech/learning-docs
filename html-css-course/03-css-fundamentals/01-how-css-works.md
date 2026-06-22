# How CSS Works

HTML gives a web page its structure, like the walls and rooms of a house. CSS is the paint, the furniture, and the lighting. In this lesson you will learn what CSS is, how a CSS rule is built, and the different ways to add CSS to your pages.

## What You'll Learn

- What CSS is and what it does
- The parts of a CSS rule: selector, declaration, property, and value
- The three ways to add CSS to a page
- Why an external stylesheet is usually the best choice
- How the browser reads and applies your styles
- How to write comments in CSS
- A first taste of the cascade and inheritance

## What Is CSS?

**CSS** stands for **Cascading Style Sheets**. It is the language we use to style web pages. With CSS you control colors, fonts, sizes, spacing, layout, and much more.

Think of it this way. HTML says *what* something is (a heading, a paragraph, a button). CSS says *how* it should look (big and blue, with space around it).

Here is a tiny example. The HTML creates a heading. The CSS makes it blue and centered.

```html
<h1>Welcome to my site</h1>
```

```css
h1 {
  color: blue;
  text-align: center;
}
```

Without CSS, the heading would just be plain black text on the left. With CSS, it becomes blue and centered.

## The Anatomy of a Rule

A CSS **rule** is a single instruction that tells the browser how to style something. Every rule has the same shape. Let's break it down piece by piece.

```css
h1 {
  color: blue;
}
```

- **Selector** — the part before the curly braces. Here it is `h1`. The selector chooses *which* elements the rule applies to. This rule targets every `<h1>` on the page.
- **Declaration block** — everything inside the curly braces `{ }`. It holds one or more declarations.
- **Declaration** — a single style instruction, like `color: blue;`. A declaration is made of a property and a value.
- **Property** — the thing you want to change. Here it is `color`.
- **Value** — what you want to change it to. Here it is `blue`.
- **Colon** `:` — separates the property from the value.
- **Semicolon** `;` — ends each declaration. Always put one after every declaration.

Here is the same rule with labels:

```text
selector
  |
  h1 {
    color : blue ;
    |       |    |
 property  value semicolon
  }
```

A rule can have many declarations. Just put each one on its own line and end each with a semicolon.

```css
p {
  color: gray;
  font-size: 16px;
  line-height: 1.5;
}
```

This rule targets every `<p>` (paragraph) and sets three things at once.

## Three Ways to Add CSS

There are three places you can put CSS. Let's look at each one.

### 1. Inline CSS

**Inline CSS** lives right on the element, using the `style` attribute.

```html
<p style="color: red; font-size: 20px;">This text is red.</p>
```

This works, but it has problems. You can only style one element at a time, and the style mixes in with your HTML. It gets messy fast. Use inline CSS only for quick tests.

### 2. Internal CSS

**Internal CSS** goes inside a `<style>` tag in the `<head>` of your HTML file.

```html
<!DOCTYPE html>
<html>
  <head>
    <title>My Page</title>
    <style>
      p {
        color: green;
      }
    </style>
  </head>
  <body>
    <p>This text is green.</p>
  </body>
</html>
```

This is better. One rule can style many paragraphs. But the styles only apply to this one HTML file. If you have many pages, you would have to copy the styles into each one.

### 3. External CSS

**External CSS** lives in its own file that ends in `.css`. You connect it to your HTML with a `<link>` tag in the `<head>`.

First, the CSS file, named `styles.css`:

```css
p {
  color: purple;
}
```

Then, the HTML file links to it:

```html
<!DOCTYPE html>
<html>
  <head>
    <title>My Page</title>
    <link rel="stylesheet" href="styles.css" />
  </head>
  <body>
    <p>This text is purple.</p>
  </body>
</html>
```

The `<link>` tag has two important parts:

- `rel="stylesheet"` tells the browser this is a stylesheet.
- `href="styles.css"` is the path to your CSS file.

### Why External Is Best

External CSS is the way the pros work. Here is why:

- **One file, many pages.** Link the same `.css` file to every page on your site. Change one rule and every page updates.
- **Clean separation.** Your HTML stays about structure. Your CSS stays about style. Each file is easier to read.
- **Faster sites.** The browser downloads the CSS file once and reuses it across pages.

For real projects, almost always use external CSS.

## How the Browser Applies Styles

When you open a page, the browser does roughly this:

1. It reads the HTML and builds a model of the page in memory.
2. It reads all the CSS (inline, internal, and external).
3. For each element, it figures out which rules apply.
4. It paints the page using those styles.

The browser matches selectors to elements. If a rule says `h1 { color: blue; }`, the browser finds every `<h1>` and makes it blue.

## Comments in CSS

A **comment** is a note for humans. The browser ignores it. Comments help you explain what your CSS does.

In CSS, comments start with `/*` and end with `*/`.

```css
/* This styles the main heading */
h1 {
  color: blue; /* a calm, friendly color */
}

/*
You can also write
comments across
multiple lines.
*/
```

Use comments to label sections of your stylesheet so you can find things later.

## A First Full Example

Let's style a small page from top to bottom. Here is the HTML:

```html
<!DOCTYPE html>
<html>
  <head>
    <title>My First Styled Page</title>
    <link rel="stylesheet" href="styles.css" />
  </head>
  <body>
    <h1>Hello, World</h1>
    <p>This is my first styled page. Welcome!</p>
    <p>CSS makes the web colorful.</p>
  </body>
</html>
```

And here is `styles.css`:

```css
/* Style the page background and text */
body {
  background-color: #f4f4f4;
  color: #333333;
  font-family: Arial, sans-serif;
}

/* Make the heading stand out */
h1 {
  color: #2a7ae2;
  text-align: center;
}

/* Add some breathing room to paragraphs */
p {
  font-size: 18px;
  line-height: 1.6;
}
```

Notice how each rule targets a different element. The `body` rule sets a light gray background and dark gray text for the whole page. The `h1` rule colors and centers the heading. The `p` rule sizes the paragraph text and spaces out the lines.

## A Peek at the Cascade and Inheritance

Two ideas power CSS. We will study them deeply in later lessons, but here is a quick preview.

The **cascade** is how CSS decides what wins when two rules touch the same element. For example, if one rule says a paragraph is red and another says it is blue, the cascade decides. Order and specificity (how exact a selector is) matter here.

**Inheritance** means some styles flow down from a parent element to its children. If you set the text color on `body`, the paragraphs inside it usually inherit that color, even if you never style the paragraphs directly. This saves you from repeating yourself.

Do not worry about mastering these yet. Just know the words. We will come back to them.

## Common Mistakes

- **Forgetting the semicolon** at the end of a declaration. Without it, the next declaration may be ignored.
- **Forgetting the closing curly brace** `}`. This can break every rule that follows.
- **Wrong file path in `<link>`.** If `href` does not point to your CSS file, no styles load.
- **Using `//` for comments.** That is not valid CSS. Always use `/* */`.
- **Mixing up the colon and semicolon.** Use a colon `:` between property and value, and a semicolon `;` at the end.

## Exercises

1. Create an HTML file and a separate `styles.css` file. Link them with a `<link>` tag. Make all paragraphs dark blue.
2. Write a single CSS rule for `h1` that sets the color, makes it centered, and sets the font size. Use three declarations.
3. Add a comment above each rule in your stylesheet explaining what it does.
4. Take a paragraph and style it three different ways: once with inline CSS, once with internal CSS, and once with external CSS. Notice which method you prefer.
5. Set a `background-color` on the `body` and a different `color` for the text. Make sure the page is still easy to read.

## Key Takeaways

- CSS styles your web pages: colors, fonts, sizes, spacing, and layout.
- A rule is made of a selector and a declaration block of property-value pairs.
- End each declaration with a semicolon and wrap the block in curly braces.
- There are three ways to add CSS: inline, internal, and external.
- External CSS is usually best because it is reusable and keeps your code clean.
- Comments use `/* */` and are ignored by the browser.
- The cascade decides which rule wins, and inheritance passes some styles down to children. More on both later.

---

[← Back to Course Index](../README.md) | [Next: Selectors →](02-selectors.md)
