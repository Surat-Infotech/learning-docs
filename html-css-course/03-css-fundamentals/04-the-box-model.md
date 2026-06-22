# The Box Model

Here is a secret that unlocks CSS layout: every element on a page is a rectangular box. Once you understand how those boxes are built, spacing and sizing finally make sense. This is one of the most important lessons in the whole course, so take your time with it.

## What You'll Learn

- Why every element is a box
- The four layers of the box: content, padding, border, and margin
- How `width` and `height` work
- Padding and margin shorthand with 1 to 4 values
- What margin collapsing is
- The difference between `content-box` and `border-box`
- Why `box-sizing: border-box` is the standard fix
- How `overflow` controls content that does not fit

## Everything Is a Box

When the browser draws a page, it treats each element, a heading, a paragraph, an image, a button, as a rectangular box. Even text sits inside a box. Layout in CSS is really just arranging these boxes.

Each box is made of four layers, from the inside out:

1. **Content** — the actual text or image.
2. **Padding** — space *inside* the box, between the content and the border.
3. **Border** — a line around the padding.
4. **Margin** — space *outside* the box, pushing other boxes away.

Here is the box model drawn out:

```text
+---------------------------------------+
|              MARGIN                   |
|   +-------------------------------+   |
|   |           BORDER              |   |
|   |   +-----------------------+   |   |
|   |   |       PADDING         |   |   |
|   |   |   +---------------+   |   |   |
|   |   |   |   CONTENT     |   |   |   |
|   |   |   +---------------+   |   |   |
|   |   +-----------------------+   |   |
|   +-------------------------------+   |
+---------------------------------------+
```

A helpful way to picture it: think of a framed photo on a wall.

- The **content** is the photo.
- The **padding** is the white mat between the photo and the frame.
- The **border** is the frame.
- The **margin** is the empty wall space around the frame, keeping other frames at a distance.

## Width and Height

The `width` and `height` properties set the size of the **content** area (we will see how border and padding affect the total size below).

```css
.box {
  width: 300px;
  height: 150px;
}
```

This makes the content area 300px wide and 150px tall.

You can also set `max-width`, `min-width`, `max-height`, and `min-height` to set limits.

```css
.card {
  width: 100%;
  max-width: 400px; /* grows to fill, but never past 400px */
}
```

## Padding

**Padding** is the space inside the box, between the content and the border. Use it to give content room to breathe.

```css
.box {
  padding: 20px;
}
```

This adds 20px of space on all four sides.

You can set each side on its own:

```css
.box {
  padding-top: 10px;
  padding-right: 20px;
  padding-bottom: 10px;
  padding-left: 20px;
}
```

### Padding Shorthand (1 to 4 values)

The `padding` shorthand lets you set sides quickly. The number of values changes the meaning:

```css
.a { padding: 20px; }              /* all four sides: 20px */
.b { padding: 10px 20px; }         /* top/bottom: 10px, left/right: 20px */
.c { padding: 10px 20px 30px; }    /* top: 10px, left/right: 20px, bottom: 30px */
.d { padding: 10px 20px 30px 40px; } /* top, right, bottom, left */
```

For the four-value version, remember the order goes clockwise starting from the top: **top, right, bottom, left**. A trick to remember it: think of a clock, or the word **TRouBLe** (Top, Right, Bottom, Left).

## Border

A **border** is a line around the box, between the padding and the margin. We will cover borders in depth in a later lesson, but here is the basic shape:

```css
.box {
  border: 2px solid black;
}
```

This sets the border width (2px), style (solid), and color (black) all at once.

## Margin

**Margin** is the space outside the box. It pushes neighboring elements away.

```css
.box {
  margin: 30px;
}
```

Margin uses the same shorthand rules as padding (1 to 4 values, in the same TRouBLe order).

```css
.box {
  margin: 10px 20px; /* top/bottom: 10px, left/right: 20px */
}
```

A very common trick is centering a block horizontally with `auto` left and right margins:

```css
.container {
  width: 600px;
  margin: 0 auto; /* 0 top/bottom, auto left/right = centered */
}
```

The `auto` value tells the browser to split the leftover space evenly on both sides, which centers the box.

## Margin Collapsing

Here is a quirk that surprises beginners: **margin collapsing**. When two vertical margins meet, top-to-bottom, they do not add up. Instead, the browser uses the larger of the two and ignores the smaller.

```css
.first {
  margin-bottom: 30px;
}

.second {
  margin-top: 20px;
}
```

```html
<p class="first">First paragraph</p>
<p class="second">Second paragraph</p>
```

You might expect 50px of space between them (30 + 20). But because of margin collapsing, the gap is just **30px**, the larger of the two.

A few things to know:

- Collapsing happens only with **vertical** margins (top and bottom), not horizontal ones.
- Padding does not collapse. If you want predictable spacing, padding is sometimes simpler.

