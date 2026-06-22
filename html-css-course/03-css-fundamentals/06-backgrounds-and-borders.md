# Backgrounds and Borders

Backgrounds and borders are where pages start to look polished. With them you can add colors, images, gradients, rounded corners, and soft shadows. In this lesson you will learn the properties that turn plain boxes into attractive cards, banners, and buttons.

## What You'll Learn

- Setting a `background-color` and a `background-image`
- Controlling images with `background-size`, `background-position`, and `background-repeat`
- The `background` shorthand
- Creating linear and radial gradients
- Drawing borders, including individual sides
- Rounding corners with `border-radius`, including perfect circles
- Adding depth with `box-shadow`
- The difference between `outline` and `border`

## background-color

The simplest background is a solid color, set with `background-color`.

```css
.box {
  background-color: #f0f4ff;
}
```

You can use any color format you learned earlier: keywords, hex, `rgb()`, or `hsl()`.

```css
.tinted {
  background-color: rgba(0, 0, 0, 0.05); /* a faint dark tint */
}
```

## background-image

You can set an image as the background using `background-image` with the `url()` function. Put the path to your image inside the parentheses.

```css
.hero {
  background-image: url("mountains.jpg");
}
```

By default, the image keeps its natural size and tiles to fill the box, which is rarely what you want. The next three properties fix that.

## background-size

The `background-size` property controls how the image is scaled. Two values do most of the work:

```css
.hero {
  background-image: url("mountains.jpg");
  background-size: cover;
}
```

- `cover` — scale the image so it completely covers the box, cropping edges if needed. Great for full-bleed banners.
- `contain` — scale the image so it fits entirely inside the box, possibly leaving empty space. Great when you must see the whole image.

You can also give exact sizes:

```css
.box {
  background-size: 200px 100px; /* width height */
}
```

## background-position

The `background-position` property sets where the image sits inside the box. This matters most with `cover`, since it decides which part gets cropped.

```css
.hero {
  background-image: url("mountains.jpg");
  background-size: cover;
  background-position: center;
}
```

`center` keeps the middle of the image in view. You can also use keywords like `top`, `bottom`, `left`, `right`, combinations like `top right`, or exact values like `50% 25%`.

## background-repeat

By default, a background image **repeats** (tiles) to fill the box. The `background-repeat` property controls this.

```css
.pattern {
  background-image: url("dot.png");
  background-repeat: repeat; /* tile in both directions (the default) */
}

.no-tile {
  background-repeat: no-repeat; /* show the image only once */
}
```

Other values include `repeat-x` (tile horizontally only) and `repeat-y` (tile vertically only). For a single hero image, you almost always want `no-repeat`.

## The background Shorthand

The `background` shorthand lets you set many of these properties in one line.

```css
.hero {
  background: url("mountains.jpg") center / cover no-repeat #222;
}
```

Reading this: the image, positioned at `center`, sized as `cover` (note the slash between position and size), set to `no-repeat`, with a dark `#222` fallback color behind it. The order is flexible, but position and size must be joined by a slash: `position / size`.

When you are starting out, it is fine to write each property on its own line for clarity, then switch to the shorthand once you are comfortable.

## Gradients

A **gradient** is a smooth blend from one color to another. Gradients are technically images, so they go in `background-image` (or `background`). There are two main kinds.

### Linear Gradients

A **linear gradient** blends colors along a straight line.

```css
.banner {
  background-image: linear-gradient(to right, #ff7e5f, #feb47b);
}
```

The first value is the direction. `to right` blends left to right; `to bottom` blends top to bottom. You can also use an angle like `45deg`. After the direction, list the colors.

You can blend more than two colors:

```css
.rainbow {
  background-image: linear-gradient(to right, red, orange, yellow, green);
}
```

### Radial Gradients

A **radial gradient** blends colors outward from a center point, like ripples.

```css
.glow {
  background-image: radial-gradient(circle, #ffffff, #6a82fb);
}
```

It starts with the first color in the middle and fades to the last color at the edges. The `circle` keyword makes it round; the default is an ellipse that matches the box shape.

## Borders

A **border** is a line drawn around an element's box. The `border` shorthand sets three things at once: width, style, and color.

```css
.box {
  border: 2px solid #333;
}
```

- **Width** — how thick the line is (`2px`).
- **Style** — the kind of line. Common values: `solid`, `dashed`, `dotted`, `double`, and `none`.
- **Color** — any color value.

### Individual Sides

You can style each side on its own with `border-top`, `border-right`, `border-bottom`, and `border-left`.

```css
.box {
  border-bottom: 3px solid tomato; /* a line only on the bottom */
}
```

This is great for things like an underline under a heading, or a colored accent on one edge.

You can also target just one part, like the color of one side:

```css
.box {
  border: 1px solid #ccc;
  border-left-color: red;
}
```

