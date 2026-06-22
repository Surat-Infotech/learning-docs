# Positioning

Sometimes you need to move an element to an exact spot: a badge in the corner of a card, a header that sticks to the top, or a button that floats in the corner of the screen. CSS **positioning** gives you that control. In this lesson you will learn the five position values and when to use each one.

## What You'll Learn

- The five values of the `position` property
- How `top`, `right`, `bottom`, and `left` move elements
- Why `absolute` positions relative to its nearest **positioned ancestor**
- How `z-index` controls which element sits in front
- Real patterns: badges, sticky headers, fixed buttons, and overlays
- When **not** to use positioning, and reach for Flexbox or Grid instead

## The `position` Property

Every element has a `position`, and by default it is `static`. The `position` property has five values:

- `static` (the default)
- `relative`
- `absolute`
- `fixed`
- `sticky`

The values `relative`, `absolute`, `fixed`, and `sticky` unlock four extra properties called **offsets**: `top`, `right`, `bottom`, and `left`. These offsets nudge the element away from an edge.

## `position: static`

This is the default. The element sits exactly where the normal flow puts it. The offset properties (`top`, `left`, and so on) do **nothing** on a static element.

```css
.box {
  position: static; /* the default; offsets are ignored */
}
```

You rarely write `static` yourself. It is mostly useful to know that it is the starting point.

## `position: relative`

A **relatively positioned** element stays in the normal flow, keeping its original space. But you can now nudge it from its original spot using offsets.

```css
.nudged {
  position: relative;
  top: 20px;   /* move 20px down from where it would be */
  left: 30px;  /* move 30px right from where it would be */
}
```

```html
<p>Normal paragraph.</p>
<p class="nudged">I am shifted down and to the right.</p>
<p>Another normal paragraph.</p>
```

Important: the gap where the element *would* have been stays empty. The element moves visually, but its reserved space does not.

`relative` has a second, very common job: it acts as an **anchor** for absolutely positioned children. You will see this in the next section.

## `position: absolute`

An **absolutely positioned** element is removed from the normal flow. It no longer takes up space, and other elements act as if it is not there. You then place it precisely using offsets.

The key question is: positioned relative to *what*? An absolute element is placed relative to its **nearest positioned ancestor**, meaning the closest parent (or grandparent) that has a `position` other than `static`. If there is no positioned ancestor, it positions relative to the whole page.

This is why you will constantly see this pattern: a parent with `position: relative` and a child with `position: absolute`.

```css
.card {
  position: relative; /* anchor for the badge */
  width: 200px;
  height: 120px;
  background: #eee;
}

.badge {
  position: absolute;
  top: 8px;
  right: 8px;
  background: crimson;
  color: white;
  padding: 4px 8px;
  border-radius: 999px;
}
```

```html
<div class="card">
  <span class="badge">New</span>
  Product name
</div>
```

The badge sits in the top-right corner of the card, because the card is the nearest positioned ancestor. If you removed `position: relative` from `.card`, the badge would jump to the top-right corner of the whole page instead.

## `position: fixed`

A **fixed** element is also removed from the flow, but it positions relative to the **viewport** (the visible browser window). It stays put even when you scroll the page.

This is perfect for a "back to top" button that floats in the corner:

```css
.back-to-top {
  position: fixed;
  bottom: 20px;
  right: 20px;
  padding: 12px 16px;
  background: #333;
  color: white;
  border-radius: 8px;
}
```

```html
<button class="back-to-top">↑ Top</button>
```

No matter how far down the user scrolls, the button stays glued to the bottom-right of the window.

## `position: sticky`

A **sticky** element is a hybrid. It behaves like a normal (relative) element until you scroll past a threshold, then it "sticks" in place like a fixed element. It is most often used for headers and section titles.

You must give it an offset (usually `top`) so the browser knows where to stick it.

```css
.sticky-header {
  position: sticky;
  top: 0;
  background: white;
  border-bottom: 1px solid #ddd;
  padding: 12px;
}
```

```html
<header class="sticky-header">My Site</header>
<main>
  <!-- lots of scrolling content -->
</main>
```

