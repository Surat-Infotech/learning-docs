# Lists

Lists are everywhere: shopping lists, step-by-step recipes, menus, and more. HTML gives you a few kinds of lists so you can group related items in a clear, structured way. In this lesson you will learn all three list types and when to use each.

## What You'll Learn

- How to make an **unordered list** with `<ul>` and `<li>`
- How to make an **ordered list** with `<ol>` and `<li>`
- How to change numbering with the `type` and `start` attributes
- How to put a list inside another list (**nesting**)
- How to make a **description list** with `<dl>`, `<dt>`, and `<dd>`
- When to choose each kind of list
- How lists form the backbone of navigation menus

## Unordered Lists

An **unordered list** is a list where the order does not matter, like a grocery list. You can buy the items in any order. It uses two elements:

- `<ul>` ("unordered list") wraps the whole list.
- `<li>` ("list item") wraps each single item.

```html
<ul>
  <li>Milk</li>
  <li>Bread</li>
  <li>Eggs</li>
</ul>
```

By default, each item shows with a small bullet point (•) in front. On the page it looks like:

```text
• Milk
• Bread
• Eggs
```

Every item must be wrapped in its own `<li>`. The `<ul>` should contain only `<li>` elements directly.

## Ordered Lists

An **ordered list** is for items where the order *does* matter, like steps in a recipe. It uses:

- `<ol>` ("ordered list") wraps the whole list.
- `<li>` wraps each item (same as before).

```html
<ol>
  <li>Boil the water.</li>
  <li>Add the pasta.</li>
  <li>Cook for ten minutes.</li>
</ol>
```

The browser numbers the items automatically:

```text
1. Boil the water.
2. Add the pasta.
3. Cook for ten minutes.
```

The big advantage: if you add or remove a step, the numbers update by themselves. You never type the numbers yourself.

### The `type` Attribute

By default an ordered list uses numbers (1, 2, 3). The `type` attribute changes the style of marker.

```text
type="1"   1, 2, 3   (numbers, the default)
type="A"   A, B, C   (uppercase letters)
type="a"   a, b, c   (lowercase letters)
type="I"   I, II, III  (uppercase Roman numerals)
type="i"   i, ii, iii  (lowercase Roman numerals)
```

```html
<ol type="A">
  <li>First choice</li>
  <li>Second choice</li>
</ol>
```

This shows "A. First choice" and "B. Second choice."

### The `start` Attribute

By default an ordered list begins at 1. The `start` attribute lets you begin from a different number.

```html
<ol start="5">
  <li>This is item five.</li>
  <li>This is item six.</li>
</ol>
```

This is handy when a list continues across two separate blocks.

## Nested Lists

You can place a list **inside** a list item to create sub-items. This is called **nesting**. A nested list goes *inside* the `<li>`, after its text.

```html
<ul>
  <li>Fruits
    <ul>
      <li>Apple</li>
      <li>Banana</li>
    </ul>
  </li>
  <li>Vegetables
    <ul>
      <li>Carrot</li>
      <li>Potato</li>
    </ul>
  </li>
</ul>
```

On the page, the inner items are indented and often use a different bullet style, showing the hierarchy. You can nest as deeply as you need, and you can even mix ordered and unordered lists.

A common mistake is putting the nested list *between* the `<li>` items instead of *inside* one. Always place it inside the parent `<li>`, before its closing `</li>`.

## Description Lists

A **description list** pairs each item with a description, like a dictionary or a glossary of terms. It uses three elements:

- `<dl>` ("description list") wraps the whole list.
- `<dt>` ("description term") is the term being defined.
- `<dd>` ("description details") is the description of that term.

```html
<dl>
  <dt>HTML</dt>
  <dd>The language that structures content on the web.</dd>

  <dt>CSS</dt>
  <dd>The language that styles and lays out web pages.</dd>
</dl>
```

On the page, each `<dt>` term appears, with its `<dd>` description indented below it. You can have multiple `<dd>` elements for one `<dt>` if a term has several descriptions.

## When to Use Each List

Choosing the right list makes your meaning clear:

- Use a **`<ul>`** when order does not matter (features, ingredients, tags).
- Use an **`<ol>`** when order matters (steps, rankings, instructions).
- Use a **`<dl>`** for name-and-value pairs (terms and definitions, labels and details).

Pick a list for its **meaning**, not just for the bullets or numbers. If you only want indentation or spacing, use CSS later instead.

## Lists as Navigation Menus

Here is a powerful idea: the menus at the top of almost every website are really just lists of links. A navigation menu is an unordered list because the order of menu items is flexible.

```html
<nav>
  <ul>
    <li><a href="index.html">Home</a></li>
    <li><a href="about.html">About</a></li>
    <li><a href="services.html">Services</a></li>
    <li><a href="contact.html">Contact</a></li>
  </ul>
</nav>
```

Right now this shows as a bulleted, vertical list of links. Later, with CSS, you will remove the bullets and lay the items out side by side into a horizontal menu bar. Building menus from lists keeps them organized and accessible, because screen readers can announce "a list of four items."

## Common Mistakes

- Putting text or other tags directly inside `<ul>` or `<ol>` instead of inside an `<li>`.
- Typing your own numbers in an ordered list instead of letting the browser number them.
- Placing a nested list outside the parent `<li>` rather than inside it.
- Using a list just to get indentation, when you really want CSS spacing.
- Mixing up `<dt>` (the term) and `<dd>` (the description) in a description list.

## Exercises

1. Make an unordered list of your five favorite foods.
2. Write an ordered list of the steps to make a cup of tea.
3. Create a nested list: a list of two countries, each with a sub-list of two cities.
4. Build a description list defining three words from this course.
5. Build a navigation menu using a `<nav>` with a `<ul>` of four links.

## Key Takeaways

- `<ul>` makes an unordered (bulleted) list; order does not matter.
- `<ol>` makes an ordered (numbered) list; order matters.
- Every list item goes inside its own `<li>`.
- `type` and `start` change the markers and starting number of an ordered list.
- Nest a list *inside* an `<li>` to create sub-items.
- `<dl>`, `<dt>`, and `<dd>` make a description list of terms and details.
- Navigation menus are usually built from an unordered list of links.

---

[← Back to Course Index](../README.md) | [Next: Tables →](06-tables.md)
