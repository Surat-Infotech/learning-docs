# Selectors

A **selector** is how you tell CSS which elements to style. It is like pointing at things on the page and saying "this one, and that one." Learning selectors well is one of the most useful CSS skills, because once you can target the right elements, styling them is easy.

## What You'll Learn

- How to select elements by type, class, and id
- The universal selector and how to group selectors
- How to combine selectors with combinators
- How to select by attribute
- A first look at pseudo-classes
- The difference between classes and ids, and why classes are preferred
- How to combine selectors for precise targeting

## The Type (Element) Selector

The simplest selector is the **type selector**, also called the **element selector**. You just write the tag name. It targets every element of that type.

```css
p {
  color: gray;
}
```

This styles every `<p>` on the page.

```html
<p>This paragraph is gray.</p>
<p>So is this one.</p>
```

Both paragraphs turn gray. The type selector is broad: it hits all matching elements.

## The Class Selector

A **class** is a label you give to elements. Many elements can share the same class. You target a class in CSS with a dot `.` followed by the class name.

First, add a `class` attribute in your HTML:

```html
<p class="highlight">Important note.</p>
<p>A normal paragraph.</p>
<span class="highlight">Also important.</span>
```

Then style the class:

```css
.highlight {
  background-color: yellow;
  font-weight: bold;
}
```

Both the paragraph and the span with `class="highlight"` get a yellow background and bold text. The plain paragraph does not.

Classes are reusable. You can put the same class on as many elements as you like, and an element can have several classes at once:

```html
<p class="highlight rounded">A note with two classes.</p>
```

Separate multiple classes with a space.

## The Id Selector

An **id** is a unique label. Only one element on the page should have a given id. You target an id with a hash `#` followed by the id name.

```html
<div id="header">The top of the page</div>
```

```css
#header {
  background-color: navy;
  color: white;
}
```

An id should appear only once per page. Use ids for one-of-a-kind elements, like the main header or a specific section.

## The Universal Selector

The **universal selector** is an asterisk `*`. It targets every single element on the page.

```css
* {
  margin: 0;
  padding: 0;
}
```

This is often used to reset default spacing across the whole page. Use it carefully, since it touches everything.

## Grouping Selectors

If several selectors share the same styles, you do not need to repeat yourself. Separate them with commas to group them.

```css
h1,
h2,
h3 {
  color: darkblue;
  font-family: Georgia, serif;
}
```

This one rule styles every `<h1>`, `<h2>`, and `<h3>` the same way. Without grouping, you would write the same declarations three times.

## Combinators

**Combinators** let you select elements based on their relationship to other elements. There are four common ones.

### Descendant Combinator (space)

A space between two selectors means "any element inside another." This is the **descendant combinator**.

```css
article p {
  color: gray;
}
```

This selects every `<p>` that is *anywhere* inside an `<article>`, no matter how deeply nested.

```html
<article>
  <p>Selected.</p>
  <div>
    <p>Also selected, even though it is deeper.</p>
  </div>
</article>
<p>Not selected, because it is outside the article.</p>
```

### Child Combinator `>`

The greater-than sign `>` means "a direct child only." This is the **child combinator**.

```css
ul > li {
  list-style: square;
}
```

This selects only `<li>` elements that are *direct* children of a `<ul>`, not ones nested deeper inside another list.

```html
<ul>
  <li>Direct child. Selected.</li>
  <li>
    Direct child. Selected.
    <ul>
      <li>Grandchild. Not selected by this rule.</li>
    </ul>
  </li>
</ul>
```

### Adjacent Sibling Combinator `+`

The plus sign `+` selects an element that comes *immediately after* another, sharing the same parent. This is the **adjacent sibling combinator**.

```css
h2 + p {
  font-weight: bold;
}
```

This makes only the first paragraph right after an `<h2>` bold.

```html
<h2>Title</h2>
<p>This paragraph is bold (it comes right after the h2).</p>
<p>This one is normal.</p>
```

### General Sibling Combinator `~`

The tilde `~` selects *all* siblings that come after an element, not just the first. This is the **general sibling combinator**.

