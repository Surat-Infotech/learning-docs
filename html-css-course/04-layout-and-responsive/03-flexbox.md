# Flexbox

Flexbox is one of the most useful tools in all of CSS. It makes arranging elements in a row or a column simple, and it solves problems that used to be painful: centering things, spacing them evenly, and making cards the same height. In this lesson you will learn Flexbox step by step with many small examples.

## What You'll Learn

- What problem Flexbox solves (one-dimensional layouts)
- How to turn on Flexbox with `display: flex`
- The **main axis** and **cross axis**, and how `flex-direction` sets them
- Spacing items with `justify-content`, aligning with `align-items` and `align-self`
- Letting items wrap with `flex-wrap` and spacing them with `gap`
- The `flex` shorthand: `flex-grow`, `flex-shrink`, and `flex-basis`
- Reordering items with `order`
- Common real patterns: nav bars, centering, equal-height cards, and card rows

## What Flexbox Solves

Flexbox is built for **one-dimensional layouts**, meaning a single row **or** a single column. If you want a row of buttons, a navigation bar, or a centered box, Flexbox is the right tool. (For two-dimensional grids of rows *and* columns at once, you will use CSS Grid in the next lesson.)

The element you apply `display: flex` to is the **flex container**. Its direct children automatically become **flex items**.

```css
.container {
  display: flex;
}
```

```html
<div class="container">
  <div>One</div>
  <div>Two</div>
  <div>Three</div>
</div>
```

Just by adding `display: flex`, the three `<div>` elements line up in a row instead of stacking. That is the magic.

## The Two Axes: Main and Cross

Flexbox works with two directions, called **axes**:

- The **main axis** is the direction items flow along (by default, left to right).
- The **cross axis** is the direction across the main axis (by default, top to bottom).

This matters because the alignment properties refer to these axes, not to "horizontal" and "vertical" directly. When you change the direction, the axes swap too.

```text
Default (row):
  main axis  →  →  →  (horizontal)
  cross axis ↓        (vertical)
```

## `flex-direction`

The `flex-direction` property sets which way the main axis points.

```css
.container {
  display: flex;
  flex-direction: row;    /* default: left to right */
}
```

The four values:

- `row` — left to right (the default)
- `row-reverse` — right to left
- `column` — top to bottom
- `column-reverse` — bottom to top

```css
.stack {
  display: flex;
  flex-direction: column; /* items stack vertically */
}
```

When you use `column`, the main axis becomes vertical and the cross axis becomes horizontal. Keep this in mind when you align things.

## `justify-content`: Spacing Along the Main Axis

`justify-content` controls how items are spaced **along the main axis**. With the default `row` direction, this means horizontal spacing.

```css
.container {
  display: flex;
  justify-content: center;
}
```

The common values:

- `flex-start` — pack items at the start (default)
- `flex-end` — pack items at the end
- `center` — center the items
- `space-between` — first and last items touch the edges; equal gaps between
- `space-around` — equal space around each item
- `space-evenly` — equal space between and around every item

```text
flex-start:    [A][B][C]..............
center:        ......[A][B][C].........
flex-end:      ..............[A][B][C]
space-between: [A].....[B].....[C]
space-evenly:  ...[A]...[B]...[C]...
```

A navigation bar with the logo on the left and links on the right is just:

```css
nav {
  display: flex;
  justify-content: space-between;
  align-items: center;
}
```

```html
<nav>
  <div class="logo">MySite</div>
  <div class="links">Home About Contact</div>
</nav>
```

## `align-items`: Aligning Along the Cross Axis

`align-items` controls how items line up **along the cross axis**. With `row` direction, the cross axis is vertical, so this controls vertical alignment.

```css
.container {
  display: flex;
  align-items: center;
}
```

The common values:

- `stretch` — items stretch to fill the container's height (default)
- `flex-start` — align to the top (of the cross axis)
- `flex-end` — align to the bottom
- `center` — center vertically
- `baseline` — line up the text baselines

### Centering Anything

The most loved Flexbox trick is centering a box both horizontally and vertically. Combine `justify-content` (main axis) with `align-items` (cross axis):

```css
.center-screen {
  display: flex;
  justify-content: center; /* horizontal */
  align-items: center;     /* vertical */
  height: 300px;
  border: 1px solid #ccc;
}
```

```html
<div class="center-screen">
  <div class="box">I am perfectly centered</div>
</div>
```

This used to be a famously hard CSS problem. With Flexbox it is two lines.

## `align-self`: Overriding One Item

`align-items` aligns *all* items. If you want one item to align differently, use `align-self` on that single item. It overrides `align-items` just for that one.

```css
.container {
  display: flex;
  align-items: flex-start;
}

.special {
  align-self: flex-end; /* this one item drops to the bottom */
}
```

```html
<div class="container">
  <div>Top</div>
  <div class="special">Bottom</div>
  <div>Top</div>
</div>
```

## `flex-wrap`: Letting Items Wrap

By default, flex items try to fit on one line, even if they get squished. With `flex-wrap: wrap`, items that do not fit move to the next line.

```css
.container {
  display: flex;
  flex-wrap: wrap;
}
```

The values:

- `nowrap` — everything on one line (default)
- `wrap` — items wrap onto new lines as needed
- `wrap-reverse` — wraps in the opposite direction

