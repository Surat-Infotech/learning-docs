# Accessibility Basics

Accessibility means building websites that everyone can use, including people with disabilities. It is one of the most important and rewarding skills a web developer can have. The good news is that much of it comes naturally when you write clean, semantic HTML. In this lesson you will learn the practical basics.

## What You'll Learn

- What accessibility (a11y) is and who it helps
- Why semantic HTML is the foundation of an accessible page
- How to write good `alt` text for images
- Why every form field needs a label
- How keyboard navigation and focus order work
- Why visible focus styles and good color contrast matter
- A light introduction to `aria-label` and ARIA roles
- How to add a skip link and test with a keyboard or screen reader

## What Is Accessibility?

**Accessibility** means making sure your website works for as many people as possible, including those with disabilities. You will often see it written as **a11y**, which is short for "accessibility" (the letter a, then 11 letters, then y).

Who does it help? Many people, such as:

- People who are blind or have low vision and use a **screen reader** (software that reads the page aloud).
- People who cannot use a mouse and navigate with only a keyboard.
- People who are color blind and rely on more than color to understand things.
- People with motor difficulties who need large, easy-to-click targets.

And honestly, it helps *everyone*. Captions help in a noisy room. Good contrast helps in bright sunlight. Accessibility is just good, thoughtful design.

## Semantic HTML Is the Foundation

Here is the most encouraging fact: if you write **semantic HTML**, you have already done much of the accessibility work.

When you use a real `<button>`, it is automatically clickable, focusable with the keyboard, and announced as a button by screen readers. When you use `<nav>`, `<main>`, and `<header>`, screen reader users can jump straight to those regions.

```html
<!-- Good: a real button, accessible for free -->
<button>Save</button>

<!-- Bad: a div pretending to be a button, accessible to nobody -->
<div class="button">Save</div>
```

The fake button in the second example cannot be focused or pressed with a keyboard, and a screen reader will not call it a button. **Always use the native HTML element built for the job.** This single habit prevents most accessibility problems.

## Writing Good `alt` Text

The **`alt`** attribute on an image gives a text description for people who cannot see it. A screen reader reads the `alt` text aloud, and it also shows up if the image fails to load.

```html
<img src="dog.jpg" alt="A brown dog catching a frisbee in a park">
```

Good `alt` text describes what the image *shows* and *means*, briefly.

- Describe the content: "A bar chart showing sales rising from 2024 to 2026."
- Do not start with "Image of..." (the screen reader already says it is an image).
- Keep it short but useful.

If an image is purely decorative and adds no information (like a swirl or a background flourish), give it an **empty** `alt` so screen readers skip it:

```html
<img src="divider.png" alt="">
```

An empty `alt=""` is correct for decoration. Leaving `alt` off entirely is a mistake, because then some screen readers read out the file name instead.

## Labels for Form Fields

Every form field needs a **label**, as you learned in the forms lesson. This is an accessibility essential. Without a connected label, a screen reader user reaches a text box with no idea what to type.

```html
<label for="email">Email address</label>
<input type="email" id="email" name="email">
```

Remember to match `for` and `id`. A placeholder is *not* a label, because it disappears when the user starts typing and screen readers may not announce it reliably.

## Keyboard Navigation and Focus Order

Many people use only a keyboard, not a mouse. They move through a page by pressing the **Tab** key, which jumps from one interactive element (links, buttons, fields) to the next.

The element the keyboard is currently "on" is said to have **focus**. The order in which Tab moves through elements is the **focus order**.

The key rule: **focus order should follow the visual order**, top to bottom, left to right. The browser does this automatically when your HTML is in a sensible order. If you write your HTML in the same order things appear, keyboard users will move through your page naturally.

Try it yourself right now: open any web page and press Tab a few times. Watch where the highlight jumps.

## Visible Focus Styles

When an element has focus, the browser draws an outline around it so keyboard users can see where they are. This is the **focus indicator**.

Some developers remove this outline because they think it looks ugly:

```css
/* Please do NOT do this */
button:focus {
  outline: none;
}
```

Removing the focus outline leaves keyboard users completely lost, with no way to tell where they are on the page. If the default outline does not fit your design, **replace it with a clear one** instead of deleting it:

```css
button:focus {
  outline: 3px solid #0066cc;
  outline-offset: 2px;
}
```

The rule: a visible focus style is required, not optional.

## Color Contrast

**Color contrast** is the difference in brightness between text and its background. Low contrast (like light gray text on a white background) is hard to read, especially for people with low vision.

```css
/* Hard to read: too little contrast */
color: #aaaaaa;
background: #ffffff;

/* Easy to read: strong contrast */
color: #222222;
background: #ffffff;
```

There is a recommended minimum from the accessibility guidelines (a contrast ratio of at least 4.5 to 1 for normal text). You do not need to memorize the math. Free online "contrast checkers" let you paste in two colors and tell you instantly if they pass.

