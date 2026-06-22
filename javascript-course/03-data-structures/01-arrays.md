# Arrays

An **array** is an ordered list of things. Think of a shopping list: milk, bread,
eggs. The items have an order (milk is first), and you can have many of them in
one place. In JavaScript, an array lets you store many values in a single
variable.

This lesson teaches you the basics of arrays. We save the powerful transform
methods (like `map` and `filter`) for the next lesson.

## What You'll Learn

- What an array is and why it is useful
- How to create arrays
- How to read items using their position (index)
- How to find how many items an array has (`length`)
- How to add and remove items (`push`, `pop`, `shift`, `unshift`)
- How to change items
- How to check if something is an array (`Array.isArray`)
- Arrays inside arrays (nested arrays)
- How to loop over an array (`for`, `for...of`, `forEach`)

## What Is an Array?

An array is a list. The list has an **order**. Each item sits in a numbered
"slot". You can put numbers, text (strings), or even other arrays inside.

```js
const fruits = ["apple", "banana", "cherry"];
console.log(fruits); // => ["apple", "banana", "cherry"]
```

Without arrays, you would need a separate variable for each fruit. With one
array, you keep them all together.

## Creating Arrays

The normal way is to use **square brackets** `[ ]`. Put your items inside,
separated by commas.

```js
const numbers = [10, 20, 30];      // an array of numbers
const names = ["Sam", "Alex"];     // an array of strings
const mixed = [1, "two", true];    // you can mix types
const empty = [];                  // an empty array (no items yet)

console.log(numbers); // => [10, 20, 30]
console.log(empty);   // => []
```

You will mostly use `[ ]`. There is also `new Array()`, but it is rare and a bit
confusing, so we will skip it.

## Indexing (0-Based)

Each item has a position called an **index**. The important rule: counting
starts at **0**, not 1. So the first item is at index `0`.

```js
const fruits = ["apple", "banana", "cherry"];

console.log(fruits[0]); // => "apple"   (first item)
console.log(fruits[1]); // => "banana"  (second item)
console.log(fruits[2]); // => "cherry"  (third item)
```

If you ask for an index that does not exist, you get `undefined` (which means
"nothing is there").

```js
console.log(fruits[5]); // => undefined
```

A common trick: to get the **last** item, use `length - 1` (we explain `length`
next).

```js
console.log(fruits[fruits.length - 1]); // => "cherry"
```

## Length

`length` tells you how many items are in the array.

```js
const fruits = ["apple", "banana", "cherry"];
console.log(fruits.length); // => 3

const empty = [];
console.log(empty.length); // => 0
```

Note: `length` counts the items, but indexes start at 0. So if `length` is `3`,
the valid indexes are `0`, `1`, and `2`.

## Adding and Removing Items

There are four common methods. Two work at the **end** of the array, and two
work at the **start**.

### push — add to the end

```js
const fruits = ["apple", "banana"];
fruits.push("cherry");
console.log(fruits); // => ["apple", "banana", "cherry"]
```

### pop — remove from the end

```js
const fruits = ["apple", "banana", "cherry"];
const removed = fruits.pop();
console.log(removed); // => "cherry"  (pop gives back the removed item)
console.log(fruits);  // => ["apple", "banana"]
```

### unshift — add to the start

```js
const fruits = ["banana", "cherry"];
fruits.unshift("apple");
console.log(fruits); // => ["apple", "banana", "cherry"]
```

### shift — remove from the start

```js
const fruits = ["apple", "banana", "cherry"];
const removed = fruits.shift();
console.log(removed); // => "apple"
console.log(fruits);  // => ["banana", "cherry"]
```

A simple way to remember:
- **push / pop** work at the back (the "p" twins).
- **shift / unshift** work at the front.

## Accessing and Changing Items

You read an item with its index. You can also **change** an item by assigning a
new value to that index.

```js
const fruits = ["apple", "banana", "cherry"];

fruits[1] = "blueberry"; // change the item at index 1
console.log(fruits); // => ["apple", "blueberry", "cherry"]
```

