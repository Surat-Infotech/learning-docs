# CSS Grid

CSS Grid is the most powerful layout tool in CSS. While Flexbox arranges items in a single row or column, Grid arranges them in **rows and columns at the same time**, like a spreadsheet or a checkerboard. In this lesson you will learn how to build real two-dimensional layouts with surprisingly little code.

## What You'll Learn

- What problem Grid solves (two-dimensional layouts)
- How to turn on Grid with `display: grid`
- Defining columns and rows with `grid-template-columns` and `grid-template-rows`
- The flexible `fr` unit and the handy `repeat()` function
- Adding space with `gap`, and flexible sizing with `minmax()`
- Building responsive grids with `auto-fit` and `auto-fill`, no media queries needed
- Placing and spanning items with `grid-column` and `grid-row`
- Naming layout areas with `grid-template-areas`
- When to choose Grid versus Flexbox

## What Grid Solves

Grid is built for **two-dimensional layouts**: when you need control over both rows and columns together. Think of a photo gallery, a dashboard, or a whole page layout with a header, sidebar, content, and footer.

The element you apply `display: grid` to is the **grid container**. Its direct children become **grid items**, and they automatically slot into the grid you define.

```css
.gallery {
  display: grid;
}
```

On its own, `display: grid` does not look like much yet. The power comes from defining the columns and rows.

## Defining Columns with `grid-template-columns`

`grid-template-columns` describes the columns of your grid. You list a size for each column.

```css
.gallery {
  display: grid;
  grid-template-columns: 100px 100px 100px; /* three 100px columns */
}
```

```html
<div class="gallery">
  <div>1</div>
  <div>2</div>
  <div>3</div>
  <div>4</div>
  <div>5</div>
  <div>6</div>
</div>
```

This makes three columns. The six items fill them row by row: items 1, 2, 3 on the first row, then 4, 5, 6 on the second. Grid automatically created the second row for us.

## The `fr` Unit

Fixed pixel columns are rigid. The **`fr` unit** ("fraction") tells columns to share the available space flexibly. Think of `fr` as "parts" of the leftover space.

```css
.layout {
  display: grid;
  grid-template-columns: 1fr 1fr 1fr; /* three equal columns */
}
```

Each column takes one fraction, so all three are equal and they stretch to fill the container. You can mix proportions:

```css
.layout {
  display: grid;
  grid-template-columns: 2fr 1fr; /* left column twice as wide as the right */
}
```

You can even mix `fr` with fixed sizes. Fixed columns take their space first, and the `fr` columns share whatever is left:

```css
.layout {
  display: grid;
  grid-template-columns: 250px 1fr; /* fixed sidebar, flexible content */
}
```

## `repeat()`

Writing `1fr 1fr 1fr 1fr` gets tedious. The **`repeat()`** function does it for you. It takes a count and a size.

```css
.gallery {
  display: grid;
  grid-template-columns: repeat(4, 1fr); /* same as 1fr 1fr 1fr 1fr */
}
```

You can repeat patterns too:

```css
grid-template-columns: repeat(3, 200px 100px); /* 200 100 200 100 200 100 */
```

## `gap`: Space Between Cells

Just like in Flexbox, `gap` adds even spacing between grid cells without margins.

```css
.gallery {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 16px; /* space between every row and column */
}
```

You can set row and column gaps separately:

```css
gap: 24px 12px; /* row-gap column-gap */
```

## Defining Rows with `grid-template-rows`

By default, Grid creates rows automatically as needed, sized to fit their content. If you want specific row heights, use `grid-template-rows`.

```css
.layout {
  display: grid;
  grid-template-columns: 1fr 1fr;
  grid-template-rows: 100px 200px; /* first row 100px, second 200px */
}
```

## `minmax()`: Flexible But Bounded

The **`minmax()`** function sets a minimum and a maximum size for a track (a row or column). It means "never smaller than this, never larger than that."

```css
.layout {
  display: grid;
  grid-template-columns: minmax(200px, 1fr) 2fr;
}
```

Here the first column is at least 200px but can grow as a `1fr` share. This keeps columns from getting uncomfortably narrow.

## Responsive Grids with `auto-fit` and `auto-fill`

Here is one of Grid's most impressive features: a responsive grid that reflows on its own, **without any media queries**. The trick combines `repeat()`, `auto-fit`, and `minmax()`.

```css
.cards {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 16px;
}
```

Read this as: "Make as many columns as fit, where each column is at least 200px wide but grows to fill the space." On a wide screen you get many columns; on a phone you get one. The grid figures it out.

```html
<div class="cards">
  <div class="card">A</div>
  <div class="card">B</div>
  <div class="card">C</div>
  <div class="card">D</div>
</div>
```

### `auto-fit` vs `auto-fill`

These two keywords look identical until the row is not full:

- **`auto-fill`** keeps empty columns reserved, so existing items do not stretch to fill the leftover space.
- **`auto-fit`** collapses empty columns, so existing items expand to fill the whole row.

