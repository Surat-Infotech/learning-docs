# DevTools and Debugging

Every modern browser has a built-in toolkit for looking inside web pages. It is called **DevTools** (short for Developer Tools). With it you can see your HTML, change your CSS live, find errors, and figure out why something looks wrong. Learning DevTools well is one of the biggest speed boosts you can give yourself as a developer.

## What You'll Learn

- How to open DevTools and what the main panels do
- How to inspect and live-edit HTML and CSS
- How to read the Styles pane, including crossed-out rules
- How to use the box model viewer
- How to toggle states like `:hover`
- How to test responsive layouts with device mode
- How to read errors in the Console
- How to find and fix common CSS bugs

## Opening DevTools

To open DevTools, press **F12**. On most browsers, you can also right-click any element on a page and choose **Inspect**. On a Mac you might use `Cmd + Option + I`.

When DevTools opens, you will see several tabs along the top. The ones you will use most are:

- **Elements** (sometimes called **Inspector**): shows the page's HTML and CSS.
- **Console**: shows errors and messages.
- **Network**: shows files the page downloads (you will use this more later).

For this lesson we focus on **Elements** and **Console**.

## The Elements Panel

The **Elements panel** shows the live HTML of the page as a tree you can expand and collapse. This is not the original file on disk; it is what the browser actually built and is showing right now. That difference matters: if your JavaScript or CSS changes the page, you see the *current* state here.

Click the small arrow icon at the top-left of DevTools (the "inspect" cursor), then click anything on the page. DevTools jumps to that element in the tree and highlights it. This is the fastest way to answer "what is this thing, and which class controls it?"

### Live-Editing HTML

You can change the HTML right inside DevTools to test an idea. Double-click any text to edit it, or right-click an element and choose **Edit as HTML**.

```html
<!-- Double-click "Old title" and type a new one -->
<h1>Old title</h1>
```

These edits are temporary. They vanish when you reload the page. That is the point: you can experiment freely without touching your real files. Once you like a change, copy it into your actual code.

## The Styles Pane

When an element is selected, the **Styles pane** (usually on the right) shows every CSS rule that applies to it. This is where most debugging happens.

You can edit values live. Click a value, type a new one, and the page updates instantly:

```css
.button {
  background: blue;   /* click "blue", type "green", see it change */
  padding: 10px;
}
```

You can also add a brand-new property by clicking in the empty space inside a rule and typing it.

### Crossed-Out Rules and Specificity

Sometimes you will see a property with a **line through it**, like this:

```css
.button {
  color: red;   /* crossed out */
}
.button--primary {
  color: white; /* this one wins */
}
```

A crossed-out property means it was **overridden**. Another rule with higher **specificity** (a stronger selector) or one that comes later won the fight, so the crossed-out value is ignored.

This is incredibly useful. When your color is "not working," the crossed-out line shows you exactly which other rule is winning. DevTools lists rules from most specific at the top to least specific at the bottom, so you can trace the chain.

## The Box Model Viewer

At the bottom of the Styles pane (or in a "Computed" tab) you will find the **box model viewer**. It is a diagram of nested boxes showing, from inside out:

```text
content  ->  padding  ->  border  ->  margin
```

Each layer shows its size in pixels. If an element looks too big or has mysterious space around it, this diagram tells you whether the cause is margin, padding, or border. Hovering over each layer highlights it on the page in a different color.

## Toggling Pseudo-States

A **pseudo-state** is a special condition, like when the mouse is hovering over a button (`:hover`) or a link is focused (`:focus`). These are hard to inspect because the moment you move your mouse to DevTools, the hover ends.

DevTools fixes this. In the Styles pane there is a button labeled **:hov** (or "Toggle Element State"). Click it and you can force states on:

```css
.link:hover {
  text-decoration: underline;  /* now visible even without hovering */
}
```

Check the box for `:hover` and the element stays in its hover style so you can inspect it calmly.

## Device and Responsive Mode

