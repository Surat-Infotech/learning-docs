# Specificity and the Cascade

Have you ever written a CSS rule, refreshed the page, and... nothing changed? Your color stayed the same, like the browser ignored you. Usually this is not a bug. It is the **cascade** and **specificity** doing their job. Once you understand these two ideas, those confusing moments turn into "oh, of course" moments.

## What You'll Learn

- What the cascade is and why CSS is called "cascading"
- How source order decides winners when rules are equal
- What specificity is and how to calculate it as a score
- How inline styles, IDs, classes, and elements rank
- What `!important` does and why you should avoid it
- How inheritance works, plus `inherit`, `initial`, and `unset`
- How to debug "why isn't my style applying?"
- Simple habits to keep specificity low and your CSS easy to manage

## What Is the Cascade?

CSS stands for **Cascading Style Sheets**. The "cascading" part means the browser has a set of rules for choosing which style wins when many rules try to style the same thing.

Think of it like water flowing down a series of steps. Styles flow down and pile up. When two styles clash, the browser needs a tiebreaker to decide which one reaches the element last and wins.

The browser asks three questions, in this order:

1. **Origin and importance** — Where did the rule come from, and is it marked `!important`?
2. **Specificity** — How precise is the selector?
3. **Source order** — If everything else ties, which rule came last in the code?

Let's walk through these from the simplest to the most important.

## Source Order: Last One Wins

When two rules have the **same specificity**, the one written **later** in your CSS wins.

```css
p {
  color: blue;
}

p {
  color: red;
}
```

Both rules target every `<p>`. They are equally specific. The paragraph text will be **red**, because the red rule came last. The later rule overrides the earlier one.

This is why the **order of your stylesheets matters** too. If you load `base.css` and then `theme.css`, rules in `theme.css` can override matching rules in `base.css`.

## Specificity: How Precise Is Your Selector?

**Specificity** is a score the browser gives each selector based on how exact it is. A more specific selector beats a less specific one, even if the less specific one came later.

We measure specificity with three numbers, often written as **(a, b, c)**:

- **a** = the number of **ID** selectors (`#header`)
- **b** = the number of **class**, **attribute**, and **pseudo-class** selectors (`.btn`, `[type="text"]`, `:hover`)
- **c** = the number of **element** and **pseudo-element** selectors (`div`, `p`, `::before`)

(There is also a special slot for inline styles, which we cover next. You can think of it as a fourth number to the left.)

To compare two selectors, you compare the numbers from left to right, like comparing version numbers. A higher `a` always wins. If `a` ties, the higher `b` wins. If `b` ties, the higher `c` wins.

### Counting Examples

Let's score some selectors.

```css
/* (0,0,1) — one element */
p { }

/* (0,1,0) — one class */
.intro { }

/* (1,0,0) — one ID */
#main { }

/* (0,1,1) — one class + one element */
p.intro { }

/* (0,2,1) — two classes + one element */
.card.active p { }

/* (1,1,1) — one ID + one class + one element */
#main .intro p { }
```

Now imagine these two rules both target the same paragraph:

```css
.intro {            /* (0,1,0) */
  color: blue;
}

p {                 /* (0,0,1) */
  color: red;
}
```

The text is **blue**. Even though `p` came last, `.intro` has a higher score: a class (0,1,0) beats an element (0,0,1).

### A Tricky One

```css
#sidebar a {        /* (1,0,1) */
  color: green;
}

.menu .link {       /* (0,2,0) */
  color: orange;
}
```

Which wins for a link that matches both? Compare left to right. The first has `a = 1`. The second has `a = 0`. The ID wins immediately, so the link is **green**. A single ID outranks any number of classes.

## Inline Styles

An **inline style** is a style written directly on an HTML element using the `style` attribute.

```html
<p style="color: purple;">Hello</p>
```

Inline styles are even more specific than IDs. In the (a, b, c) system, they sit in a slot to the left of `a`, so they almost always win against rules in your stylesheet.

```css
#main p {
  color: green;  /* loses */
}
```

```html
<div id="main">
  <p style="color: purple;">I am purple.</p>
</div>
```

The paragraph is **purple**. This is one reason to avoid inline styles: they are hard to override and they mix content with presentation.

## `!important`: The Override Hammer

Adding `!important` to a declaration lifts it above the normal specificity contest.

```css
p {
  color: red !important;
}

#main p {
  color: green;  /* still loses */
}
```

The paragraph is **red**, even though `#main p` is far more specific. The `!important` flag jumps to the front of the line.

### Why You Should Avoid It

`!important` feels powerful, but it is a trap:

- It breaks the natural cascade, so styles become unpredictable.
- The only way to override an `!important` is with **another** `!important` that is more specific. This starts an escalating war.
- Six months later, nobody knows why a value can't be changed.