```css
h2 ~ p {
  color: teal;
}
```

```html
<h2>Title</h2>
<p>Teal.</p>
<span>Not a paragraph, skipped.</span>
<p>Also teal.</p>
```

Both paragraphs after the `<h2>` turn teal, even with something in between.

## Attribute Selectors

You can select elements based on their attributes using square brackets `[ ]`. This is an **attribute selector**.

```css
input[type="text"] {
  border: 1px solid gray;
}
```

This selects only `<input>` elements whose `type` is exactly `text`.

```html
<input type="text" />
<!-- selected -->
<input type="password" />
<!-- not selected -->
```

You can also select by just the presence of an attribute:

```css
a[target] {
  color: green;
}
```

This selects every link that has a `target` attribute, no matter its value.

## A First Look at Pseudo-Classes

A **pseudo-class** targets elements in a special state or position. You write it with a colon `:`. Here are two common ones.

`:hover` styles an element when the mouse is over it:

```css
button:hover {
  background-color: lightblue;
}
```

The button changes color only while the pointer is on it.

`:first-child` styles an element only if it is the first child of its parent:

```css
li:first-child {
  font-weight: bold;
}
```

The first item in a list becomes bold.

There are many more pseudo-classes, like `:last-child`, `:focus`, and `:nth-child()`. We will cover more of them in later lessons. For now, just know they exist and follow this colon pattern.

## Class vs Id: Why Classes Win

Classes and ids both let you label elements. So which should you use? In most cases, prefer **classes**. Here is why:

- **Reusable.** A class can be used on many elements. An id is meant for just one.
- **Flexible.** An element can have several classes, but only one id.
- **Lower stakes.** Ids carry more "weight" in the cascade, which can make styles harder to override later. Classes are easier to manage.

A common rule of thumb: use classes for styling, and save ids for unique anchors or JavaScript hooks.

## Combining Selectors

You can stack selectors with no space to require multiple conditions on the same element.

```css
p.intro {
  font-size: 20px;
}
```

This selects only `<p>` elements that *also* have the class `intro`. A plain `<p>` or a `<div class="intro">` would not match.

You can combine even more:

```css
ul li.active {
  color: red;
}
```

This selects `<li>` elements with the class `active` that are inside a `<ul>`.

The more selectors you combine, the more specific (and exact) your target becomes.

## Common Mistakes

- **Forgetting the dot or hash.** `highlight` selects a `<highlight>` tag (which does not exist). You need `.highlight` for a class.
- **Reusing an id.** Ids should be unique. Using the same id twice can cause confusing behavior.
- **Confusing descendant and child.** `div p` selects all paragraphs inside a div, but `div > p` selects only direct children.
- **Adding a space when combining.** `p.intro` (no space) means a `p` with class `intro`. `p .intro` (with space) means an element with class `intro` inside a `p`. These are very different.
- **Missing the quotes in attribute selectors.** Write `input[type="text"]`, with quotes around the value.

## Exercises

1. Create three paragraphs. Give two of them the class `note` and style that class with a colored background.
2. Make a navigation bar of links. Use a `:hover` pseudo-class so links change color when hovered.
3. Build a list and use `li:first-child` to make only the first item bold.
4. Use a child combinator `>` to style only the direct `<li>` children of a `<ul>`, then add a nested list and confirm the inner items are not affected.
5. Use an attribute selector to give all text inputs a border, but leave password inputs alone.

## Key Takeaways

- Selectors choose which elements a rule applies to.
- Type selectors target tags, class selectors use `.`, and id selectors use `#`.
- The universal selector `*` targets everything; commas group selectors together.
- Combinators select by relationship: descendant (space), child `>`, adjacent sibling `+`, and general sibling `~`.
- Attribute selectors use square brackets, like `[type="text"]`.
- Pseudo-classes like `:hover` and `:first-child` target special states and positions.
- Prefer classes over ids for styling because they are reusable and easier to manage.

---

[← Back to Course Index](../README.md) | [Next: Colors and Units →](03-colors-and-units.md)