For most card layouts you want `auto-fit`, so items grow nicely. Use `auto-fill` when you want a consistent column width even with few items.

## Placing Items: `grid-column` and `grid-row`

So far items have filled the grid automatically. You can also place an item exactly, and make it **span** multiple columns or rows.

Grid lines are numbered starting at 1. A four-column grid has five vertical lines: 1, 2, 3, 4, 5.

```css
.gallery {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 10px;
}

.feature {
  grid-column: 1 / 3; /* start at line 1, end at line 3 (spans 2 columns) */
}
```

```html
<div class="gallery">
  <div class="feature">Big featured item</div>
  <div>B</div>
  <div>C</div>
  <div>D</div>
</div>
```

You can also use the `span` keyword, which is often easier to read:

```css
.feature {
  grid-column: span 2; /* take up 2 columns, wherever it lands */
}

.tall {
  grid-row: span 2; /* take up 2 rows */
}
```

## Named Areas with `grid-template-areas`

For whole-page layouts, `grid-template-areas` lets you draw your layout as text. You give each region a name, then sketch where it goes. This is wonderfully readable.

```css
.page {
  display: grid;
  grid-template-columns: 200px 1fr;
  grid-template-rows: auto 1fr auto;
  grid-template-areas:
    "header  header"
    "sidebar content"
    "footer  footer";
  gap: 12px;
  min-height: 100vh;
}

.page-header  { grid-area: header; }
.page-sidebar { grid-area: sidebar; }
.page-content { grid-area: content; }
.page-footer  { grid-area: footer; }
```

```html
<div class="page">
  <header class="page-header">Header</header>
  <aside class="page-sidebar">Sidebar</aside>
  <main class="page-content">Content</main>
  <footer class="page-footer">Footer</footer>
</div>
```

The strings in `grid-template-areas` are a picture of your layout. The header spans both columns, the sidebar and content share the middle row, and the footer spans the bottom. Repeating a name (like `header header`) makes that area stretch across those cells.

## `justify-items` and `align-items`

These properties control how items sit **inside their own cells**.

- `justify-items` aligns items horizontally within each cell (`start`, `end`, `center`, `stretch`).
- `align-items` aligns items vertically within each cell (`start`, `end`, `center`, `stretch`).

```css
.grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  justify-items: center; /* center each item horizontally in its cell */
  align-items: center;   /* center each item vertically in its cell */
}
```

By default both are `stretch`, so items fill their cells. To center a single item differently, use `justify-self` and `align-self` on that item.

## Grid vs Flexbox: Which to Choose?

Both tools are great, and you will often use them together. Here is the simple guidance:

- Use **Flexbox** for **one-dimensional** layouts: a row of buttons, a nav bar, items in a line.
- Use **Grid** for **two-dimensional** layouts: galleries, dashboards, and full page structures with rows and columns.

A common real-world pattern is Grid for the overall page and Flexbox inside individual components (like aligning a card's title and button). They are partners, not rivals.

## Common Mistakes

- Forgetting that `grid-template-columns` defines the columns. Without it you just get one column.
- Confusing grid line numbers (the lines between cells) with the cells themselves. A 3-column grid has 4 vertical lines.
- Mixing up `auto-fit` and `auto-fill`. Use `auto-fit` when you want items to stretch and fill the row.
- Misaligning the strings in `grid-template-areas`. Each row must have the same number of names, and names that span must touch.
- Reaching for Grid when a simple row of items would be cleaner with Flexbox.

## Exercises

1. Build a 3-column gallery with `grid-template-columns: repeat(3, 1fr)` and a `16px` gap. Add six boxes and watch them fill two rows.
2. Create a responsive card grid using `repeat(auto-fit, minmax(220px, 1fr))`. Resize the browser and watch the column count change with no media queries.
3. Make a 4-column grid and give one item `grid-column: span 2` so it takes up two columns.
4. Build a full-page layout (header, sidebar, content, footer) using `grid-template-areas`. Then swap the sidebar to the right side by editing only the area strings.
5. Take a grid and set `justify-items: center` and `align-items: center` to center every item inside its cell.

## Key Takeaways

- Grid handles **two-dimensional** layouts: rows and columns together.
- `display: grid` plus `grid-template-columns` defines the column structure.
- The `fr` unit shares available space; `repeat()` avoids repetition.
- `gap` spaces cells; `minmax()` sets flexible bounds for tracks.
- `repeat(auto-fit, minmax(...))` builds responsive grids without media queries.
- `grid-column` and `grid-row` (with `span`) place and stretch items.
- `grid-template-areas` draws whole-page layouts as readable text.
- Use Grid for 2-D layouts and Flexbox for 1-D; they work well together.

---

[← Back to Course Index](../README.md) | [Next: Responsive Design and Media Queries →](05-responsive-design-and-media-queries.md)
