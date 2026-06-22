# Text, Headings, and Paragraphs

Most of the web is text: articles, blog posts, product descriptions, and more. HTML gives you special elements to mark up text so it has meaning and structure. In this lesson you will learn how to add headings, paragraphs, and many small touches that make writing clear and readable.

## What You'll Learn

- How to use headings `<h1>` through `<h6>` and order them correctly
- How to write paragraphs with `<p>`
- How to add line breaks `<br>` and dividers `<hr>`
- The difference between meaningful emphasis (`<strong>`, `<em>`) and plain styling (`<b>`, `<i>`)
- Useful inline tags: `<span>`, `<mark>`, `<small>`, `<sub>`, `<sup>`
- How to show quotes with `<blockquote>` and `<q>`
- How to show code with `<pre>` and `<code>`
- How **HTML entities** let you show special characters

## Headings

A **heading** is a title for a section of your page. HTML has six levels of heading, from `<h1>` (the most important) down to `<h6>` (the least important).

```html
<h1>Main Page Title</h1>
<h2>A Big Section</h2>
<h3>A Smaller Sub-Section</h3>
<h4>Even Smaller</h4>
<h5>Smaller Still</h5>
<h6>The Smallest Heading</h6>
```

By default, `<h1>` shows up large and bold, and each level down gets smaller.

### Use Only One `<h1>`

Think of a page like a book. The `<h1>` is the title of the whole book, and there should be only **one** of them per page. The `<h2>` headings are the chapter titles, the `<h3>` headings are the sections within a chapter, and so on.

### Heading Hierarchy

Headings should follow an order without skipping levels. Don't jump from an `<h2>` straight to an `<h4>`. This **hierarchy** helps screen readers and search engines understand how your content is organized.

```html
<h1>My Cooking Blog</h1>
  <h2>Breakfast Recipes</h2>
    <h3>Pancakes</h3>
    <h3>Omelettes</h3>
  <h2>Dinner Recipes</h2>
    <h3>Pasta</h3>
```

Important: choose a heading level for its **meaning**, not its size. If you just want smaller text, use CSS later — don't pick `<h4>` just because it looks small.

## Paragraphs

A **paragraph** is a block of regular text. Wrap it in `<p>` tags.

```html
<p>HTML is the language of the web. It tells the browser what each
piece of content means.</p>
```

The browser automatically adds space above and below each paragraph, so blocks of text are nicely separated.

### Whitespace Collapsing

Here is something that surprises beginners: the browser **collapses whitespace**. This means that many spaces, tabs, or new lines in your code all shrink down to a single space on the page.

```html
<p>This     sentence   has
many    spaces.</p>
```

On the page, this shows as: "This sentence has many spaces." — all the extra spacing disappears. So pressing Enter or Space many times in your HTML will not create gaps on the page.

## Line Breaks and Horizontal Rules

To force text onto a new line *without* starting a new paragraph, use `<br>`. It is a void element, so it has no closing tag.

```html
<p>Roses are red,<br>
Violets are blue.</p>
```

This shows the two lines stacked, like a short poem. Use `<br>` sparingly — for addresses or poems, not for spacing between paragraphs.

A **horizontal rule** `<hr>` draws a line across the page to separate sections. It is also a void element.

```html
<p>End of the first topic.</p>
<hr>
<p>Start of a brand new topic.</p>
```

## Inline Text: Emphasis and Styling

**Inline** elements style small bits of text *inside* a paragraph, without breaking onto a new line.

### Meaningful Emphasis: `<strong>` and `<em>`

- `<strong>` means the text is **important**. Browsers usually show it bold.
- `<em>` means the text should be **emphasized**, like stressing a word when speaking. Browsers usually show it in italics.

```html
<p><strong>Warning:</strong> do not touch the <em>hot</em> stove.</p>
```

These are **semantic** tags. "Semantic" means they carry meaning. A screen reader may change its tone of voice to reflect the importance or emphasis.

### Plain Styling: `<b>` and `<i>`

- `<b>` makes text bold with no special meaning.
- `<i>` makes text italic with no special meaning.

```html
<p>The species name is <i>Felis catus</i>.</p>
```

