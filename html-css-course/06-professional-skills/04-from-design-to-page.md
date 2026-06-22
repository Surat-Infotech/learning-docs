# From Design to Page

This is the capstone lesson, where everything you have learned comes together. A designer hands you a picture of a website. Your job is to turn that picture into real, working HTML and CSS. In this lesson you will learn a clear, repeatable process for doing exactly that, and you will build a complete landing page step by step.

## What You'll Learn

- How to read a design and pull out its key details
- How to plan your HTML structure before writing styles
- A mobile-first build order that keeps things simple
- How to extract a color palette and spacing scale into variables
- How to build reusable components
- A full walkthrough of building a landing page
- What to learn next after this course

## How to Read a Design

A **design** (or **mockup**) is a picture of what the finished page should look like. Before you write any code, study it like a detective. You are looking for patterns.

Ask yourself these questions:

- **Colors:** What are the main colors? Usually there is one brand color, a dark text color, a light background, and maybe an accent.
- **Fonts:** How many fonts are used? What sizes are the headings versus the body text?
- **Spacing:** How much space sits between sections? Does the spacing seem to follow a pattern, like multiples of 8 pixels?
- **Components:** What pieces repeat? Buttons, cards, and navigation links often appear many times in the same style.

The goal is to see the design not as one big picture, but as a small set of reusable parts and consistent values. That insight makes the build far easier.

## Plan the HTML Structure First

A great habit is **content-out** thinking: figure out *what content* the page holds before you worry about *how it looks*. Write the HTML structure first, with no CSS at all.

For a landing page, the structure might be:

```html
<header>
  <!-- logo and navigation -->
</header>

<main>
  <section class="hero">
    <!-- big headline and a button -->
  </section>

  <section class="features">
    <!-- a grid of feature cards -->
  </section>
</main>

<footer>
  <!-- small print and links -->
</footer>
```

Notice this uses **semantic tags** (`header`, `main`, `section`, `footer`) that describe their meaning. If your page reads sensibly with no styles at all, you have a solid foundation.

## A Mobile-First Build Order

We will build **mobile-first**: style the small phone view first, then add styles for bigger screens. A good order to work in:

```text
1. Write all the HTML.
2. Add a reset and your CSS variables.
3. Style the base (phone) layout, section by section.
4. Build each reusable component (button, card).
5. Add media queries to adjust the layout for wider screens.
6. Polish: spacing, colors, hover states.
```

Working top to bottom and small to large keeps you from feeling overwhelmed.

## Extract a Palette and Spacing Scale

Before styling, collect your design's values into **CSS variables** at the top of your stylesheet. This gives you a single source of truth.

```css
:root {
  /* Color palette */
  --color-brand: #2563eb;
  --color-text: #1f2937;
  --color-muted: #6b7280;
  --color-bg: #ffffff;
  --color-light: #f3f4f6;

  /* Spacing scale (multiples that feel consistent) */
  --space-1: 8px;
  --space-2: 16px;
  --space-3: 24px;
  --space-4: 48px;
  --space-5: 80px;

  /* Type */
  --font-body: system-ui, sans-serif;
}
```

Now every color and every gap in the page comes from this list. If the brand color changes, you edit one line.

## Build Reusable Components

A **component** is a styled piece you use again and again, like a button. Build it once with a class, then reuse it everywhere.

```css
.button {
  display: inline-block;
  padding: var(--space-1) var(--space-2);
  background: var(--color-brand);
  color: white;
  text-decoration: none;
  border-radius: 6px;
  font-family: var(--font-body);
}

.button:hover {
  opacity: 0.9;
}
```

```html
<a href="#" class="button">Get Started</a>
```

Because this button uses variables, it automatically matches the rest of the page. Build your card component the same way.

## Full Walkthrough: A Landing Page

Now let's build a complete page: a hero, a features grid, and a footer. Follow along and type it yourself.

### Step 1: The HTML

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>BrightApp</title>
  <link rel="stylesheet" href="styles.css">
</head>
<body>
  <header class="site-header">
    <a href="#" class="logo">BrightApp</a>
    <nav class="nav">
      <a href="#features">Features</a>
      <a href="#" class="button">Sign Up</a>
    </nav>
  </header>

  <main>
    <section class="hero">
      <h1 class="hero__title">Get more done, faster.</h1>
      <p class="hero__text">A simple tool that keeps your team organized.</p>
      <a href="#" class="button">Start Free</a>
    </section>

    <section class="features" id="features">
      <article class="card">
        <h2 class="card__title">Fast</h2>
        <p class="card__text">Built for speed from day one.</p>
      </article>
      <article class="card">
        <h2 class="card__title">Simple</h2>
        <p class="card__text">No manual needed. Just start.</p>
      </article>
      <article class="card">
        <h2 class="card__title">Reliable</h2>
        <p class="card__text">Always there when you need it.</p>
      </article>
    </section>
  </main>

  <footer class="site-footer">
    <p>© 2026 BrightApp. All rights reserved.</p>
  </footer>
</body>
</html>
```

Notice the `<meta name="viewport">` tag. It tells phones to use their real width, which is essential for mobile-first design to work.

### Step 2: Reset and Variables

```css
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

