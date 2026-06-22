# Colors and Units

Colors bring a page to life, and units control sizes and spacing. In this lesson you will learn the different ways to write colors in CSS, and the difference between fixed sizes and flexible ones. Getting comfortable with both makes everything else in CSS easier.

## What You'll Learn

- The different ways to write colors: keywords, hex, rgb, and hsl
- How to add transparency with `rgba`, `hsla`, and opacity
- The difference between `color` and `background-color`
- Absolute units like `px` and relative units like `em`, `rem`, and `%`
- Viewport units `vw` and `vh`, and the `ch` unit
- When to use `rem` instead of `px`
- Why relative units help your pages adapt to different screens

## Colors

CSS gives you several ways to describe a color. They all do the same job; they just use different formats. Let's go through them.

### Color Keywords

The easiest way is a **color keyword**: a plain English name.

```css
h1 {
  color: red;
}

p {
  color: darkslategray;
}
```

There are about 140 named colors, including `red`, `blue`, `green`, `tomato`, `gold`, `white`, and `black`. Keywords are great for quick work, but they are limited. For exact colors, use one of the formats below.

### Hex Colors

A **hex color** (short for hexadecimal) starts with a `#` followed by six characters. These describe the amount of red, green, and blue.

```css
h1 {
  color: #ff0000; /* pure red */
}

p {
  color: #333333; /* dark gray */
}
```

The six characters come in three pairs: `#rrggbb`. The first pair is red, the second is green, the third is blue. Each pair ranges from `00` (none) to `ff` (full).

- `#000000` is black (no color)
- `#ffffff` is white (full of every color)
- `#ff0000` is red, `#00ff00` is green, `#0000ff` is blue

**Shorthand hex.** If each pair is a doubled character, you can write just three characters. `#ffffff` becomes `#fff`, and `#ff0000` becomes `#f00`. The browser expands each character by repeating it.

```css
body {
  color: #333; /* same as #333333 */
}
```

### rgb() and rgba()

The `rgb()` function sets a color using three numbers from 0 to 255: red, green, and blue.

```css
h1 {
  color: rgb(255, 0, 0); /* red */
}
```

The `rgba()` function adds a fourth value: **alpha**, which controls transparency. Alpha goes from `0` (fully see-through) to `1` (fully solid).

```css
.overlay {
  background-color: rgba(0, 0, 0, 0.5); /* black at 50% transparency */
}
```

This is perfect for tinted overlays, where you want to see what is behind the color.

### hsl() and hsla()

The `hsl()` function describes color in a way that is easy for humans to reason about: **hue**, **saturation**, and **lightness**.

```css
h1 {
  color: hsl(0, 100%, 50%); /* red */
}
```

- **Hue** is the color itself, given as a number from 0 to 360 (a color wheel). 0 is red, 120 is green, 240 is blue.
- **Saturation** is how vivid the color is, from `0%` (gray) to `100%` (full color).
- **Lightness** is how bright it is, from `0%` (black) to `100%` (white). 50% is the pure color.

`hsl` makes it easy to create shades. Want a lighter red? Just raise the lightness:

```css
.light-red {
  color: hsl(0, 100%, 75%); /* a softer pink-red */
}
```

Like `rgba`, `hsla()` adds an alpha value for transparency:

```css
.tint {
  background-color: hsla(200, 100%, 50%, 0.3);
}
```

### Opacity

The `opacity` property makes a whole element see-through, from `0` (invisible) to `1` (solid).

```css
.faded {
  opacity: 0.4;
}
```

Note the difference: `opacity` affects the **entire element**, including its text and children. The alpha in `rgba`/`hsla` affects only the color you applied it to. If you want just a transparent background but solid text, use `rgba` on the background, not `opacity` on the element.

## color vs background-color

Two properties are easy to mix up:

- `color` sets the **text** color.
- `background-color` sets the **background** behind the element.

```css
.button {
  color: white; /* text is white */
  background-color: #2a7ae2; /* background is blue */
}
```

```html
<button class="button">Click me</button>
```

Always check that your text color and background color have enough contrast. Light gray text on a white background is hard to read.

## Units

Many CSS properties need a size: font sizes, widths, padding, margins, and more. Sizes use **units**. There are two families: absolute and relative.

### Absolute Units: px

The most common absolute unit is the **pixel**, written `px`. A pixel is a fixed dot on the screen. 16px is always 16px.

```css
p {
  font-size: 16px;
  margin: 20px;
}
```

