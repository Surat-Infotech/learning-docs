# Responsive Design and Media Queries

People visit websites on phones, tablets, laptops, and giant monitors. A good website looks great on all of them. **Responsive design** is the craft of building one site that adapts to any screen size. In this lesson you will learn the tools that make it possible, and you will build a small responsive layout from start to finish.

## What You'll Learn

- What responsive design is and why it matters
- The viewport meta tag (a quick recap)
- **Mobile-first** thinking and why it leads to cleaner CSS
- Writing `@media` queries with `min-width` and `max-width`
- Fluid layouts using `%`, `fr`, `max-width`, and the `min()` and `clamp()` functions
- Making images and typography respond to screen size
- Showing and hiding elements per screen
- Testing your work in browser DevTools device mode

## What Is Responsive Design?

A **responsive** website automatically rearranges and resizes its content to fit the screen it is shown on. On a wide laptop you might see three columns of cards side by side. On a narrow phone those same cards stack into a single column you can scroll through.

The goal is one codebase that works everywhere, instead of building separate sites for phones and desktops. Responsive design is built from a few simple ideas: flexible sizes, and rules that change at certain screen widths.

## Recap: The Viewport Meta Tag

Before anything else, a responsive page needs this line in the `<head>` of the HTML:

```html
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
```

This tag tells mobile browsers to use the device's real width and not to zoom out. Without it, phones pretend to be about 980px wide and shrink your whole page to fit, which ruins your responsive layout. **Always include it.** It is the foundation everything else stands on.

## Mobile-First Thinking

**Mobile-first** means you design and write your base styles for small screens first, then add styles for larger screens on top. This sounds backwards, but it leads to simpler CSS.

Why? A phone layout is usually the simplest: things stack in one column. Larger screens add complexity, like multiple columns and sidebars. Starting simple and adding complexity is easier than starting complex and stripping it away.

In practice, mobile-first means your media queries use **`min-width`** (apply these styles when the screen is *at least* this wide):

```css
/* Base styles: written for small screens (phones) */
.container {
  display: flex;
  flex-direction: column;
}

/* Then enhance for wider screens */
@media (min-width: 768px) {
  .container {
    flex-direction: row;
  }
}
```

On a phone, the container stacks vertically. On a tablet or wider, it becomes a row.

## `@media` Queries

A **media query** is a block of CSS that only applies when a condition is true, usually a condition about screen width. You write it with the `@media` rule.

```css
@media (min-width: 600px) {
  /* These rules apply only when the screen is 600px wide or more */
  body {
    background: lightyellow;
  }
}
```

### `min-width` vs `max-width`

- **`min-width`** applies when the screen is *at least* that wide. Used for mobile-first.
- **`max-width`** applies when the screen is *at most* that wide. Used for desktop-first or for targeting only small screens.

```css
/* Apply only on small screens (phones) */
@media (max-width: 600px) {
  .sidebar {
    display: none;
  }
}
```

### Common Breakpoints

A **breakpoint** is the screen width where your layout changes. There is no official set, but these are common starting points based on typical device sizes:

```css
/* Small tablets and up */
@media (min-width: 600px) { /* ... */ }

/* Tablets and small laptops */
@media (min-width: 768px) { /* ... */ }

/* Laptops */
@media (min-width: 1024px) { /* ... */ }

/* Large desktops */
@media (min-width: 1280px) { /* ... */ }
```

Tip: do not memorize exact numbers. Add a breakpoint wherever your content starts to look cramped or awkward. Let the design tell you where to break.

## Fluid Layouts

Media queries handle big rearrangements. But between breakpoints, you also want elements that *stretch* smoothly. That is where fluid units come in.

### Percentages and `fr`

Instead of fixed pixel widths, use percentages or the `fr` unit so elements scale with their container.

```css
.column {
  width: 50%; /* always half the parent, whatever the screen size */
}
```

```css
.grid {
  display: grid;
  grid-template-columns: 1fr 1fr; /* two equal, flexible columns */
}
```

### `max-width` for Readable Content

Text that stretches across a huge monitor is hard to read. Use `max-width` to cap how wide a block gets, while still letting it shrink on small screens.

```css
.article {
  max-width: 700px; /* never wider than 700px */
  width: 100%;      /* but shrink freely on small screens */
  margin: 0 auto;   /* center it */
}
```

This pattern, `width: 100%` plus `max-width`, is everywhere in responsive design. It says "be as wide as you can, but not wider than 700px."

### `min()` and `clamp()`

Modern CSS gives you functions that pick a value based on the screen, removing the need for some media queries.

**`min()`** returns the smaller of the values you give it:

```css
.box {
  width: min(90%, 600px); /* whichever is smaller: 90% of the screen, or 600px */
}
```

On a phone, `90%` wins (so the box has comfortable margins). On a wide screen, `600px` wins (so it does not get too big). One line replaces a media query.

**`clamp()`** takes three values: a minimum, a preferred (flexible) value, and a maximum. It locks a value between two bounds while letting it flex in between.

```css
.title {
  font-size: clamp(1.5rem, 4vw, 3rem);
  /* never smaller than 1.5rem, never larger than 3rem,
     and 4vw (4% of viewport width) in between */
}
```

`clamp()` is perfect for fluid typography, which we cover next.

## Responsive Typography

Text should be readable on every device. A few techniques help:

```css
body {
  font-size: 16px; /* a comfortable base for phones */
}

@media (min-width: 768px) {
  body {
    font-size: 18px; /* slightly larger on bigger screens */
  }
}
```

Or use `clamp()` for headings that scale smoothly without breakpoints:

```css
h1 {
  font-size: clamp(1.75rem, 5vw, 3.5rem);
}
```

The `vw` unit means "1% of the viewport width," so the heading grows and shrinks with the window, but `clamp()` keeps it within sensible limits.

## Responsive Images

Images can overflow their container and break a layout. This one rule prevents that for almost every image on your page:

```css
img {
  max-width: 100%; /* never wider than its container */
  height: auto;    /* keep the original proportions */
}
```

`max-width: 100%` stops the image from spilling past its parent, and `height: auto` keeps it from stretching out of shape. Put this in your base styles and most image problems disappear.

## Hiding and Showing Elements Per Screen

Sometimes you want different content on different screens, like a full menu on desktop and a compact one on mobile. Use `display: none` inside media queries.

```css
/* Hide the desktop menu by default (mobile-first) */
.desktop-menu {
  display: none;
}

/* Show it on wider screens */
@media (min-width: 768px) {
  .desktop-menu {
    display: flex;
  }
  .mobile-menu {
    display: none; /* hide the mobile version on desktop */
  }
}
```

```html
<nav class="desktop-menu">Home About Services Contact</nav>
<button class="mobile-menu">☰ Menu</button>
```

Use this thoughtfully. Hidden elements still download, so do not hide huge amounts of content just to "save space."

## Putting It All Together: A Small Responsive Layout

Let's build a page header and a card section that adapts from one column on phones to three columns on desktops. Here is the HTML:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Responsive Demo</title>
</head>
<body>
  <header class="site-header">
    <h1>My Shop</h1>
  </header>

  <main class="content">
    <section class="cards">
      <div class="card">Card 1</div>
      <div class="card">Card 2</div>
      <div class="card">Card 3</div>
    </section>
  </main>
</body>
</html>
```

And the responsive CSS, written mobile-first:

```css
/* Base (phone) styles */
* {
  box-sizing: border-box;
}

body {
  margin: 0;
  font-size: 16px;
  font-family: sans-serif;
}

.site-header {
  background: #2b6cb0;
  color: white;
  padding: 16px;
  text-align: center;
}

.site-header h1 {
  font-size: clamp(1.5rem, 5vw, 2.5rem);
  margin: 0;
}

.content {
  width: 100%;
  max-width: 1100px;
  margin: 0 auto;
  padding: 16px;
}

/* Cards: one column on phones */
.cards {
  display: grid;
  grid-template-columns: 1fr;
  gap: 16px;
}

.card {
  background: #edf2f7;
  padding: 24px;
  border-radius: 8px;
}

/* Two columns on tablets */
@media (min-width: 600px) {
  .cards {
    grid-template-columns: 1fr 1fr;
  }
}

/* Three columns on laptops and up */
@media (min-width: 992px) {
  .cards {
    grid-template-columns: 1fr 1fr 1fr;
  }
}
```

What happens as the screen grows:

- On a phone, the cards stack in one column and the heading is small.
- At 600px, the grid becomes two columns.
- At 992px, it becomes three columns.
- The heading font scales smoothly the whole way thanks to `clamp()`.
- The content area never gets wider than 1100px, staying readable on big monitors.

That is a complete responsive layout in a single, readable stylesheet.

## Testing in DevTools Device Mode

You do not need a pile of phones to test. Every modern browser has a **device mode** built in.

1. Open your page in the browser.
2. Open DevTools (right-click and choose "Inspect", or press F12).
3. Click the small device icon (often called "Toggle device toolbar"), or press Ctrl+Shift+M (Cmd+Shift+M on a Mac).
4. Pick a device from the dropdown, or drag the edges to resize freely.

Watch your layout change as you resize. This is the fastest way to find breakpoints that need attention. Resize slowly and look for any width where the design looks cramped or broken, then add or adjust a media query there.

## Common Mistakes

- Forgetting the viewport meta tag, which makes the whole page look zoomed-out on phones.
- Using fixed pixel widths everywhere, so content cannot shrink on small screens.
- Mixing `min-width` and `max-width` queries inconsistently, which causes overlapping, conflicting rules. Pick mobile-first (`min-width`) and stay consistent.
- Forgetting `max-width: 100%` on images, letting them overflow the layout.
- Hiding large content with `display: none` to "save space." It still downloads, and users may want it.
- Picking breakpoints based on specific phone models instead of where the content actually breaks.

## Exercises

1. Add the viewport meta tag to a page, then remove it and view both on your phone or in device mode. Note how different they look.
2. Build a layout that is one column by default and becomes two columns at `min-width: 700px` using a media query.
3. Use `clamp()` to make a heading that smoothly scales between `1.5rem` and `3rem` as the window resizes.
4. Add `max-width: 100%; height: auto;` to images and confirm a large image no longer overflows its container.
5. Recreate the responsive card layout from this lesson, then open DevTools device mode and resize from phone to desktop, watching the columns change.

## Key Takeaways

- **Responsive design** makes one site adapt to every screen size.
- The viewport meta tag is required for responsive pages to work on phones.
- **Mobile-first** means base styles for small screens, then `min-width` media queries to enhance larger ones.
- `@media` queries apply CSS only at certain screen widths; add breakpoints where the content needs them.
- Fluid units (`%`, `fr`, `max-width`, `min()`, `clamp()`) let elements stretch smoothly between breakpoints.
- `max-width: 100%; height: auto;` keeps images from overflowing.
- `clamp()` is excellent for fluid, responsive typography.
- Test everything in DevTools device mode by resizing the window.

---

[← Back to Course Index](../README.md) | [Next: Specificity and the Cascade →](../05-advanced-css/01-specificity-and-the-cascade.md)