This is essential for responsive card layouts that should reflow on small screens.

## `gap`: Space Between Items

The `gap` property adds even spacing between flex items without fiddly margins. It only adds space *between* items, not on the outer edges.

```css
.container {
  display: flex;
  gap: 16px; /* 16px between every item */
}
```

You can also set row and column gaps separately:

```css
.container {
  display: flex;
  flex-wrap: wrap;
  gap: 12px 20px; /* row-gap column-gap */
}
```

`gap` is the modern, clean way to space items. Prefer it over adding margins to each child.

## The `flex` Shorthand

So far we have controlled the container. Now let's control how individual items grow and shrink. Each flex item has three properties, usually written together with the `flex` shorthand:

- **`flex-grow`** — how much an item grows to fill extra space (a number; default `0`)
- **`flex-shrink`** — how much an item shrinks when space is tight (a number; default `1`)
- **`flex-basis`** — the item's starting size before growing or shrinking (a length; default `auto`)

```css
.item {
  flex: 1 1 200px; /* grow shrink basis */
}
```

### `flex: 1` Means "Share Space Equally"

The most common shorthand you will write is `flex: 1`. It means grow to fill space, with all such items sharing equally.

```css
.column {
  flex: 1; /* each column takes an equal share of the width */
}
```

```html
<div class="container">
  <div class="column">A</div>
  <div class="column">B</div>
  <div class="column">C</div>
</div>
```

Each column gets one-third of the width. If you gave one item `flex: 2`, it would take twice the share of the others.

### `flex-basis` Sets the Starting Width

`flex-basis` is like setting a width that Flexbox is allowed to adjust:

```css
.sidebar {
  flex: 0 0 240px; /* don't grow, don't shrink, stay 240px */
}

.main {
  flex: 1; /* take all remaining space */
}
```

```html
<div class="container">
  <aside class="sidebar">Sidebar</aside>
  <main class="main">Main content</main>
</div>
```

This makes a fixed-width sidebar next to a flexible main area, a classic layout.

## `order`: Reordering Items

The `order` property lets you change the visual order of items without changing the HTML. Items are laid out from lowest `order` to highest. The default is `0`.

```css
.first {
  order: -1; /* moves before items with the default order of 0 */
}
```

```html
<div class="container">
  <div>A</div>
  <div class="first">B</div>
  <div>C</div>
</div>
```

Now B appears first: B, A, C. Use `order` sparingly, because it can confuse people using screen readers if the visual and code order drift too far apart.

## Putting It Together: Common Patterns

### Equal-Height Cards in a Row

Because `align-items` defaults to `stretch`, flex items in a row automatically become the same height as the tallest one, even if their content differs.

```css
.cards {
  display: flex;
  gap: 16px;
}

.card {
  flex: 1;
  padding: 16px;
  border: 1px solid #ddd;
}
```

```html
<div class="cards">
  <div class="card">Short text.</div>
  <div class="card">A much longer block of text that takes more lines.</div>
  <div class="card">Medium amount of text here.</div>
</div>
```

All three cards line up at the same height with no extra effort.

### A Responsive Card Row

Combine `flex-wrap`, `gap`, and `flex-basis` so cards sit in a row but wrap onto new lines when the screen shrinks:

```css
.grid {
  display: flex;
  flex-wrap: wrap;
  gap: 16px;
}

.grid .card {
  flex: 1 1 200px; /* aim for ~200px, grow to fill, wrap when too tight */
}
```

On a wide screen you might see four cards per row; on a phone they stack. No media queries needed.

## Common Mistakes

- Applying `justify-content` and `align-items` and confusing the two. Remember: `justify-content` follows the main axis, `align-items` follows the cross axis.
- Forgetting that when `flex-direction: column`, the axes swap, so `justify-content` now controls vertical spacing.
- Using margins between items when `gap` would be cleaner.
- Expecting Flexbox to handle a full two-dimensional grid. It handles one row or one column at a time. Use Grid for that.
- Writing `flex: 1` and then also setting a fixed `width`, which fight each other. Use `flex-basis` instead.

## Exercises

1. Build a navigation bar with a logo on the left and links on the right using `display: flex` and `justify-content: space-between`.
2. Center a single box both horizontally and vertically inside a 400px-tall container using `justify-content` and `align-items`.
3. Create three cards with different amounts of text and confirm they all stretch to equal height. Then set `align-items: flex-start` and watch the heights differ.
4. Make a two-column layout with a fixed 250px sidebar (`flex: 0 0 250px`) and a flexible main area (`flex: 1`).
5. Build a row of cards with `flex-wrap: wrap` and `flex: 1 1 200px`. Resize the browser and watch them reflow onto new lines.

## Key Takeaways

- Flexbox handles **one-dimensional** layouts: a single row or column.
- `display: flex` makes a container; its direct children become flex items.
- The **main axis** follows `flex-direction`; the **cross axis** runs across it.
- `justify-content` spaces items along the main axis; `align-items` aligns along the cross axis.
- `align-self` overrides alignment for one item; `flex-wrap` lets items reflow.
- `gap` is the clean way to space items apart.
- The `flex` shorthand (`flex-grow flex-shrink flex-basis`) controls how items size; `flex: 1` shares space equally.

---

[← Back to Course Index](../README.md) | [Next: CSS Grid →](04-grid.md)
