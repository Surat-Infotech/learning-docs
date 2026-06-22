# Pseudo-classes and Pseudo-elements

So far you have styled elements based on what they are or what class they have. But sometimes you want to style an element based on its **state** (like being hovered) or its **position** (like being the first child). And sometimes you want to add a little decoration that isn't even in your HTML. **Pseudo-classes** and **pseudo-elements** make all of this possible.

## What You'll Learn

- The difference between a pseudo-class and a pseudo-element
- Why one uses a single colon and the other uses double colons
- State pseudo-classes: `:hover`, `:focus`, `:active`, `:visited`
- Structural pseudo-classes: `:first-child`, `:last-child`, `:nth-child()`
- Useful logic pseudo-classes: `:not()`, `:checked`, `:disabled`, `:focus-within`
- Generated content with `::before` and `::after`
- Text pseudo-elements: `::first-letter`, `::first-line`, `::placeholder`, `::selection`
- Practical patterns: custom bullets, tooltips, and decorative icons

## Pseudo-class vs Pseudo-element

These two sound similar but do different jobs.

- A **pseudo-class** selects an element that is in a certain **state** or **position**. The element already exists in your HTML; you are just targeting it under certain conditions. It uses a **single colon**: `:hover`.
- A **pseudo-element** lets you style a **specific part** of an element, or even create brand-new content that isn't in your HTML. It uses a **double colon**: `::before`.

The simple rule: **single colon = pseudo-class** (a condition), **double colon = pseudo-element** (a piece). You may see old code use a single colon for `::before`; that still works for backward compatibility, but always write two colons in new code.

## State Pseudo-classes

These respond to how the user interacts with an element.

### `:hover`

Applies when the mouse pointer is over the element.

```css
button:hover {
  background-color: #2563eb;
  color: white;
}
```

### `:focus`

Applies when an element is focused, usually by clicking into it or pressing Tab. This matters a lot for keyboard users and accessibility.

```css
input:focus {
  outline: 2px solid #2563eb;
  border-color: #2563eb;
}
```

Never remove focus styles without replacing them. Keyboard users rely on seeing where they are.

### `:active`

Applies in the moment the element is being clicked (pressed down).

```css
button:active {
  transform: scale(0.97); /* a tiny press-down effect */
}
```

### `:visited`

Applies to links the user has already visited. For privacy reasons, browsers only let you change a limited set of properties here, mainly color.

```css
a:visited {
  color: #7c3aed;
}
```

**Order matters** for link states. The classic memory trick is **LoVe HAte**: `:link`, `:visited`, `:hover`, `:active`. Write them in that order so they don't override each other unexpectedly.

## Structural Pseudo-classes

These select elements based on their position among their siblings.

### `:first-child` and `:last-child`

```css
li:first-child {
  font-weight: bold;
}

li:last-child {
  border-bottom: none;
}
```

`:first-child` matches a `<li>` only if it is the very first child of its parent. `:last-child` matches the last one.

### `:nth-child()`

This is the powerful one. It selects elements by a pattern. You put a formula or keyword inside the parentheses.

```css
/* every even row */
tr:nth-child(even) {
  background-color: #f3f4f6;
}

/* every odd row */
tr:nth-child(odd) {
  background-color: white;
}

/* the third item exactly */
li:nth-child(3) {
  color: red;
}

/* every third item: 3, 6, 9, ... */
li:nth-child(3n) {
  color: blue;
}

/* every third item starting from the first: 1, 4, 7, ... */
li:nth-child(3n + 1) {
  color: green;
}
```

The formula `an + b` looks scary but reads simply: `n` counts up 0, 1, 2, 3... and the result tells the browser which children to pick. `even` and `odd` are friendly shortcuts you'll use most often, especially for striped tables.

## Logic and State Pseudo-classes

### `:not()`

Selects elements that do **not** match what's inside.

```css
/* every button except the disabled ones */
button:not(:disabled) {
  cursor: pointer;
}

/* every list item except the last */
li:not(:last-child) {
  margin-bottom: 8px;
}
```

### `:checked`

Applies to a checked checkbox or radio button. This unlocks neat tricks where a checkbox controls visible styling with no JavaScript.

```css
input[type="checkbox"]:checked + label {
  text-decoration: line-through;
  color: gray;
}
```

The `+` is the **adjacent sibling** selector, meaning "the label right after a checked checkbox."

### `:disabled`

Applies to form elements that are disabled.

```css
input:disabled {
  background-color: #e5e7eb;
  cursor: not-allowed;
}
```