Pixels are predictable and easy to understand. The downside is that they do not adapt. If a user increases their browser's default font size for readability, fixed pixel sizes ignore that choice.

### Relative Units

**Relative units** are sized in relation to something else. This makes them flexible.

**em** is relative to the font size of the current element. If an element's font size is 20px, then `1em` is 20px, `2em` is 40px, and `0.5em` is 10px.

```css
.box {
  font-size: 20px;
  padding: 1em; /* 20px, because 1em = the element's font size */
}
```

One catch with `em`: it can compound when nested, because each child's `em` is based on its own (possibly inherited) font size. This can lead to surprises.

**rem** is relative to the *root* font size, which is the `<html>` element's font size (16px by default). Because it always points to the same root, `rem` does not compound. This makes it predictable.

```css
html {
  font-size: 16px;
}

h1 {
  font-size: 2rem; /* 32px */
}

p {
  font-size: 1rem; /* 16px */
}
```

**%** (percent) is relative to the parent element. A width of `50%` means half the parent's width.

```css
.half {
  width: 50%;
}
```

### Viewport Units: vw and vh

The **viewport** is the visible area of the browser window.

- `1vw` is 1% of the viewport's **width**.
- `1vh` is 1% of the viewport's **height**.

```css
.hero {
  width: 100vw; /* full width of the window */
  height: 100vh; /* full height of the window */
}
```

These are handy for full-screen sections that should fill the window no matter its size.

### The ch Unit

The **ch** unit is roughly the width of the character "0" in the current font. It is useful for controlling text width.

```css
.article {
  max-width: 65ch; /* about 65 characters wide */
}
```

A line of text around 50 to 75 characters is comfortable to read, and `ch` makes that easy to set.

## rem vs px: Which to Use

A common question is whether to use `rem` or `px`. Here is simple guidance:

- Use **rem** for font sizes and most spacing. If a user changes their browser's default font size, your layout scales with them. This is better for accessibility.
- Use **px** for things that should stay fixed, like a thin border (`1px solid`) or fine details.

A popular trick: keep the root font size at 16px, then think of `rem` in those terms. `1rem` = 16px, `1.5rem` = 24px, `2rem` = 32px.

## Why Relative Units Help Responsiveness

A **responsive** site adapts to different screen sizes, from phones to large monitors. Relative units are a big part of how that works.

If you size a column at `50%`, it stays half the parent no matter the screen width. If you size text in `rem`, it respects the user's settings. Fixed pixels cannot adapt this way. By leaning on relative units, your page bends to fit instead of breaking.

```css
.container {
  width: 90%; /* always 90% of the parent, on any screen */
  max-width: 1200px; /* but never wider than 1200px */
  margin: 0 auto; /* centered */
}
```

This pattern, a percentage width with a pixel max-width, is everywhere in responsive design.

## Common Mistakes

- **Forgetting the unit.** `font-size: 16;` is invalid. You need `16px` (line-height is the main exception, where a plain number is fine).
- **Confusing `color` and `background-color`.** `color` is text; `background-color` is behind it.
- **Using `opacity` when you wanted a transparent background.** `opacity` fades the whole element, text and all. Use `rgba` for a transparent background only.
- **Mixing up hex pairs.** Remember the order is red, green, blue: `#rrggbb`.
- **Hard-coding everything in `px`.** It works, but relative units adapt better.

## Exercises

1. Write the same color (a medium blue) four ways: as a keyword, hex, `rgb()`, and `hsl()`. Apply each to a different element and confirm they look the same.
2. Create a dark overlay using `rgba()` with 50% transparency over an image or colored box.
3. Set the root font size to 16px, then size a heading at `2rem` and a paragraph at `1rem`. Confirm the heading is twice the paragraph.
4. Build a box that is `100vw` wide and `100vh` tall so it fills the screen.
5. Limit a paragraph's width with `max-width: 60ch` and notice how the line length becomes easier to read.

## Key Takeaways

- Colors can be written as keywords, hex (`#rrggbb`), `rgb()`, or `hsl()`.
- `rgba()` and `hsla()` add an alpha value for transparency; `opacity` fades the whole element.
- `color` styles text; `background-color` styles the area behind it.
- Absolute units like `px` are fixed; relative units like `em`, `rem`, and `%` adapt.
- `rem` is based on the root font size and is predictable; prefer it for fonts and spacing.
- Viewport units `vw`/`vh` size things to the window; `ch` sizes to character width.
- Relative units are key to building responsive pages that fit any screen.

---

[← Back to Course Index](../README.md) | [Next: The Box Model →](04-the-box-model.md)