## border-radius

The `border-radius` property rounds the corners of a box. It works even without a visible border, since it also clips the background.

```css
.card {
  border-radius: 8px;
}
```

Bigger values make rounder corners. You can round each corner differently with up to four values, in the same clockwise order (top-left, top-right, bottom-right, bottom-left).

```css
.tab {
  border-radius: 8px 8px 0 0; /* rounded top, square bottom */
}
```

### Making a Circle

A neat trick: if an element is a perfect square (equal width and height), setting `border-radius: 50%` turns it into a circle.

```css
.avatar {
  width: 100px;
  height: 100px;
  border-radius: 50%;
}
```

This is the standard way to make round profile pictures and circular buttons.

## box-shadow

The `box-shadow` property adds a shadow around a box, giving it depth and making it feel like it lifts off the page.

```css
.card {
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
}
```

The values, in order, are:

1. **Horizontal offset** — how far right the shadow moves (`0`).
2. **Vertical offset** — how far down (`4px`).
3. **Blur** — how soft and spread out the shadow is (`12px`). Bigger is softer.
4. **Color** — usually a semi-transparent black with `rgba` so it looks natural.

A small downward shadow with a soft blur and low opacity looks gentle and professional. Big, dark, sharp shadows tend to look heavy.

You can add an optional fourth length, the **spread**, to grow or shrink the shadow before blurring:

```css
.box {
  box-shadow: 0 2px 6px 1px rgba(0, 0, 0, 0.2);
}
```

## outline vs border

An **outline** looks like a border but works differently. The key differences:

- An outline is drawn *outside* the border and does **not** take up space in the box model. It will not shift your layout.
- An outline does not follow `border-radius`; it usually stays rectangular.

```css
button:focus {
  outline: 2px solid blue;
}
```

The most important use of `outline` is the focus ring: the highlight that appears when you tab to a button or link with the keyboard. It helps people who navigate without a mouse. Do not remove it unless you replace it with another clear focus style, since it matters for accessibility.

## A Polished Card Example

Let's combine everything into a clean card, the kind you see all over the web.

```html
<div class="card">
  <img class="card-image" src="photo.jpg" alt="A scenic view" />
  <div class="card-body">
    <h2>Mountain Trip</h2>
    <p>A weekend hike with stunning views and fresh air.</p>
  </div>
</div>
```

```css
* {
  box-sizing: border-box;
}

.card {
  width: 300px;
  background-color: #ffffff;
  border: 1px solid #e5e5e5;
  border-radius: 12px;
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.1);
  overflow: hidden; /* keeps the image corners rounded */
}

.card-image {
  width: 100%;
  height: 180px;
  object-fit: cover; /* fill the area without stretching */
}

.card-body {
  padding: 16px;
}

.card-body h2 {
  margin: 0 0 8px;
}
```

Notice how the pieces work together: a subtle border, rounded corners, a soft shadow for depth, and `overflow: hidden` so the image respects the rounded corners. Small touches like these make a design feel finished.

## Common Mistakes

- **Forgetting `background-repeat: no-repeat`.** A single image will tile and look messy without it.
- **Skipping `background-size`.** Use `cover` for banners so the image fills the box neatly.
- **Forgetting the path quotes or a wrong path in `url()`.** If the path is wrong, the image just will not appear.
- **Removing the focus outline with no replacement.** This hurts keyboard users. Always provide a visible focus style.
- **Using huge, dark `box-shadow` values.** Subtle shadows look more professional than heavy ones.
- **Expecting `border-radius: 50%` to make a non-square element a circle.** It needs equal width and height to be a perfect circle.

## Exercises

1. Create a banner with a background image. Use `cover`, `center`, and `no-repeat` so it fills the area neatly.
2. Make a button with a `linear-gradient` background that blends two colors from left to right.
3. Build a square element and turn it into a circle with `border-radius: 50%`.
4. Add a soft `box-shadow` to a box and experiment with the blur and the alpha until it looks gentle.
5. Recreate the card example, then change the `border-radius` and shadow values and see how the feel changes.

## Key Takeaways

- `background-color` sets a solid background; `background-image: url()` sets an image.
- Control images with `background-size` (`cover`/`contain`), `background-position`, and `background-repeat`.
- The `background` shorthand combines these; position and size are joined by a slash.
- Gradients are images: use `linear-gradient()` for straight blends and `radial-gradient()` for circular ones.
- `border` sets width, style, and color, and you can style each side on its own.
- `border-radius` rounds corners; `50%` on a square makes a circle.
- `box-shadow` adds depth; keep it soft and subtle.
- `outline` looks like a border but takes no space and is important for keyboard focus.

---

[← Back to Course Index](../README.md) | [Next: Display and the Normal Flow →](../04-layout-and-responsive/01-display-and-the-flow.md)
