# Tables

A table organizes information into rows and columns, like a spreadsheet. Tables are perfect for data that naturally fits a grid, such as a price list or a schedule. In this lesson you will learn how to build well-structured, accessible tables.

## What You'll Learn

- When you should and should not use a table
- The basic building blocks: `<table>`, `<tr>`, `<td>`, and `<th>`
- How to group rows with `<thead>`, `<tbody>`, and `<tfoot>`
- How `scope` makes headers clear to screen readers
- How to merge cells with `colspan` and `rowspan`
- How to add a title with `<caption>`
- How to keep your tables accessible

## When to Use a Table

Use a table for **tabular data** — information that belongs in rows and columns and has a relationship between them. Good examples:

- A price list (product, price, stock)
- A class schedule (day, time, subject)
- Sports standings (team, wins, losses)

Important rule: **do not use tables for page layout.** Long ago, people used tables to position things on a page, but that is now a mistake. It makes pages hard to read for screen readers and hard to adjust on phones. For layout, you will use CSS later. Tables are for *data* only.

## The Basic Building Blocks

A table is built from a few nested elements:

- `<table>` wraps the whole table.
- `<tr>` ("table row") is one horizontal row.
- `<td>` ("table data") is one cell of normal data.
- `<th>` ("table header") is a header cell, like a column title. It is bold and centered by default.

Here is a simple table:

```html
<table>
  <tr>
    <th>Name</th>
    <th>Age</th>
  </tr>
  <tr>
    <td>Alice</td>
    <td>30</td>
  </tr>
  <tr>
    <td>Bob</td>
    <td>25</td>
  </tr>
</table>
```

On the page, this shows a small grid:

```text
Name    Age
Alice   30
Bob     25
```

Read it row by row: each `<tr>` is a row, and inside each row the cells line up into columns. The first row uses `<th>` for the headers; the rest use `<td>` for data.

## Grouping Rows: thead, tbody, tfoot

For larger tables, you can group the rows into three sections. This makes the table easier to read, style, and understand:

- `<thead>` ("table head") holds the header row(s).
- `<tbody>` ("table body") holds the main data rows.
- `<tfoot>` ("table foot") holds a summary or total row.

```html
<table>
  <thead>
    <tr>
      <th>Product</th>
      <th>Price</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Apple</td>
      <td>$1</td>
    </tr>
    <tr>
      <td>Bread</td>
      <td>$2</td>
    </tr>
  </tbody>
  <tfoot>
    <tr>
      <td>Total</td>
      <td>$3</td>
    </tr>
  </tfoot>
</table>
```

These sections are optional for tiny tables but very helpful as tables grow.

## The `scope` Attribute

When a screen reader reads a table aloud, it needs to know whether a header belongs to a column or a row. The `scope` attribute tells it. This is a small detail that makes a big difference for accessibility.

- `scope="col"` means the header is for a whole **column**.
- `scope="row"` means the header is for a whole **row**.

```html
<table>
  <thead>
    <tr>
      <th scope="col">Day</th>
      <th scope="col">Subject</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th scope="row">Monday</th>
      <td>Math</td>
    </tr>
    <tr>
      <th scope="row">Tuesday</th>
      <td>Science</td>
    </tr>
  </tbody>
</table>
```

Here the days are row headers and the column titles are column headers. A screen reader can now say "Monday, Subject: Math," which makes perfect sense.

## Merging Cells: colspan and rowspan

Sometimes one cell needs to stretch across several columns or rows.

- `colspan` makes a cell span across multiple **columns** (horizontally).
- `rowspan` makes a cell span across multiple **rows** (vertically).

Here a header spans two columns:

```html
<table>
  <tr>
    <th colspan="2">Contact Information</th>
  </tr>
  <tr>
    <td>Email</td>
    <td>hello@example.com</td>
  </tr>
</table>
```

The "Contact Information" cell stretches across the full width of the two columns below it.

Here a cell spans two rows:

```html
<table>
  <tr>
    <td rowspan="2">Fruits</td>
    <td>Apple</td>
  </tr>
  <tr>
    <td>Banana</td>
  </tr>
</table>
```

The "Fruits" cell covers two rows, sitting next to both "Apple" and "Banana." When a cell spans multiple rows or columns, remember to remove the cells it now covers, or the table will become uneven.

## Adding a Caption

A `<caption>` gives the table a title. It goes right after the opening `<table>` tag and shows above the table. Captions help everyone, especially screen reader users, understand what the table is about.

```html
<table>
  <caption>Weekly Class Schedule</caption>
  <tr>
    <th scope="col">Day</th>
    <th scope="col">Subject</th>
  </tr>
  <tr>
    <td>Monday</td>
    <td>Math</td>
  </tr>
</table>
```

## A Full Worked Example

Let's put everything together into one complete, accessible table: a small price list with a header, a body, a total row, scoped headers, and a caption.

```html
<table>
  <caption>Bakery Price List</caption>
  <thead>
    <tr>
      <th scope="col">Item</th>
      <th scope="col">Size</th>
      <th scope="col">Price</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th scope="row">Bread</th>
      <td>Large</td>
      <td>$3</td>
    </tr>
    <tr>
      <th scope="row">Muffin</th>
      <td>Small</td>
      <td>$2</td>
    </tr>
    <tr>
      <th scope="row">Cake</th>
      <td>Large</td>
      <td>$10</td>
    </tr>
  </tbody>
  <tfoot>
    <tr>
      <th scope="row" colspan="2">Total</th>
      <td>$15</td>
    </tr>
  </tfoot>
</table>
```

Notice how every part works together: the caption titles the table, the `<thead>` holds column headers, each row's first cell is a row header, the `<tfoot>` holds the total, and `colspan="2"` lets the "Total" label stretch across two columns.

## Table Accessibility

A few simple habits make your tables work well for everyone:

- Use `<th>` for header cells, not plain `<td>`.
- Add `scope="col"` or `scope="row"` to every header.
- Give each table a `<caption>` describing its purpose.
- Keep tables simple; very complex tables are hard for everyone to read.
- Never use a table just to arrange a page layout.

## Common Mistakes

- Using a table for page layout instead of for data.
- Using `<td>` for headers instead of `<th>`.
- Forgetting the `scope` attribute on header cells.
- Leaving uneven rows after using `colspan` or `rowspan` (too many or too few cells).
- Skipping the `<caption>`, so the table's purpose is unclear.
- Forgetting to wrap each row's cells in a `<tr>`.

## Exercises

1. Build a simple table with three columns and three rows of data.
2. Add a `<caption>` and split your table into `<thead>` and `<tbody>`.
3. Add `scope="col"` to your column headers and `scope="row"` to your row headers.
4. Use `colspan` to make a header that spans two columns.
5. Build the full bakery price list example and add a `<tfoot>` total row.

## Key Takeaways

- Use tables for **tabular data**, never for page layout.
- `<table>`, `<tr>`, `<td>`, and `<th>` are the core building blocks.
- `<thead>`, `<tbody>`, and `<tfoot>` group rows into meaningful sections.
- `scope="col"` and `scope="row"` make headers clear to screen readers.
- `colspan` and `rowspan` merge cells across columns and rows.
- A `<caption>` gives the table a clear title.
- Good headers, scopes, and captions make tables accessible to everyone.

---

[← Back to Course Index](../README.md) | [Next: Forms and Inputs →](../02-forms-and-semantics/01-forms-and-inputs.md)