As you scroll, the header rides along until it reaches the top of the screen, then it stays pinned there. Sticky positioning works relative to the element's scrolling container, so the header sticks for as long as its parent is on screen.

## Stacking and `z-index`

When positioned elements overlap, which one appears in front? By default, elements that come later in the HTML stack on top of earlier ones. To take control, use **`z-index`**.

`z-index` accepts a number. A higher number sits in front of a lower number. It only works on **positioned** elements (anything that is not `static`).

```css
.behind {
  position: absolute;
  z-index: 1;
  background: skyblue;
}

.in-front {
  position: absolute;
  z-index: 2; /* higher number wins, sits on top */
  background: orange;
}
```

Think of `z-index` like layers in a stack of transparent sheets. The sheet with the highest number is closest to your eyes.

### Modal Overlay Example

A classic use of positioning and `z-index` together is a **modal**: a popup box over a dimmed background.

```css
.overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;            /* covers the whole viewport */
  background: rgba(0, 0, 0, 0.5); /* dim the page */
  z-index: 100;
}

.modal {
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%); /* perfectly centered */
  background: white;
  padding: 24px;
  border-radius: 8px;
  z-index: 101;        /* above the overlay */
}
```

```html
<div class="overlay"></div>
<div class="modal">
  <p>Are you sure?</p>
  <button>Yes</button>
  <button>No</button>
</div>
```

The overlay fills the screen and dims everything. The modal sits on top of it, centered. The `transform: translate(-50%, -50%)` trick pulls the box back by half its own width and height so its center lands on the screen's center.

## When NOT to Use Positioning

Positioning is great for small overlapping pieces: badges, tooltips, modals, sticky bars, and floating buttons. But it is the **wrong tool for general page layout**.

Years ago, people tried to build whole pages with `position: absolute`, carefully calculating pixel coordinates. It was fragile and broke easily when content changed or screens resized.

Today, for arranging rows of cards, navigation bars, sidebars, and page grids, you should use **Flexbox** and **CSS Grid** (your next lessons). They handle spacing, alignment, and resizing automatically.

A good rule of thumb:

- Need to overlap an element on top of another, or pin it to the screen? Use positioning.
- Need to arrange several elements into rows, columns, or a grid? Use Flexbox or Grid.

## Common Mistakes

- Forgetting to set `position: relative` on the parent, so an absolute child jumps to the corner of the whole page.
- Expecting offsets like `top` and `left` to work on a `static` element. They only work on positioned elements.
- Using `z-index` on a static element and wondering why it does nothing. The element must be positioned first.
- Relying on `z-index` to fix every overlap. Often the real fix is which element is positioned.
- Trying to build full page layouts with `absolute`, which is brittle. Use Flexbox or Grid instead.

## Exercises

1. Make a card with `position: relative` and place a "Sale" badge in its top-right corner using `position: absolute`. Then remove the parent's `position: relative` and observe the badge jump to the page corner.
2. Add a "Back to top" button with `position: fixed` in the bottom-right corner. Add enough page content to scroll and confirm the button stays put.
3. Create a sticky header with `position: sticky; top: 0;`. Scroll the page and watch it pin to the top.
4. Stack two overlapping absolute boxes and use `z-index` to control which one appears in front. Swap the values and see the order flip.
5. Build a simple modal: a full-screen dim overlay plus a centered white box, using `position: fixed` and `z-index`.

## Key Takeaways

- `position` has five values: `static`, `relative`, `absolute`, `fixed`, and `sticky`.
- Offsets (`top`, `right`, `bottom`, `left`) only work on positioned (non-static) elements.
- `relative` keeps the element in the flow and acts as an anchor for absolute children.
- `absolute` is removed from the flow and positions relative to the nearest positioned ancestor.
- `fixed` pins to the viewport; `sticky` pins after you scroll past a threshold.
- `z-index` controls stacking order on positioned elements; higher numbers sit in front.
- Use positioning for overlaps and pinned elements, not for general page layout.

---

[← Back to Course Index](../README.md) | [Next: Flexbox →](03-flexbox.md)