Notice we used `const` but still changed the array. That is allowed! `const`
means you cannot point the variable to a brand-new array, but you can still
change what is **inside** the array.

## Checking with Array.isArray

Sometimes you need to know: "Is this value an array?" Use `Array.isArray()`. It
gives back `true` or `false`.

```js
console.log(Array.isArray([1, 2, 3])); // => true
console.log(Array.isArray("hello"));   // => false
console.log(Array.isArray(42));        // => false
```

Why not use `typeof`? Because `typeof [1, 2, 3]` gives `"object"`, which is not
helpful. `Array.isArray` is the clear, correct way.

## Nested Arrays

An array can hold other arrays. This is called a **nested array**. It is useful
for grids, tables, or rows of data.

```js
const grid = [
  [1, 2, 3],
  [4, 5, 6],
  [7, 8, 9],
];

console.log(grid[0]);    // => [1, 2, 3]  (the first row)
console.log(grid[0][2]); // => 3          (first row, third item)
console.log(grid[2][0]); // => 7          (third row, first item)
```

Read `grid[0][2]` from left to right: first pick row `0`, then pick item `2`
inside that row.

## Looping Over Arrays

Looping means doing something for each item in the array. There are three common
ways.

### The classic for loop

This gives you full control using the index. You start at `0` and go while the
index is less than `length`.

```js
const fruits = ["apple", "banana", "cherry"];

for (let i = 0; i < fruits.length; i++) {
  console.log(i, fruits[i]);
}
// => 0 "apple"
// => 1 "banana"
// => 2 "cherry"
```

### for...of

This is cleaner when you only need the items (not the index). It hands you each
item one by one.

```js
const fruits = ["apple", "banana", "cherry"];

for (const fruit of fruits) {
  console.log(fruit);
}
// => "apple"
// => "banana"
// => "cherry"
```

### forEach

`forEach` runs a function for each item. The function can receive the item and
its index.

```js
const fruits = ["apple", "banana", "cherry"];

fruits.forEach(function (fruit, index) {
  console.log(index, fruit);
});
// => 0 "apple"
// => 1 "banana"
// => 2 "cherry"
```

You can write the same thing shorter with an **arrow function**:

```js
fruits.forEach((fruit, index) => console.log(index, fruit));
```

Use `for...of` or `forEach` for most everyday loops. Use the classic `for` loop
when you need careful control over the index.

## Common Mistakes

- **Forgetting indexes start at 0.** The first item is `arr[0]`, not `arr[1]`.
- **Going past the end.** `arr[arr.length]` is `undefined`. The last item is
  `arr[arr.length - 1]`.
- **Mixing up push and unshift.** `push` adds to the end; `unshift` adds to the
  start.
- **Expecting pop/shift to return the new array.** They return the **removed**
  item, not the changed array.
- **Using `typeof` to check for an array.** It returns `"object"`. Use
  `Array.isArray()` instead.

## Exercises

1. Create an array called `colors` with three colors. Print the first and last
   items using indexes. Then print how many colors there are.
2. Start with `const stack = []`. Use `push` three times to add the numbers 1, 2,
   and 3. Then use `pop` once and print the array. What is left?
3. Make a nested array (a 2x2 grid) of any four numbers. Print the number in the
   second row, first column.
4. Loop over an array of names and print each name in a sentence, like
   `"Hello, Sam!"`. Do it once with `for...of` and once with `forEach`.

## Key Takeaways

- An **array** is an ordered list of values, written with `[ ]`.
- Indexes start at **0**. The first item is `arr[0]`.
- `length` tells you how many items there are.
- `push`/`pop` work at the end; `unshift`/`shift` work at the start.
- Change an item by assigning to its index: `arr[1] = "new"`.
- Use `Array.isArray()` to check if a value is an array.
- Arrays can hold other arrays (nested arrays) for grids and tables.
- Loop with `for`, `for...of`, or `forEach`.

---

[← Back to Course Index](../README.md) | [Next: Array Methods: map, filter, reduce →](02-array-methods.md)
