# Typography

**Typography** is the art of arranging text so it looks good and reads easily. Good typography is quiet: people do not notice it, they just enjoy reading. In this lesson you will learn the CSS properties that control fonts, sizing, spacing, and alignment, and how to use them to style real text.

## What You'll Learn

- How to set fonts with `font-family` and font stacks
- The difference between web-safe fonts and web fonts (like Google Fonts)
- Sizing and weighting text with `font-size`, `font-weight`, and `font-style`
- Spacing text with `line-height` and `letter-spacing`
- Aligning, decorating, and transforming text
- The handy `font` shorthand
- Why a comfortable line length matters

## font-family and Font Stacks

The `font-family` property sets the typeface for text.

```css
p {
  font-family: Arial;
}
```

But there is a catch. Not every computer has every font. If a font is missing, the text falls back to the browser's default, which might look wrong. The solution is a **font stack**: a list of fonts, in order of preference.

```css
body {
  font-family: "Helvetica Neue", Arial, sans-serif;
}
```

The browser tries each font in turn. It uses Helvetica Neue if available, otherwise Arial, otherwise any sans-serif font. The last item should always be a **generic family** as a safety net.

The generic families are:

- `serif` — fonts with little feet on the letters (like Times New Roman). Traditional, formal.
- `sans-serif` — clean fonts without the feet (like Arial). Modern, common for screens.
- `monospace` — every character is the same width (like Courier). Used for code.

Wrap any font name that contains spaces in quotes, like `"Helvetica Neue"`.

## Web-Safe Fonts vs Web Fonts

**Web-safe fonts** are fonts that come pre-installed on almost every device, so you can count on them. Examples include Arial, Helvetica, Times New Roman, Georgia, and Courier New. They are reliable but limited in variety.

**Web fonts** are fonts you load from the internet, so you are not limited to what is already installed. The most popular source is **Google Fonts**, which is free.

To use a Google Font, add a `<link>` in your HTML `<head>`:

```html
<link
  rel="stylesheet"
  href="https://fonts.googleapis.com/css2?family=Roboto&display=swap"
/>
```

Then use it in your CSS like any other font:

```css
body {
  font-family: "Roboto", sans-serif;
}
```

Always keep a fallback (here, `sans-serif`) in case the web font fails to load.

## font-size

The `font-size` property sets how big the text is.

```css
p {
  font-size: 16px;
}

h1 {
  font-size: 2rem; /* twice the root font size */
}
```

As you learned earlier, `rem` is a great choice for font sizes because it respects the user's settings and scales nicely. A common base size for body text is 16px (or `1rem`).

## font-weight

The `font-weight` property controls how bold or thin the text is.

```css
.bold {
  font-weight: bold;
}

.thin {
  font-weight: 300;
}
```

You can use keywords (`normal`, `bold`) or numbers from 100 (thin) to 900 (heavy). `400` is normal and `700` is bold. Not every font supports every weight; the browser uses the closest one it has.

## font-style

The `font-style` property is mainly used to make text *italic*.

```css
.quote {
  font-style: italic;
}
```

The values are `normal` (the default) and `italic`.

## line-height

**Line height** is the vertical space between lines of text. Good line height makes paragraphs easier to read because the lines are not cramped together.

```css
p {
  line-height: 1.6;
}
```

A unitless number like `1.6` is the best choice. It means 1.6 times the font size, and it scales correctly with the text. For body text, something between `1.4` and `1.8` is comfortable. Headings usually look better tighter, around `1.1` to `1.3`.

## letter-spacing

**Letter spacing** adds or removes space between individual characters.

```css
h1 {
  letter-spacing: 2px;
}

.tight {
  letter-spacing: -0.5px;
}
```

A small amount of extra spacing can make uppercase headings feel more elegant. Use it sparingly; too much hurts readability.

## text-align

The `text-align` property controls horizontal alignment of text within its box.

```css
.center { text-align: center; }
.right { text-align: right; }
.justify { text-align: justify; }
```

The values:

- `left` — the default for most languages.
- `right` — text hugs the right edge.
- `center` — text is centered.
- `justify` — lines stretch to fill the full width, making both edges straight.

Use `center` for headings and short bits. Avoid `justify` for narrow columns, since it can create awkward gaps.

## text-decoration

The `text-decoration` property adds or removes lines on text, like underlines.

```css
a {
  text-decoration: none; /* remove the underline from links */
}

.strikethrough {
  text-decoration: line-through;
}

.underline {
  text-decoration: underline;
}
```

