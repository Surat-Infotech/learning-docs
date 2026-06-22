# Transitions and Transforms

Up to now, your CSS changes have been instant. A button is one color, then on hover it is suddenly another color, with no in-between. **Transitions** and **transforms** add smooth motion and visual change. Transforms move, scale, and rotate elements. Transitions make those changes happen gradually instead of jumping. Together they bring your pages to life.

## What You'll Learn

- How to move, grow, rotate, and slant elements with `transform`
- How to combine multiple transforms in one declaration
- How `transform-origin` changes the pivot point
- How to animate changes smoothly with `transition`
- The timing functions: `ease`, `linear`, `ease-in-out`, and `cubic-bezier`
- How to trigger transitions on `:hover` and `:focus`
- Which properties can and cannot be transitioned
- A performance tip: prefer `transform` and `opacity`

## Transforms

The **`transform`** property changes how an element looks — its position, size, angle, or shape — **without** affecting the layout around it. Other elements behave as if the element never moved. This makes transforms cheap and smooth.

### `translate` — Move

`translate` shifts an element along the X (horizontal) and Y (vertical) axes.

```css
.box {
  transform: translateX(20px);   /* right 20px */
}

.box2 {
  transform: translateY(-10px);  /* up 10px */
}

.box3 {
  transform: translate(20px, -10px); /* both at once: x, then y */
}
```

Positive X moves right, positive Y moves **down**. Negative values go the other way.

### `scale` — Resize

`scale` grows or shrinks an element. `1` is normal size, `2` is double, `0.5` is half.

```css
.box {
  transform: scale(1.2);     /* 20% bigger in both directions */
}

.box2 {
  transform: scale(2, 0.5);  /* 2x wide, half height */
}
```

### `rotate` — Spin

`rotate` turns the element. Angles use `deg` (degrees). Positive is clockwise.

```css
.box {
  transform: rotate(45deg);
}
```

### `skew` — Slant

`skew` slants the element, like pushing the top sideways.

```css
.box {
  transform: skewX(10deg);
}
```

`skew` is used less often, but it's handy for stylized, slanted shapes.

### Combining Transforms

You can apply several transforms at once by listing them, separated by spaces, in a single `transform` value.

```css
.box {
  transform: translateX(20px) rotate(15deg) scale(1.1);
}
```

**Order matters.** The browser applies them in order, and each transform affects the next. `rotate` then `translate` gives a different result than `translate` then `rotate`. If something looks off, try swapping the order.

### `transform-origin` — The Pivot Point

By default, transforms happen around the **center** of the element. `transform-origin` moves that pivot.

```css
.box {
  transform-origin: top left; /* rotate/scale around the top-left corner */
  transform: rotate(20deg);
}
```

Think of `transform-origin` as the pin you stick into the element before spinning or scaling it. You can use keywords (`top`, `left`, `center`) or exact values like `0 0` or `50% 100%`.

## Transitions

A **transition** tells the browser to animate a change in a property over time, instead of snapping to the new value instantly. You set it once, and any time that property changes, the change happens smoothly.

A transition is built from four parts.

### `transition-property` — What to Animate

Which property should change smoothly. Use `all` to animate everything, or name a specific property (better for performance).

```css
.box {
  transition-property: background-color;
}
```

### `transition-duration` — How Long

How much time the change takes. Use `s` (seconds) or `ms` (milliseconds).

```css
.box {
  transition-duration: 0.3s;
}
```

### `transition-timing-function` — The Pace

How the speed changes during the transition. This controls whether the motion feels mechanical or natural.

- **`linear`** — constant speed the whole way. Good for spinners.
- **`ease`** — starts slow, speeds up, ends slow. The default and a good all-rounder.
- **`ease-in`** — starts slow, finishes fast.
- **`ease-out`** — starts fast, finishes slow.
- **`ease-in-out`** — slow at both ends, faster in the middle.
- **`cubic-bezier(...)`** — define your own custom curve.

```css
.box {
  transition-timing-function: ease-in-out;
}
```

A **`cubic-bezier`** lets you craft a precise speed curve with four numbers. You rarely write these by hand; instead, use an online "cubic-bezier" tool to drag a curve and copy the values.

```css
.box {
  transition-timing-function: cubic-bezier(0.25, 0.1, 0.25, 1);
}
```

### `transition-delay` — Wait First

How long to wait before the transition starts.

```css
.box {
  transition-delay: 0.1s;
}
```

### The `transition` Shorthand

Writing all four properties separately is tedious. The **`transition` shorthand** combines them in one line. The order is: property, duration, timing-function, delay.

