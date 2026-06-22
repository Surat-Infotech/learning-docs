# Best Practices and Performance

Knowing HTML and CSS is one thing. Writing them *well* is another. Professional developers follow a set of habits that make their pages fast, accessible, and easy to maintain. This lesson collects those habits into one place, plus a checklist you can run before any site goes live.

## What You'll Learn

- How to write semantic, meaningful HTML
- How to keep your CSS DRY (without repeating yourself)
- Why mobile-first design is the default today
- How to use a consistent spacing scale
- Why to avoid inline styles and `!important`
- How to optimize images and load them lazily
- How web fonts affect performance
- What the critical render path is, in simple terms
- How to validate, test compatibility, and check accessibility

## Write Semantic HTML

**Semantic HTML** means using tags that describe their meaning, not just their look. A `<button>` is for buttons. A `<nav>` is for navigation. A `<header>` is for the top of a page or section.

```html
<!-- Avoid: meaningless boxes -->
<div class="header">
  <div class="nav">...</div>
</div>

<!-- Better: tags that describe themselves -->
<header>
  <nav>...</nav>
</header>
```

Semantic tags help **screen readers** (software that reads pages aloud for blind users), help search engines understand your page, and make your code easier to read. When a tag with the right meaning exists, use it.

## Keep Your CSS DRY

**DRY** stands for "Don't Repeat Yourself." If you copy the same styles over and over, one small change later means editing many places, and you will miss some.

```css
/* Repetitive */
.title-1 { color: #333; font-family: Arial; }
.title-2 { color: #333; font-family: Arial; }
.title-3 { color: #333; font-family: Arial; }

/* DRY: one rule, shared by all */
.title { color: #333; font-family: Arial; }
```

**CSS variables** (also called custom properties) are a great DRY tool. You define a value once and reuse it everywhere:

```css
:root {
  --brand-color: #2563eb;
  --space: 16px;
}

.button { background: var(--brand-color); }
.link   { color: var(--brand-color); }
```

Now changing the brand color is a one-line edit.

## Design Mobile-First

**Mobile-first** means you write your base styles for small phone screens, then add styles for larger screens on top. Most people browse on phones, so starting there keeps your core experience solid.

You do this with **media queries** that use `min-width`, which means "apply this when the screen is at least this wide":

```css
/* Base: phone styles first */
.layout {
  display: block;
}

/* Then enhance for wider screens */
@media (min-width: 768px) {
  .layout {
    display: flex;
  }
}
```

Building up from small to large is usually simpler than stripping a complex desktop layout down to a phone.

## Use a Consistent Spacing Scale

A **spacing scale** is a small set of allowed spacing values that you reuse everywhere, instead of picking random numbers like `7px` here and `13px` there.

```css
:root {
  --space-1: 4px;
  --space-2: 8px;
  --space-3: 16px;
  --space-4: 32px;
  --space-5: 64px;
}

.card { padding: var(--space-3); }
.section { margin-bottom: var(--space-5); }
```

A scale makes a design feel calm and consistent, because spacing follows a rhythm rather than looking accidental.

## Avoid Inline Styles and !important

An **inline style** is CSS written directly on an HTML element with the `style` attribute:

```html
<!-- Avoid this -->
<p style="color: red; margin-top: 20px;">Hello</p>
```

Inline styles are hard to reuse, hard to override, and scatter your styling across the HTML. Keep CSS in stylesheets and use classes instead.

`!important` is a flag that forces a style to win no matter what:

```css
.text { color: red !important; }
```

It works, but it is a trap. Once you use `!important`, the only way to beat it later is another `!important`, and soon you have a mess. Reach for it only as a rare last resort. Most of the time, fixing your **specificity** (using a better selector) is the real solution.

## Optimize Images

Images are usually the heaviest part of a page. A few simple steps make a huge difference.

**Choose the right format:**

- **JPG** for photos.
- **PNG** for images that need transparency or sharp lines.
- **SVG** for logos and icons (it stays crisp at any size).
- **WebP** is a modern format that is smaller than JPG or PNG with similar quality.

**Size them correctly.** Do not load a 4000-pixel-wide photo into a 400-pixel-wide spot. Resize the file to roughly the size it will display.

**Lazy-load offscreen images.** **Lazy loading** means the browser waits to download an image until the user is about to scroll to it. Add one attribute:

```html
<img src="photo.jpg" alt="A sunny beach" loading="lazy" width="800" height="600">
```

Setting `width` and `height` also helps: it reserves space so the page does not jump around as images load.

## Minimize and Combine CSS