A very common use is `text-decoration: none` to remove the default underline from links, then add your own styling.

## text-transform

The `text-transform` property changes the capitalization of text without changing your HTML.

```css
.shout { text-transform: uppercase; }
.quiet { text-transform: lowercase; }
.title { text-transform: capitalize; }
```

- `uppercase` — MAKES ALL LETTERS CAPITAL.
- `lowercase` — makes all letters small.
- `capitalize` — Makes The First Letter Of Each Word Capital.

This is handy for styling: you can write normal text in your HTML and let CSS handle the look.

## color

The `color` property sets the text color, as you learned in the colors lesson.

```css
p {
  color: #333;
}
```

Dark gray (like `#333`) is often easier on the eyes than pure black for long passages of text.

## The font Shorthand

The `font` shorthand lets you set several font properties in one line. The order matters, and you must include at least the font size and family.

```css
p {
  font: italic bold 16px/1.6 "Helvetica Neue", Arial, sans-serif;
}
```

Reading this: italic style, bold weight, 16px size with a 1.6 line height, in the Helvetica Neue font stack. The size and line height are joined by a slash: `size/line-height`.

The shorthand is compact, but when you are starting out, it is often clearer to write each property on its own line. Use whichever feels readable to you.

## Readable Line Length

Here is a typography rule that matters a lot: lines that are too long are tiring to read, because the eye struggles to find the start of the next line. A comfortable line is about **50 to 75 characters**.

You can control this with `max-width`, and the `ch` unit makes it easy:

```css
.article {
  max-width: 65ch; /* about 65 characters per line */
  margin: 0 auto; /* center the column */
}
```

This single rule does more for readability than almost anything else.

## Putting It Together: Styling an Article

Let's style a short article using what we learned.

```html
<link
  rel="stylesheet"
  href="https://fonts.googleapis.com/css2?family=Merriweather&display=swap"
/>

<article class="article">
  <h1>The Joy of Reading</h1>
  <p class="lead">A short article styled with good typography.</p>
  <p>
    Good typography makes text easy and pleasant to read. With the right font,
    size, spacing, and line length, your words feel effortless to follow.
  </p>
</article>
```

```css
body {
  font-family: "Merriweather", Georgia, serif;
  color: #333;
}

.article {
  max-width: 65ch;
  margin: 40px auto;
  line-height: 1.7;
}

h1 {
  font-size: 2.5rem;
  font-weight: 700;
  line-height: 1.2;
  text-align: center;
}

.lead {
  font-size: 1.25rem;
  font-style: italic;
  color: #555;
}

p {
  font-size: 1.1rem;
  margin-bottom: 1.2em;
}
```

This pulls together a web font, a readable line length, generous line height, and a clear hierarchy between the heading, the lead, and the body. The result is text that is a pleasure to read.

## Common Mistakes

- **No fallback font.** Always end a font stack with a generic family like `sans-serif`.
- **Forgetting quotes** around font names that contain spaces, like `"Times New Roman"`.
- **Using a unit on `line-height`.** A unitless number like `1.6` scales best.
- **Lines that are too long.** Without a `max-width`, text can stretch across the whole screen and become hard to read.
- **Overusing `text-transform: uppercase`.** All-caps is fine for short headings but tiring for long text.

## Exercises

1. Set up a font stack with three fonts ending in a generic family. Confirm the text still renders if the first font is misspelled.
2. Load a Google Font with a `<link>` and apply it to your page body, keeping a fallback.
3. Style a heading with a custom `font-size`, `font-weight`, and a tight `line-height`. Style body paragraphs with a comfortable `line-height` of 1.6.
4. Remove the underline from all links with `text-decoration: none`, then give them a color instead.
5. Limit an article to `max-width: 65ch` and center it. Compare how it reads against full-width text.

## Key Takeaways

- `font-family` sets the typeface; always provide a fallback stack ending in a generic family.
- Web-safe fonts are pre-installed and reliable; web fonts (like Google Fonts) give more variety via a `<link>`.
- Size and style text with `font-size` (prefer `rem`), `font-weight`, and `font-style`.
- Space text with `line-height` (use a unitless number) and `letter-spacing`.
- Align, decorate, and transform text with `text-align`, `text-decoration`, and `text-transform`.
- The `font` shorthand sets several properties at once; size and family are required.
- Keep lines about 50 to 75 characters for comfortable reading, often with `max-width` and `ch`.

---

[← Back to Course Index](../README.md) | [Next: Backgrounds and Borders →](06-backgrounds-and-borders.md)