```css
.box {
  transition: background-color 0.3s ease-in-out 0s;
}
```

You can transition several properties by separating them with commas:

```css
.box {
  transition: background-color 0.3s ease, transform 0.2s ease-out;
}
```

Most of the time you'll just write `transition: all 0.3s ease;` for quick work, then tighten it to specific properties later.

## Triggering on Hover and Focus

The key idea: put the `transition` on the **normal** state of the element, then change the value in `:hover` or `:focus`. The transition handles both directions automatically — into the hover and back out.

### A Hover Button

```css
.btn {
  background-color: #2563eb;
  color: white;
  padding: 12px 24px;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  transition: background-color 0.25s ease, transform 0.1s ease;
}

.btn:hover {
  background-color: #1d4ed8;
}

.btn:active {
  transform: scale(0.97);
}
```

```html
<button class="btn">Save Changes</button>
```

Hover and the color eases in. Move away and it eases back, no extra code needed. Click and it presses down slightly with `scale`.

### A Card Lift

A favorite effect: a card that floats up and gains a shadow on hover.

```css
.card {
  background: white;
  padding: 24px;
  border-radius: 12px;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.1);
  transition: transform 0.2s ease, box-shadow 0.2s ease;
}

.card:hover {
  transform: translateY(-6px);
  box-shadow: 0 12px 24px rgba(0, 0, 0, 0.15);
}
```

The card rises by 6 pixels and its shadow deepens, giving a real sense of lift. Because we transition `transform` rather than `top` or `margin`, it stays smooth and doesn't disturb the layout.

## What Can and Can't Be Transitioned

Not every property can animate. The rule of thumb: a property can be transitioned if it has a clear range of in-between values the browser can step through.

- **Can transition:** colors, `opacity`, `width`, `height`, `margin`, `padding`, `transform`, `box-shadow`, `font-size`. Anything with a numeric or color value.
- **Cannot transition:** properties that switch between distinct states with no middle ground, like `display` (you can't be half `block` and half `none`) or `font-family`.

If a transition seems to do nothing, check whether the property is even transitionable. To hide and show with motion, animate `opacity` instead of toggling `display`.

## A Performance Note

Browsers can animate some properties much more cheaply than others.

- **`transform` and `opacity` are cheap.** The browser can move and fade elements smoothly, often using the graphics hardware. They don't force the page to recalculate layout.
- **Properties like `width`, `height`, `top`, `left`, and `margin` are expensive.** Changing them forces the browser to recompute the position of other elements on every frame, which can cause stutter on slower devices.

**The tip:** when you want to move or resize something for an effect, prefer `transform` over changing `top`/`left`/`width`/`height`. For showing and hiding, prefer `opacity`. Your animations will feel smoother, especially on phones.

```css
/* Smoother — uses transform */
.menu:hover { transform: translateX(0); }

/* Janky — animates layout */
.menu:hover { left: 0; }
```

## Common Mistakes

- Putting the `transition` on the `:hover` state instead of the base state, so it only eases one way.
- Trying to transition `display: none` — it won't fade; use `opacity` and `visibility` instead.
- Animating `width`/`left`/`margin` when `transform` would be smoother.
- Forgetting that combined transforms apply in order, so the result depends on order.
- Using a very long duration that makes the interface feel sluggish (0.15s–0.3s feels snappy).
- Forgetting `transform-origin` when a rotation pivots from the wrong spot.

## Exercises

1. Make a box that moves 30px to the right and rotates 20 degrees on hover, with a smooth 0.3s transition.
2. Build a button that changes background color on `:hover` and scales down slightly on `:active`. Tune the duration until it feels snappy.
3. Create a card-lift effect: on hover, the card rises with `translateY` and gains a deeper `box-shadow`.
4. Take one element and try `transform-origin: top left` versus the default center while rotating it. Describe the difference.
5. Rewrite an animation that changes `left` so it uses `transform: translateX` instead, and note that the layout no longer shifts.

## Key Takeaways

- `transform` moves, scales, rotates, and skews elements without disturbing layout.
- Combine transforms in one value; order changes the result.
- `transform-origin` sets the pivot point for rotation and scaling.
- A transition animates a property change over time using property, duration, timing, and delay.
- Timing functions (`ease`, `linear`, `ease-in-out`, `cubic-bezier`) control the feel of the motion.
- Put the transition on the base state and change values in `:hover`/`:focus` to animate both ways.
- Only properties with in-between values can transition; `display` cannot.
- Prefer `transform` and `opacity` for smooth, high-performance animation.

---

[← Back to Course Index](../README.md) | [Next: Animations →](05-animations.md)