### `:focus-within`

Applies to a parent when any element **inside it** has focus. Great for highlighting a whole form group when the user clicks into a field.

```css
.form-group:focus-within {
  border-color: #2563eb;
}
```

## Pseudo-elements: `::before` and `::after`

`::before` and `::after` create a piece of content at the start or end of an element. They need the **`content`** property to appear, even if it's just an empty string.

```css
.note::before {
  content: "Note: ";
  font-weight: bold;
}
```

```html
<p class="note">Save your work often.</p>
```

This renders as **Note:** Save your work often. The "Note: " text lives only in CSS, not in the HTML.

### Custom Bullets

You can replace boring list bullets with anything.

```css
ul {
  list-style: none; /* remove the default dots */
  padding-left: 0;
}

li::before {
  content: "✓ ";
  color: green;
}
```

### Decorative Icons and Shapes

Use an empty `content` plus sizing to draw a small box or dot.

```css
.badge::before {
  content: "";
  display: inline-block;
  width: 8px;
  height: 8px;
  background-color: #10b981;
  border-radius: 50%;
  margin-right: 6px;
}
```

Because this is decorative, screen readers ignore it. That's perfect for visual-only flourishes. Do not put meaningful text in `::before`/`::after`, since assistive technology may skip it.

### A Simple Tooltip

`::after` is great for tooltips that read from a data attribute.

```html
<button class="tip" data-tip="Click to save">Save</button>
```

```css
.tip {
  position: relative;
}

.tip::after {
  content: attr(data-tip); /* pull text from the data-tip attribute */
  position: absolute;
  bottom: 100%;
  left: 0;
  background: #111827;
  color: white;
  padding: 4px 8px;
  border-radius: 4px;
  white-space: nowrap;
  opacity: 0;
  pointer-events: none;
}

.tip:hover::after {
  opacity: 1;
}
```

Here `attr(data-tip)` reads the text straight from the HTML attribute. Notice you can chain a pseudo-class and a pseudo-element: `:hover::after` means "the `::after` of this element while it is hovered."

## Text Pseudo-elements

### `::first-letter`

Styles just the first letter of a block. Perfect for a "drop cap" look.

```css
p::first-letter {
  font-size: 2.5rem;
  font-weight: bold;
  float: left;
  margin-right: 4px;
}
```

### `::first-line`

Styles the first line of text, however long that line happens to be.

```css
p::first-line {
  font-variant: small-caps;
}
```

### `::placeholder`

Styles the faint hint text inside an empty input.

```css
input::placeholder {
  color: #9ca3af;
  font-style: italic;
}
```

### `::selection`

Styles text the user has highlighted with their mouse.

```css
::selection {
  background: #fde68a;
  color: #111827;
}
```

## Common Mistakes

- Forgetting the `content` property on `::before`/`::after` — without it, nothing appears.
- Using a single colon for pseudo-elements in new code (write two colons).
- Removing `:focus` styles, which hurts keyboard users badly.
- Writing link states in the wrong order so `:hover` stops working after a link is visited.
- Putting important text or links inside `::before`/`::after`, where screen readers may miss it.
- Confusing `:nth-child()` with element position when other element types are mixed in.

## Exercises

1. Build a navigation bar where links turn blue on `:hover` and show a clear outline on `:focus`. Test it using only the Tab key.
2. Create a striped table using `:nth-child(even)` for the row background. Then bold the `:first-child` row as a header.
3. Make a custom checklist: remove default bullets and use `li::before` with a checkmark. Cross out items using `:checked + label`.
4. Build a tooltip with `::after` and `attr()` that appears on `:hover`, reading its text from a `data-tip` attribute.
5. Add a drop cap to the first paragraph of an article using `::first-letter`, and style highlighted text with `::selection`.

## Key Takeaways

- Pseudo-classes (single colon) select elements by state or position.
- Pseudo-elements (double colon) style a part of an element or add new content.
- `:hover`, `:focus`, `:active`, and `:visited` respond to user interaction — keep focus styles visible.
- `:nth-child()` selects by pattern; `even` and `odd` are the most common.
- `:not()`, `:checked`, `:disabled`, and `:focus-within` cover handy logic and form states.
- `::before` and `::after` need a `content` value and are great for decoration and icons.
- `::first-letter`, `::first-line`, `::placeholder`, and `::selection` style specific text parts.
- Keep meaningful content in HTML; use generated content for decoration only.

---

[← Back to Course Index](../README.md) | [Next: Custom Properties (CSS Variables) →](03-custom-properties.md)