This behavior is normal, not a bug. Once you know it exists, it stops being confusing.

## box-sizing: The Big One

Now for the most important idea in this lesson. By default, when you set `width`, that width applies only to the **content**. Padding and border are added *on top*, making the box bigger than you asked for.

This default is called **content-box**.

```css
.box {
  width: 200px;
  padding: 20px;
  border: 5px solid black;
}
```

How wide is this box on screen? You set `width: 200px`, but the real width is:

```text
200px (content)
+ 20px (left padding) + 20px (right padding)
+ 5px (left border) + 5px (right border)
= 250px total
```

That is confusing. You said 200px, but you got 250px. As you build layouts, this off-by-padding math causes endless headaches.

### The Fix: border-box

CSS has a better way. Set `box-sizing: border-box` and the `width` now includes the padding and border. The content shrinks to make room, so the total box stays exactly what you set.

```css
.box {
  box-sizing: border-box;
  width: 200px;
  padding: 20px;
  border: 5px solid black;
}
```

Now the box is exactly **200px** wide on screen. The padding and border fit inside. Much easier to reason about.

### The Universal Reset

Because `border-box` is so much easier, almost every modern project turns it on for every element. The standard way is a universal selector reset at the top of your stylesheet:

```css
* {
  box-sizing: border-box;
}
```

This single rule applies `border-box` to everything. Many developers start every project with it. From here on, `width` means the real, total width, which is exactly what you want.

Here is a fuller, common reset many people use:

```css
*,
*::before,
*::after {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}
```

This also clears the default margins and padding that browsers add, giving you a clean slate.

## Overflow

What happens when content is too big for its box? That is where **overflow** comes in. The `overflow` property decides what to do with content that spills out.

```css
.box {
  width: 200px;
  height: 100px;
  overflow: hidden;
}
```

Common values:

- `visible` (the default) — extra content spills outside the box and is still shown.
- `hidden` — extra content is clipped and hidden.
- `scroll` — scrollbars are always shown so you can scroll to see the rest.
- `auto` — scrollbars appear only when needed.

```css
.scroll-area {
  height: 150px;
  overflow: auto; /* scroll only if the content is too tall */
}
```

`overflow: auto` is the friendliest choice: it adds a scrollbar only when there is too much content.

## A Worked Example

Let's put the box model together. Here is a styled card:

```html
<div class="card">
  <h2>Box Model Card</h2>
  <p>This card uses padding, border, and margin together.</p>
</div>
```

```css
* {
  box-sizing: border-box;
}

.card {
  width: 300px;
  padding: 20px; /* space inside, around the text */
  border: 1px solid #ccc; /* a thin frame */
  margin: 40px auto; /* space outside, centered horizontally */
}
```

Because of `border-box`, the card is exactly 300px wide, even with padding and a border. The padding keeps the text off the edges. The margin pushes the card away from other elements and centers it.

## Common Mistakes

- **Forgetting box-sizing.** Without `border-box`, padding and border make boxes wider than you set, breaking layouts.
- **Mixing up padding and margin.** Padding is *inside* the box (space around content); margin is *outside* (space between boxes).
- **Expecting vertical margins to add up.** They collapse to the larger value.
- **Getting the shorthand order wrong.** Four values go clockwise: top, right, bottom, left.
- **Trying to center text with `margin: auto`.** That centers a block element, not its text. Use `text-align: center` for text.

## Exercises

1. Create a box and give it different padding on each side using the four-value shorthand. Confirm the order is top, right, bottom, left.
2. Build two stacked paragraphs with a `margin-bottom` of 40px on the first and `margin-top` of 20px on the second. Measure the gap and confirm it is 40px, not 60px (margin collapsing).
3. Make a box that is `width: 200px` with `20px` padding and a `5px` border, first without `box-sizing` and then with `border-box`. Compare the real widths.
4. Add the universal reset `* { box-sizing: border-box; }` to a page and notice how sizing becomes more predictable.
5. Create a box with a fixed height and content that is too tall. Try each `overflow` value (`visible`, `hidden`, `scroll`, `auto`) and watch what changes.

## Key Takeaways

- Every element is a box made of content, padding, border, and margin.
- Padding is space inside the box; margin is space outside it.
- The padding and margin shorthands take 1 to 4 values, clockwise from the top.
- Vertical margins collapse to the larger of the two values.
- By default, `width` covers only the content (content-box), so padding and border add to the size.
- `box-sizing: border-box` makes `width` include padding and border; the universal reset `* { box-sizing: border-box; }` is the standard fix.
- `overflow` controls content that does not fit, with `auto` adding scrollbars only when needed.

---

[← Back to Course Index](../README.md) | [Next: Typography →](05-typography.md)