Also, never use **color alone** to convey meaning. For example, do not show errors only in red, because color-blind users may not see it. Add an icon or text too:

```html
<p>Error: please enter your email.</p>
```

## A Light Introduction to ARIA

**ARIA** stands for Accessible Rich Internet Applications. It is a set of extra attributes you can add to HTML to give assistive tools more information. Two common pieces are below.

The **`aria-label`** attribute provides a text label for an element when there is no visible text, often for icon-only buttons.

```html
<button aria-label="Close">X</button>
```

A screen reader will now say "Close" instead of just "X".

An **ARIA role** tells assistive tools what an element is. For example, `role="navigation"` says "this is navigation".

```html
<div role="navigation"> ... </div>
```

But here is the golden rule of ARIA, and please remember it:

> **Use native HTML first. Only reach for ARIA when no HTML element does the job.**

The `<div role="navigation">` above is unnecessary, because `<nav>` already means navigation, for free and more reliably. ARIA is powerful, but it is easy to misuse, and bad ARIA is worse than none. Most of the time, a correct semantic element is all you need.

## Headings Structure

Screen reader users often navigate a page by jumping from heading to heading, the way a sighted person scans for bold titles. For this to work, your headings must be in a logical order.

- Use one `<h1>` for the page's main title.
- Use `<h2>` for main sections, `<h3>` for subsections, and so on.
- Do not skip levels (no jumping from `<h2>` to `<h4>`).
- Do not pick a heading level just because of its size. Use CSS for size; use heading levels for *structure*.

```html
<h1>Recipe: Banana Bread</h1>
  <h2>Ingredients</h2>
  <h2>Steps</h2>
    <h3>Mixing</h3>
    <h3>Baking</h3>
```

This creates a clear outline that everyone can navigate.

## Skip Links

Most pages repeat the same menu at the top. For a keyboard user, tabbing through that whole menu on every page is tiring. A **skip link** is a hidden link at the very top that jumps straight to the main content.

```html
<body>
  <a href="#main-content" class="skip-link">Skip to main content</a>

  <header> ... navigation ... </header>

  <main id="main-content">
    ... the real content ...
  </main>
</body>
```

The link is usually hidden until a keyboard user tabs to it, at which point it appears. Clicking or activating it moves focus straight to `#main-content`. It is a small touch that keyboard users greatly appreciate.

## Testing Your Work

You do not need fancy tools to start testing. Two simple methods catch many problems.

**Test with the keyboard only.** Put your mouse aside. Use Tab to move forward, Shift+Tab to move back, and Enter or Space to activate things. Ask yourself:

- Can I reach every link, button, and field?
- Can I always see where the focus is?
- Does the order make sense?

**Test with a screen reader.** Your computer already has one built in:

- On Mac, turn on **VoiceOver** (Command + F5).
- On Windows, use **Narrator**, or the free, popular **NVDA**.

Close your eyes and listen. Does the page make sense? Are images described? Are form fields announced with their labels? It feels strange at first, but it teaches you more about accessibility than anything else.

Do not aim for perfection on day one. Every improvement helps a real person. Build the good habits in this lesson, and accessibility will become a natural part of how you write HTML.

## Common Mistakes

- Using `<div>` with a click handler instead of a real `<button>` or `<a>`.
- Removing the focus outline with `outline: none` and not replacing it.
- Leaving images without `alt`, or writing useless `alt` like "image1".
- Using a placeholder as if it were a label.
- Using color alone to show meaning, like errors only in red.
- Reaching for ARIA when a plain semantic element would do the job better.
- Skipping heading levels or choosing headings by their size instead of structure.

## Exercises

1. Take a page and tab through it with only your keyboard. Write down any element you cannot reach or any place where you lose track of the focus.
2. Add meaningful `alt` text to three images, and add an empty `alt=""` to one decorative image.
3. Write a CSS rule that gives buttons a clear, visible focus outline.
4. Add a "Skip to main content" link at the top of a page that jumps to your `<main>` element.
5. Turn on your computer's screen reader and listen to one of your pages. Note one thing that sounded confusing and one thing that worked well.

## Key Takeaways

- Accessibility (a11y) means building sites everyone can use; it helps a huge range of people.
- Semantic HTML is the foundation: native elements are accessible for free.
- Write `alt` text that describes the image, and use empty `alt=""` for decoration.
- Every form field needs a real, connected label.
- Support keyboard users: keep focus order logical and keep focus styles visible.
- Use strong color contrast and never rely on color alone for meaning.
- Use ARIA only when native HTML cannot do the job.
- Order your headings well, add a skip link, and test with a keyboard and a screen reader.

---

[← Back to Course Index](../README.md) | [Next: How CSS Works →](../03-css-fundamentals/01-how-css-works.md)
