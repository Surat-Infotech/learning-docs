# Links and Navigation

Links are what turn a pile of separate pages into a connected **web**. They let visitors jump from one page to another, send an email, or skip to a section further down. In this lesson you will learn how to create links and how to build a simple navigation menu.

## What You'll Learn

- How to create a link with the `<a>` element and its `href` attribute
- The difference between **absolute** and **relative** URLs
- How to link between your own pages
- How to open links in a new tab safely with `target` and `rel`
- How to link to an email address and a phone number
- How to jump to a section of a page using `id` and `#`
- What makes link text clear and accessible
- How to build a basic navigation menu with `<nav>`

## The Anchor Element

A link is made with the **anchor element**, written as `<a>`. (It is called "anchor" for historical reasons.) The most important attribute is `href`, which stands for "hypertext reference." It tells the browser **where** the link goes.

```html
<a href="https://example.com">Visit Example</a>
```

This shows the words "Visit Example" as a clickable link. The content between the tags is what the user sees and clicks; the `href` is the destination.

A link without an `href` is not really a link, so always include it.

## Absolute vs Relative URLs

A **URL** is a web address. There are two kinds you will use.

### Absolute URLs

An **absolute URL** is the full address, starting with `https://`. It works from anywhere, just like a complete postal address with country, city, and street.

```html
<a href="https://www.wikipedia.org">Go to Wikipedia</a>
```

Use absolute URLs to link to **other** websites.

### Relative URLs

A **relative URL** is a short address that points to another file in *your own* project, relative to the page you are on. It is like saying "the house next door" instead of giving the full address.

```html
<a href="about.html">About Us</a>
<a href="pages/contact.html">Contact</a>
```

Some helpful patterns for relative paths:

```text
about.html        a file in the same folder
pages/contact.html  a file inside the "pages" subfolder
../index.html     a file one folder up ("../" means go up)
/index.html       start from the site's root folder
```

## Linking Between Your Own Pages

Most websites have several pages. You connect them with relative links. Imagine you have these files:

```text
index.html
about.html
pages/contact.html
```

From `index.html`, you can link like this:

```html
<a href="about.html">About</a>
<a href="pages/contact.html">Contact</a>
```

From `pages/contact.html`, to get back to the home page you go up one folder:

```html
<a href="../index.html">Home</a>
```

## Opening Links in a New Tab

By default, a link replaces the current page. To open a link in a **new tab**, add `target="_blank"`.

When you do this, you should also add `rel="noopener noreferrer"` for security and performance. Without it, the new page can gain a small amount of control over your page, which can be a safety risk.

```html
<a href="https://example.com" target="_blank" rel="noopener noreferrer">
  Open Example in a new tab
</a>
```

Tip: only open new tabs when it genuinely helps, such as linking to an outside site you don't want people to lose their place on. Too many new tabs annoy users.

## Email and Phone Links

You can make a link that opens the user's email app using `mailto:` followed by an email address.

```html
<a href="mailto:hello@example.com">Email us</a>
```

You can even pre-fill a subject:

```html
<a href="mailto:hello@example.com?subject=Hello">Email us</a>
```

For phone numbers, use `tel:`. On a phone, clicking it starts a call.

```html
<a href="tel:+15551234567">Call us</a>
```

Use the full number including the country code (the `+` part) so it works everywhere.

## Anchor Links: Jumping to a Section

You can link to a specific spot on a page, not just the top. This is great for long pages with a table of contents.

First, give the target element an `id` attribute. An **id** is a unique name for that one element.

```html
<h2 id="pricing">Pricing</h2>
```

Then link to it using `#` followed by the id:

```html
<a href="#pricing">Jump to Pricing</a>
```

Clicking the link scrolls the page down to the "Pricing" heading. To jump to a section on a *different* page, combine the file name and the id:

```html
<a href="about.html#team">Meet the team</a>
```

A link with just `href="#top"` (with `id="top"` at the top of the page) is a common "back to top" pattern.

## Writing Good Link Text

The words inside a link matter a lot, especially for people using screen readers, who often listen to a list of all the links on a page. The text should make sense on its own.

Bad, because it tells the user nothing:

```html
<a href="report.pdf">Click here</a>
```

Good, because it describes the destination:

```html
<a href="report.pdf">Download the 2026 annual report</a>
```

Aim for short, descriptive link text. Avoid "click here," "read more," or a bare URL whenever you can.

## Download Links

To make a link download a file instead of opening it, add the `download` attribute.

```html
<a href="brochure.pdf" download>Download our brochure</a>
```

You can also suggest a file name for the saved file:

```html
<a href="brochure.pdf" download="company-brochure.pdf">Download brochure</a>
```

## A Simple Navigation Menu

A **navigation menu** is the set of links that lets people move around your site. HTML has a special element, `<nav>`, that marks an area as navigation. Inside it, links are usually placed in a list (you will learn lists in detail soon).

```html
<nav>
  <ul>
    <li><a href="index.html">Home</a></li>
    <li><a href="about.html">About</a></li>
    <li><a href="pages/contact.html">Contact</a></li>
  </ul>
</nav>
```

The `<nav>` element tells screen readers and search engines "these are the main links for getting around." Using a list keeps the links organized and makes them easy to style into a menu bar later with CSS.

## Common Mistakes

- Writing an `<a>` tag with no `href`, so it isn't clickable.
- Mixing up absolute and relative URLs, leading to broken links.
- Using `target="_blank"` without `rel="noopener noreferrer"`.
- Forgetting the matching `id` when using a `#` anchor link.
- Writing vague link text like "click here."
- Forgetting the `mailto:` or `tel:` prefix on email and phone links.

## Exercises

1. Create two pages, `index.html` and `about.html`, and add links between them.
2. Add a link to an outside website that opens in a new tab, safely.
3. Add an email link and a phone link to your page.
4. Make a long page with two sections that each have an `id`, then add links at the top that jump to each section.
5. Build a `<nav>` menu with a list of three links to different pages of your site.

## Key Takeaways

- The `<a>` element with `href` creates a link.
- **Absolute URLs** point to other sites; **relative URLs** point within your own project.
- Use `target="_blank"` with `rel="noopener noreferrer"` to open new tabs safely.
- `mailto:` and `tel:` create email and phone links.
- `id` plus `#` lets you jump to a section of a page.
- Link text should clearly describe where the link goes.
- Wrap your site's main links in a `<nav>` with a list.

---

[← Back to Course Index](../README.md) | [Next: Images and Media →](04-images-and-media.md)