**Rule of thumb:** if you reach for `!important`, stop and ask why your selector isn't winning normally. There is almost always a cleaner fix. The rare acceptable use is overriding styles from a third-party library you cannot edit.

## Inheritance

Some CSS properties pass down from a parent element to its children automatically. This is called **inheritance**.

```css
body {
  color: #333;
  font-family: Arial, sans-serif;
}
```

Every paragraph, heading, and span inside `<body>` will use that color and font, even though you never targeted them directly. They **inherited** it.

**Properties that usually inherit** are mostly about text:

- `color`
- `font-family`, `font-size`, `font-weight`
- `line-height`
- `text-align`

**Properties that usually do NOT inherit** are mostly about layout and boxes:

- `margin`, `padding`
- `border`
- `width`, `height`
- `background`

This makes sense. You want text styling to flow down so you don't repeat it. But you would not want every child to copy its parent's border.

### Controlling Inheritance: `inherit`, `initial`, `unset`

You can force inheritance behavior with three special values:

- **`inherit`** — take the value from the parent, even for a property that normally would not inherit.
- **`initial`** — reset to the property's built-in default value (the value it has with no CSS at all).
- **`unset`** — act like `inherit` if the property normally inherits, otherwise act like `initial`. A handy "do the natural thing" reset.

```css
.box {
  border: 2px solid black;
}

.box .inner {
  border: inherit; /* copies the parent border on purpose */
}

a {
  color: initial;  /* back to the browser default text color */
}
```

## Debugging: "Why Isn't My Style Applying?"

When a style refuses to take effect, work through this checklist:

1. **Is the selector even matching?** Maybe you wrote `.btn` but the HTML class is `button`. Check the spelling and the HTML.
2. **Is something more specific overriding it?** Open your browser's DevTools, click the element, and look at the **Styles** panel. Overridden rules appear with a line struck through them. This shows you the exact winner.
3. **Is it source order?** Two equal rules, and the other one comes later? Reorder or make yours more specific.
4. **Is there an `!important` somewhere?** Search your CSS for it.
5. **Is the property even inherited?** Setting `border` on `<body>` will not give every child a border.
6. **Typo in the value?** `colour` instead of `color`, or a missing semicolon, makes the browser silently skip the line.

DevTools is your best friend here. The struck-through styles tell the whole story.

## Keep Specificity Low

High specificity is hard to maintain because every override has to be even more specific. The fix is to keep things flat and simple.

- **Prefer classes** for styling. They are reusable and easy to override.
- **Avoid IDs in CSS** for styling. Use IDs for JavaScript hooks and anchors instead.
- **Avoid inline styles** except for quick tests.
- **Avoid deep selector chains** like `.a .b .c .d span`. Add a single class instead.
- **Save `!important`** for true emergencies, like overriding code you cannot change.

```css
/* Hard to override — avoid */
#sidebar ul li a.active { }

/* Easy to override — prefer */
.nav-link.is-active { }
```

A flat, class-based approach keeps your specificity scores low and even, so source order does the simple work of deciding winners.

## Common Mistakes

- Using IDs everywhere, then being unable to override them later.
- Reaching for `!important` instead of finding the real conflict.
- Forgetting that inline styles beat almost everything in your stylesheet.
- Assuming a property inherits when it does not (like `border` or `padding`).
- Writing long selector chains that are hard to beat.
- Not checking DevTools when a style "doesn't work" — the answer is usually right there, struck through.

## Exercises

1. Score these selectors as (a, b, c): `nav a`, `.menu a:hover`, `#header .logo`, `ul li.active`. Then rank them from least to most specific.
2. Write two rules targeting the same `<button>` so that the **less specific looking** rule still wins because of specificity. Explain why.
3. Create a parent `<div>` with `color` and `border` set. Add a child paragraph. Predict which of the two properties the child inherits, then test it in the browser.
4. Take a rule with `!important` and rewrite the CSS so it works **without** `!important`, using specificity and source order instead.
5. Open any web page, use DevTools to click on a heading, and find one style that is overridden (struck through). Identify which rule won and why.

## Key Takeaways

- CSS resolves conflicts using origin/importance, then specificity, then source order.
- Specificity is scored as (a, b, c): IDs, then classes/attributes/pseudo-classes, then elements.
- Compare scores left to right; a higher number on the left always wins.
- Inline styles outrank stylesheet rules; `!important` outranks normal specificity.
- Avoid `!important` — it breaks the cascade and starts override wars.
- Text properties inherit; box and layout properties usually do not.
- `inherit`, `initial`, and `unset` let you control inheritance directly.
- Prefer classes and flat selectors to keep specificity low and CSS maintainable.

---

[← Back to Course Index](../README.md) | [Next: Pseudo-classes and Pseudo-elements →](02-pseudo-classes-and-elements.md)