**Responsive mode** lets you preview how your page looks on phones and tablets without owning every device. Look for a small icon of a phone and tablet near the top-left of DevTools, or press `Cmd/Ctrl + Shift + M`.

When you turn it on:

- You can drag the edges to change the screen width.
- You can pick a preset device like "iPhone" from a dropdown.
- A width number shows at the top so you can find exactly where your layout breaks.

This is the best way to test your **media queries** (the CSS rules that change at different screen sizes).

## The Console

The **Console** is where the browser reports problems. Open the Console tab and look for red messages. They often point to the exact file and line where something went wrong.

```text
Uncaught ReferenceError: total is not defined
    at script.js:12
```

Even in an HTML and CSS project, the Console is useful. A missing image or a broken stylesheet link shows up here as an error. If something on your page is just not loading, check the Console first.

## Finding Common CSS Bugs

Here are the bugs you will hit most often and how DevTools helps you spot each one.

### Typos

A single misspelled property or value is silently ignored. The Styles pane shows valid properties; if you typed `colr` instead of `color`, you will see it crossed out or marked with a warning icon. Always check that your property actually appears as active.

### Specificity

Your rule is correct but "does nothing." Select the element and look for your property crossed out in the Styles pane. A more specific rule is winning. Either raise your selector's specificity sensibly or reduce the other rule's.

### Inheritance

Some properties, like `color` and `font-family`, pass down from parent to child automatically. This is **inheritance**. If text is the wrong color and you see no rule on the element itself, check the parent. The "Computed" tab shows where an inherited value came from.

### Overflow

**Overflow** happens when content is bigger than its box, causing scrollbars or content that spills out. Inspect the element and the box model viewer; an element wider than the screen often causes a horizontal scrollbar. Look for fixed widths that are too large.

### z-index

**z-index** controls which elements stack on top of others. If something is hidden behind another element, select both and compare their `z-index` values. Remember: `z-index` only works on **positioned** elements (those with `position` set to `relative`, `absolute`, or `fixed`).

## The "Outline Everything" Trick

When your layout is mysteriously broken and you cannot see the boxes, add a temporary outline to every element:

```css
* {
  outline: 1px solid red;
}
```

The `*` selects everything, and `outline` draws a line *around* each element without changing its size (unlike `border`, which adds size). Suddenly you can see every box on the page: where it sits, how big it is, and what is overflowing. Paste this into the Styles pane, debug, then remove it. It is one of the simplest and most powerful tricks in this lesson.

## Common Mistakes

- Editing the real file when you meant to experiment, instead of testing in DevTools first.
- Ignoring crossed-out rules, then being confused about why a style "does not work."
- Forgetting that `z-index` needs a `position` value to take effect.
- Not checking the Console when something fails to load.
- Using `border` instead of `outline` for the debug trick, which changes element sizes and confuses the layout.
- Testing responsiveness by shrinking the whole browser window instead of using device mode.

## Exercises

1. Open any website, use the inspect cursor to select a heading, and change its text and color live in DevTools.
2. Find an element with extra spacing and use the box model viewer to decide if it is margin or padding.
3. Force the `:hover` state on a button and inspect its hover styles.
4. Turn on responsive mode and find the width at which a layout first looks wrong.
5. Paste the `* { outline: 1px solid red; }` trick into a page and describe what you can now see that you could not before.

## Key Takeaways

- Press F12 (or right-click then Inspect) to open DevTools.
- The Elements panel shows live HTML; edits there are temporary and safe.
- The Styles pane shows all applied rules; crossed-out ones were overridden.
- The box model viewer reveals where spacing comes from.
- Toggle pseudo-states like `:hover` to inspect them without your mouse.
- Use responsive mode to test layouts at any width.
- Check the Console for errors and failed downloads.
- The outline-everything trick reveals every box on the page instantly.

---

[← Back to Course Index](../README.md) | [Next: Best Practices and Performance →](03-best-practices-and-performance.md)