The rule of thumb: if the text truly matters or carries vocal stress, use `<strong>` or `<em>`. If you just want the *look* of bold or italics, use `<b>` or `<i>`.

### `<span>` — A Generic Inline Box

`<span>` has no meaning at all on its own. It is a blank container you use to group inline text so you can style it later with CSS.

```html
<p>My favorite color is <span class="highlight">teal</span>.</p>
```

### More Useful Inline Tags

- `<mark>` highlights text, like a yellow highlighter pen.
- `<small>` shows fine print, like a copyright line.
- `<sub>` makes subscript text (lower and smaller), as in H<sub>2</sub>O.
- `<sup>` makes superscript text (higher and smaller), as in E = mc<sup>2</sup>.

```html
<p>Water is H<sub>2</sub>O. The area is 5 m<sup>2</sup>.</p>
<p><mark>Remember this!</mark></p>
<p><small>Copyright 2026, all rights reserved.</small></p>
```

## Quotations

Use `<blockquote>` for a long quote that takes up its own block. Browsers usually indent it.

```html
<blockquote>
  The only way to do great work is to love what you do.
</blockquote>
```

Use `<q>` for a short quote inside a sentence. Browsers automatically add quotation marks around it.

```html
<p>She said <q>let's build a website today</q>, and we did.</p>
```

## Showing Code

To show code or text exactly as you typed it — including spaces and line breaks — use `<pre>`. Inside `<pre>`, whitespace is **not** collapsed.

```html
<pre>
function hello() {
    console.log("Hi!");
}
</pre>
```

For a short snippet inside a sentence, use `<code>`.

```html
<p>Use the <code>console.log()</code> function to print messages.</p>
```

Often `<pre>` and `<code>` are combined for a block of code:

```html
<pre><code>const x = 10;
const y = 20;</code></pre>
```

## HTML Entities

Some characters are tricky to type because they have special meaning in HTML. For example, the browser would read a `<` as the start of a tag. To show such characters as plain text, we use **HTML entities**: short codes that start with `&` and end with `;`.

```text
&amp;   shows  &   (ampersand)
&lt;    shows  <   (less-than)
&gt;    shows  >   (greater-than)
&copy;  shows  ©   (copyright symbol)
&nbsp;  shows  a non-breaking space
```

Here they are in use:

```html
<p>Tom &amp; Jerry</p>
<p>To write a tag, type &lt;p&gt; in your editor.</p>
<p>&copy; 2026 My Website</p>
<p>Price:&nbsp;$5</p>
```

A **non-breaking space** (`&nbsp;`) keeps two words glued together on the same line, so they never split across two lines.

## Common Mistakes

- Using more than one `<h1>` on a page.
- Skipping heading levels (jumping from `<h2>` to `<h4>`).
- Choosing a heading by how big it looks instead of what it means.
- Using many `<br>` tags to add space instead of separate paragraphs.
- Expecting extra spaces or blank lines in your code to show on the page.
- Typing a real `<` or `&` in text instead of using `&lt;` or `&amp;`.

## Exercises

1. Build a mini blog post: one `<h1>`, two `<h2>` sections, and a paragraph under each.
2. Write a two-line poem using a single `<p>` with a `<br>` between the lines.
3. Write a sentence that uses `<strong>`, `<em>`, and `<mark>` each at least once.
4. Show the chemical formula for carbon dioxide (CO2) using `<sub>` correctly.
5. Display the text `5 < 10 & 10 > 5` on the page using the correct HTML entities.

## Key Takeaways

- Use one `<h1>` per page and keep headings in order.
- `<p>` makes paragraphs; the browser collapses extra whitespace.
- `<br>` forces a line break and `<hr>` draws a divider.
- `<strong>` and `<em>` carry meaning; `<b>` and `<i>` are just styling.
- `<mark>`, `<small>`, `<sub>`, and `<sup>` handle special inline needs.
- Use `<blockquote>`, `<q>`, `<pre>`, and `<code>` for quotes and code.
- HTML entities like `&amp;` and `&lt;` let you show special characters.

---

[← Back to Course Index](../README.md) | [Next: Links and Navigation →](03-links-and-navigation.md)