**Minifying** means removing spaces, line breaks, and comments from your CSS so the file is smaller and downloads faster. The browser does not care about pretty formatting.

**Combining** means merging several CSS files into one, so the browser makes fewer requests. Each separate file is a separate trip to the server, and trips cost time. Build tools do both of these for you automatically when you prepare a site for launch.

## Web Fonts and Performance

**Web fonts** are custom fonts you load from a file or service, like Google Fonts. They make a page look great but can slow it down, and text may be invisible for a moment while the font loads.

Some tips:

- Load only the **weights** you actually use (for example regular and bold, not all eight).
- Use the `font-display: swap` setting so text shows in a fallback font immediately, then swaps to the real font when ready.

```css
@font-face {
  font-family: "MyFont";
  src: url("myfont.woff2") format("woff2");
  font-display: swap;
}
```

`woff2` is the most compressed, fastest web font format. Prefer it when you can.

## The Critical Render Path (in Simple Terms)

The **critical render path** is the sequence of steps the browser follows to turn your files into pixels on the screen. In simple terms:

```text
1. Download the HTML.
2. Download the CSS it links to.
3. Build the page and wait for that CSS before showing anything.
4. Download images, fonts, and scripts.
5. Paint the final page.
```

The key lesson: **CSS blocks rendering.** The browser will not show your page until it has the CSS. So keeping your CSS small and combined directly helps your page appear faster. Big, slow stylesheets delay everything the user sees.

## Validating Your HTML and CSS

A **validator** is a tool that checks your code for mistakes against the official rules. The **W3C** (the group that defines web standards) offers free ones:

- HTML validator: validator.w3.org
- CSS validator: jigsaw.w3.org/css-validator

Paste your code or a URL, and it lists errors like unclosed tags or invalid properties. Fixing these early prevents strange bugs and helps your page work in more browsers.

## Browser Compatibility and Graceful Degradation

Not every browser supports every feature. **Browser compatibility** is about making sure your site works across the browsers your visitors use.

**Graceful degradation** means that when a fancy feature is not supported, the page still works, just a little plainer. For example, if a browser does not support a modern layout feature, the content should still be readable, not broken.

The website **caniuse.com** tells you which browsers support a given CSS feature. Check it before relying on something new.

## Accessibility Checklist Recap

**Accessibility** means making your site usable by everyone, including people with disabilities. A quick recap of the essentials:

- Every image has a meaningful `alt` text (or empty `alt=""` if purely decorative).
- Text has enough **color contrast** against its background to be readable.
- The page works with keyboard only (you can `Tab` to every link and button).
- Form inputs have `<label>` elements connected to them.
- Headings go in order (`<h1>`, then `<h2>`, and so on) and describe the structure.

## A Pre-Launch Checklist

Before any site goes live, run through this list:

```text
[ ] HTML and CSS pass the W3C validators.
[ ] Every image has alt text and is sized correctly.
[ ] Offscreen images use loading="lazy".
[ ] The layout works on a phone (test in device mode).
[ ] No console errors in DevTools.
[ ] Links and buttons reachable by keyboard.
[ ] Color contrast is sufficient.
[ ] No leftover debug CSS (like outline: 1px solid red).
[ ] CSS is combined and minified for production.
[ ] Page tested in more than one browser.
```

## Common Mistakes

- Using `<div>` for everything instead of semantic tags.
- Repeating the same styles instead of sharing one class or variable.
- Reaching for `!important` instead of fixing specificity.
- Loading full-size images into small spaces.
- Loading many font weights you never use.
- Forgetting `alt` text and breaking accessibility.
- Skipping validation and shipping broken markup.

## Exercises

1. Take a page built only with `<div>` tags and rewrite it using semantic tags like `<header>`, `<nav>`, `<main>`, and `<footer>`.
2. Find repeated values in your CSS and replace them with CSS variables.
3. Add `loading="lazy"` plus `width` and `height` to every offscreen image on a page.
4. Run one of your pages through the W3C HTML validator and fix every error it reports.
5. Complete the full pre-launch checklist on a project of your own and note which items you had missed.

## Key Takeaways

- Semantic HTML helps accessibility, search engines, and readability.
- Keep CSS DRY with shared classes and variables.
- Build mobile-first and use `min-width` media queries.
- A consistent spacing scale makes designs feel intentional.
- Avoid inline styles and `!important`; fix specificity instead.
- Optimize images: right format, right size, and lazy loading.
- CSS blocks rendering, so keep it small and combined.
- Validate, test across browsers, and check accessibility before launch.

---

[← Back to Course Index](../README.md) | [Next: From Design to Page →](04-from-design-to-page.md)
