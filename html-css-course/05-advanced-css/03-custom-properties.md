# Custom Properties (CSS Variables)

Imagine you used the same blue color in fifty places across your stylesheet. Then your designer says, "Let's make it a slightly different blue." Now you're hunting through fifty lines. **Custom properties**, also called **CSS variables**, solve this. You define a value once, give it a name, and reuse it everywhere. Change it in one place, and the whole site updates.

## What You'll Learn

- What CSS variables are and why they're so useful
- How to declare a variable with `--name` and use it with `var()`
- The `:root` convention for site-wide variables
- Fallback values for when a variable is missing
- How variable scope and inheritance work
- How to build a reusable theme of colors and spacing
- How to switch themes (like dark mode) by changing variables
- How CSS variables differ from Sass variables

## What Is a CSS Variable?

A **CSS variable** is a named value you can store and reuse throughout your stylesheet. Instead of repeating `#2563eb` everywhere, you store it once under a friendly name like `--primary-color`, then refer to that name.

This gives you two big wins:

- **Less repetition.** Write the value once.
- **Easy changes.** Update the value in one spot, and every use updates.

## Declaring and Using Variables

You declare a variable by writing a name that starts with two dashes (`--`), then a value. You use it with the `var()` function.

```css
.button {
  --button-bg: #2563eb;          /* declare */
  background-color: var(--button-bg); /* use */
}
```

The double dash is required. `--button-bg` is a variable; `button-bg` is not. The names are **case-sensitive**, so `--Primary` and `--primary` are different.

## The `:root` Convention

Most of the time you want a variable available across your **whole** site. The standard place to declare these is the **`:root`** selector.

`:root` targets the highest element in the document, which is the `<html>` element. Declaring variables there makes them available everywhere, because variables inherit down to every child.

```css
:root {
  --primary: #2563eb;
  --text: #111827;
  --space: 16px;
}

body {
  color: var(--text);
}

.card {
  padding: var(--space);
  border-top: 4px solid var(--primary);
}
```

Now `--primary`, `--text`, and `--space` work anywhere in your CSS. Putting your design choices at the top in `:root` is like having a settings panel for your whole site.

## Fallback Values

What if a variable isn't defined? You can give `var()` a **fallback**, a backup value to use when the variable is missing.

```css
.box {
  color: var(--accent, #333);
}
```

If `--accent` exists, the browser uses it. If it doesn't, the box text falls back to `#333`. This is a safety net that keeps your design from breaking when a variable is undefined.

You can even nest fallbacks:

```css
.box {
  color: var(--accent, var(--text, black));
}
```

This reads: use `--accent`; if missing, use `--text`; if that's missing too, use `black`.

## Scope and Inheritance

CSS variables follow the same **inheritance** rules as other properties. A variable declared on an element is available to that element and all its children, but not to its parents or unrelated siblings.

```css
:root {
  --color: blue;
}

.special {
  --color: red; /* override, just for .special and its children */
}
```

```html
<p style="color: var(--color)">I am blue.</p>

<div class="special">
  <p style="color: var(--color)">I am red.</p>
</div>
```

The first paragraph reads `--color` from `:root` and is blue. The second paragraph sits inside `.special`, where `--color` is redefined as red, so it inherits the red value. This **scoping** lets you override variables for just one section without touching the rest of the page.

## Building a Theme

The real power shows up when you collect all your design decisions into one set of variables. This is your **theme** or **design system**.

```css
:root {
  /* Colors */
  --color-primary: #2563eb;
  --color-text: #111827;
  --color-muted: #6b7280;
  --color-bg: #ffffff;
  --color-surface: #f3f4f6;

  /* Spacing scale */
  --space-sm: 8px;
  --space-md: 16px;
  --space-lg: 32px;

  /* Other */
  --radius: 8px;
  --font-body: Arial, sans-serif;
}
```

Now your components read like a clear recipe:

```css
body {
  margin: 0;
  font-family: var(--font-body);
  color: var(--color-text);
  background: var(--color-bg);
}

.card {
  background: var(--color-surface);
  padding: var(--space-md);
  border-radius: var(--radius);
}

.card__title {
  color: var(--color-primary);
  margin-bottom: var(--space-sm);
}

.card__meta {
  color: var(--color-muted);
}
```

If your brand color changes, you edit one line: `--color-primary`. Every button, title, and border that uses it updates instantly.

## Switching Themes: Dark Mode Preview

Because variables can be redefined in a different scope, theme switching becomes easy. You redefine the same variable names under a new condition, and everything that uses them changes at once.

### With a Class

Add a class to `<body>` (often with JavaScript) and redefine the variables.

```css
:root {
  --color-bg: #ffffff;
  --color-text: #111827;
}

body.dark {
  --color-bg: #111827;
  --color-text: #f9fafb;
}

body {
  background: var(--color-bg);
  color: var(--color-text);
}
```

When `<body>` has the `dark` class, the variables flip, and the whole page redraws in dark colors. Your component CSS never changes — only the variable values do.

### With a Media Query

You can also respect the user's system setting using a media query.

```css
:root {
  --color-bg: #ffffff;
  --color-text: #111827;
}

@media (prefers-color-scheme: dark) {
  :root {
    --color-bg: #111827;
    --color-text: #f9fafb;
  }
}
```

`prefers-color-scheme: dark` is true when the user's operating system is set to dark mode. This gives a dark theme automatically, with no JavaScript.

## How These Differ from Sass Variables

You may hear about **Sass variables**, written like `$primary: blue`. They look similar but work very differently.

- **Sass variables are compiled away.** Sass runs as a build step before the browser ever sees your CSS. By the time the page loads, `$primary` has been replaced with a plain value. You **cannot** change it at runtime.
- **CSS custom properties are live in the browser.** They exist while the page runs. You can change them with JavaScript, with a class, or with a media query, and the page responds instantly.

That live nature is why CSS variables can power theme switching and Sass variables cannot. Both are useful; they just solve different problems. You can even use them together. In this course we focus on native CSS custom properties because they work in every modern browser with no build tools.

## Common Mistakes

- Forgetting the double dash: `var(--name)` works, `var(name)` does not.
- Mixing up case — `--Color` and `--color` are two different variables.
- Expecting a variable to be visible to a parent or sibling (variables only flow down).
- Declaring component-specific variables in `:root` when they only matter locally.
- Forgetting a fallback, so a missing variable silently breaks a value.
- Expecting CSS variables to behave like Sass variables (they're live, not compiled).

## Exercises

1. Create a `:root` block with at least three color variables and three spacing variables. Style a card component using only those variables.
2. Add a fallback to one `var()` call, then temporarily delete its declaration to confirm the fallback takes over.
3. Override one variable inside a `.highlight` section so only that section uses a different accent color, while the rest of the page stays the same.
4. Build a dark mode toggle using a `body.dark` class that redefines your color variables. Confirm the whole page updates.
5. Add a `prefers-color-scheme: dark` media query so your page automatically uses dark colors when the operating system is in dark mode.

## Key Takeaways

- CSS variables store reusable values you define once and use everywhere.
- Declare them with `--name` and read them with `var(--name)`.
- `:root` is the standard place for site-wide variables, because they inherit down.
- `var(--x, fallback)` provides a backup value if the variable is missing.
- Variables are scoped and inherited: redefine them on an element to override locally.
- A theme is just a tidy collection of variables for colors, spacing, and more.
- Redefining variables with a class or media query powers easy theme switching like dark mode.
- Unlike compiled Sass variables, CSS variables are live and changeable in the browser.

---

[← Back to Course Index](../README.md) | [Next: Transitions and Transforms →](04-transitions-and-transforms.md)