:root {
  --color-brand: #2563eb;
  --color-text: #1f2937;
  --color-muted: #6b7280;
  --color-light: #f3f4f6;
  --space-2: 16px;
  --space-3: 24px;
  --space-4: 48px;
  --space-5: 80px;
  --font-body: system-ui, sans-serif;
}

body {
  font-family: var(--font-body);
  color: var(--color-text);
  line-height: 1.5;
}
```

### Step 3: The Header

```css
.site-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: var(--space-2) var(--space-3);
}

.logo {
  font-weight: bold;
  font-size: 20px;
  text-decoration: none;
  color: var(--color-text);
}

.nav {
  display: flex;
  align-items: center;
  gap: var(--space-2);
}

.nav a {
  text-decoration: none;
  color: var(--color-text);
}
```

We use **flexbox** (`display: flex`) to put the logo on the left and the navigation on the right, with `justify-content: space-between` pushing them apart.

### Step 4: The Button Component

```css
.button {
  display: inline-block;
  padding: 10px var(--space-2);
  background: var(--color-brand);
  color: white;
  text-decoration: none;
  border-radius: 6px;
}

.button:hover {
  opacity: 0.9;
}
```

### Step 5: The Hero

```css
.hero {
  text-align: center;
  padding: var(--space-5) var(--space-3);
  background: var(--color-light);
}

.hero__title {
  font-size: 32px;
  margin-bottom: var(--space-2);
}

.hero__text {
  color: var(--color-muted);
  margin-bottom: var(--space-3);
}
```

The hero is the big welcoming section at the top. We center its text and give it generous padding so it feels open.

### Step 6: The Features Grid

On a phone, the cards stack in one column. We build that first.

```css
.features {
  display: grid;
  gap: var(--space-3);
  padding: var(--space-5) var(--space-3);
}

.card {
  padding: var(--space-3);
  background: white;
  border: 1px solid var(--color-light);
  border-radius: 8px;
}

.card__title {
  margin-bottom: var(--space-2);
}

.card__text {
  color: var(--color-muted);
}
```

Here we use **CSS grid** (`display: grid`). With no column settings, the grid stacks items in one column, which is perfect for phones.

### Step 7: The Footer

```css
.site-footer {
  text-align: center;
  padding: var(--space-4) var(--space-3);
  color: var(--color-muted);
  border-top: 1px solid var(--color-light);
}
```

### Step 8: Make It Responsive

So far everything is a single column, great on phones. Now we add one **media query** so the feature cards sit side by side on wider screens.

```css
@media (min-width: 768px) {
  .features {
    grid-template-columns: repeat(3, 1fr);
  }

  .hero__title {
    font-size: 48px;
  }
}
```

`grid-template-columns: repeat(3, 1fr)` makes three equal columns. The `1fr` unit means "one equal fraction of the space." On big screens the three cards now line up in a row; on phones they still stack. We also bump up the hero title size so it feels bold on desktop.

That is a complete, responsive landing page built from a design, using semantic HTML, variables, components, and a mobile-first media query. Everything from this course working together.

## Next Steps After This Course

You now have a real, valuable skill. Here is where to go next.

- **JavaScript:** This is the language that makes pages *interactive*. With it you can respond to clicks, validate forms, fetch data, and build apps. It is the natural next step after HTML and CSS.
- **Git and GitHub:** **Git** is a tool that saves the history of your code so you can undo mistakes and work with others. **GitHub** is a website for storing and sharing that code. Almost every job expects you to know the basics.
- **Frameworks:** Once you know JavaScript, tools like **React** help you build bigger apps by reusing components, much like the components you built here.
- **Keep building:** The fastest way to improve is to build real projects. Recreate websites you admire. Each one teaches you something new.

You started not knowing how a web page is made. Now you can take a design and build it. That is a genuine accomplishment. Keep going, build often, and stay curious.

## Common Mistakes

- Jumping straight into CSS before planning the HTML structure.
- Picking random colors and spacing instead of extracting a palette and scale.
- Designing desktop-first, then struggling to squeeze it onto phones.
- Copying the same styles instead of building reusable components.
- Forgetting the viewport meta tag, so the mobile layout never works.
- Trying to build the whole page at once instead of section by section.

## Exercises

1. Find a simple website you like and write down its colors, fonts, spacing pattern, and repeating components.
2. Build the landing page from this lesson by typing it out, then change the color palette by editing only the variables.
3. Add a fourth feature card and adjust the grid so it still looks balanced.
4. Add a second media query at `min-width: 1024px` that increases spacing for very large screens.
5. Plan and build your own one-page site (about you, a hobby, or a fake product) using the full process from this lesson.

## Key Takeaways

- Read a design as a set of reusable parts and consistent values.
- Plan and write semantic HTML before any styling (content-out).
- Build mobile-first, then enhance for larger screens.
- Store colors and spacing in CSS variables as a single source of truth.
- Build reusable components so the page stays consistent.
- A media query turns a stacked phone layout into a multi-column desktop one.
- Next, learn JavaScript, Git, and a framework, and keep building projects.

---

[← Back to Course Index](../README.md)
