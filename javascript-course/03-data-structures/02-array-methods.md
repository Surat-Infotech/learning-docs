# Array Methods: map, filter, reduce

Arrays come with many built-in tools called **methods**. A method is a function
that belongs to the array. You call it with a dot, like `numbers.map(...)`.

This is one of the most important lessons in the whole course. These methods let
you transform lists of data cleanly, without writing long loops. Take your time,
type every example, and read the output comments.

## What You'll Learn

- The big three: `map`, `filter`, and `reduce`
- Helpers: `forEach`, `find`, `findIndex`, `some`, `every`, `includes`,
  `indexOf`
- Reordering: `sort`
- Copying parts: `slice` vs `splice`
- Joining: `concat`, `join`
- **Method chaining** (using several methods in a row)
- **Immutability**: why `map` and `filter` give you a new array

## A Quick Word on Callbacks

Most of these methods take a **callback**. A callback is just a function you hand
to the method. The method runs your function on each item.

```js
const numbers = [1, 2, 3];

// the function inside is the callback
numbers.forEach((n) => console.log(n));
// => 1
// => 2
// => 3
```

We will use short **arrow functions** for callbacks. If `n => n * 2` looks new,
it means "take `n`, give back `n * 2`".

## map — Transform Each Item

`map` makes a **new array**. It runs your function on every item and collects the
results. The new array has the **same number of items**.

Think of a factory line: each item goes in, a changed item comes out.

```js
const numbers = [1, 2, 3, 4];

const doubled = numbers.map((n) => n * 2);
console.log(doubled); // => [2, 4, 6, 8]
console.log(numbers); // => [1, 2, 3, 4]  (original is unchanged!)
```

Another example: turn names into greetings.

```js
const names = ["Sam", "Alex", "Jo"];

const greetings = names.map((name) => `Hello, ${name}!`);
console.log(greetings);
// => ["Hello, Sam!", "Hello, Alex!", "Hello, Jo!"]
```

The callback can also receive the **index** as a second argument.

```js
const letters = ["a", "b", "c"];

const labeled = letters.map((letter, index) => `${index}: ${letter}`);
console.log(labeled); // => ["0: a", "1: b", "2: c"]
```

## filter — Keep Some Items

`filter` makes a **new array** with only the items that pass a test. Your
function must return `true` (keep it) or `false` (drop it). The new array can be
**smaller**.

Think of a sieve: only the items that fit fall through.

```js
const numbers = [1, 2, 3, 4, 5, 6];

const evens = numbers.filter((n) => n % 2 === 0);
console.log(evens); // => [2, 4, 6]
```

(`n % 2 === 0` is `true` when `n` divides evenly by 2, meaning it is even.)

Another example: keep only the long words.

```js
const words = ["hi", "hello", "hey", "welcome"];

const longWords = words.filter((word) => word.length > 3);
console.log(longWords); // => ["hello", "welcome"]
```

**Key difference:** `map` always keeps the same count and changes each item.
`filter` keeps each item as-is but may remove some.

## reduce — Combine Into One Value

`reduce` boils a whole array down to **one value**. That value could be a number,
a string, or even another array or object.

`reduce` takes two things:
1. A function `(accumulator, item) => newAccumulator`.
2. A starting value for the accumulator.

The **accumulator** is the running total that carries from step to step.

```js
const numbers = [1, 2, 3, 4];

const sum = numbers.reduce((total, n) => total + n, 0);
console.log(sum); // => 10
```

Here is what happens step by step (starting value is `0`):

```
start:  total = 0
n = 1:  total = 0 + 1 = 1
n = 2:  total = 1 + 2 = 3
n = 3:  total = 3 + 3 = 6
n = 4:  total = 6 + 4 = 10
result: 10
```

`reduce` is flexible. You can find the biggest number:

```js
const numbers = [4, 9, 2, 7];

const max = numbers.reduce((biggest, n) => (n > biggest ? n : biggest), 0);
console.log(max); // => 9
```

Or join words into a sentence (though `join` is easier for that, shown later):

```js
const words = ["I", "love", "JavaScript"];

const sentence = words.reduce((text, word) => text + " " + word);
console.log(sentence); // => " I love JavaScript"
```

> Tip: Always give `reduce` a starting value (the second argument). It makes the
> behavior clear and avoids surprises on empty arrays.

## forEach — Do Something for Each Item

`forEach` runs your function for each item but gives back **nothing**
(`undefined`). Use it for side effects like printing or saving, not for building
a new array.

```js
const fruits = ["apple", "banana"];

fruits.forEach((fruit) => console.log(`I like ${fruit}`));
// => "I like apple"
// => "I like banana"
```

If you want a new array, use `map`. If you just want to "do stuff", use
`forEach`.

## find and findIndex

`find` gives back the **first item** that passes a test, or `undefined` if none
do.

```js
const numbers = [5, 12, 8, 130];

const firstBig = numbers.find((n) => n > 10);
console.log(firstBig); // => 12
```

`findIndex` gives back the **index** of that first match, or `-1` if none match.

```js
const numbers = [5, 12, 8, 130];

const index = numbers.findIndex((n) => n > 10);
console.log(index); // => 1
```

## some and every

`some` returns `true` if **at least one** item passes the test.

```js
const numbers = [1, 2, 3];

console.log(numbers.some((n) => n > 2)); // => true  (3 passes)
console.log(numbers.some((n) => n > 5)); // => false (none pass)
```

`every` returns `true` only if **all** items pass the test.

```js
const numbers = [2, 4, 6];

console.log(numbers.every((n) => n % 2 === 0)); // => true  (all even)
console.log(numbers.every((n) => n > 3));        // => false (2 fails)
```

## includes, indexOf

`includes` checks if a value is in the array. It returns `true` or `false`.

```js
const fruits = ["apple", "banana"];

console.log(fruits.includes("banana")); // => true
console.log(fruits.includes("cherry")); // => false
```

`indexOf` returns the **index** of a value, or `-1` if it is not found.

```js
const fruits = ["apple", "banana", "cherry"];

console.log(fruits.indexOf("banana")); // => 1
console.log(fruits.indexOf("grape"));  // => -1
```

Use `includes` for a simple yes/no. Use `indexOf` when you need the position.

## sort — Reorder Items

`sort` reorders the array. **Careful:** by default it sorts as **text**, which
gives wrong results for numbers.

```js
const numbers = [10, 1, 5, 100];

numbers.sort();
console.log(numbers); // => [1, 10, 100, 5]  (wrong! sorted like words)
```

To sort numbers correctly, pass a compare function:

```js
const numbers = [10, 1, 5, 100];

numbers.sort((a, b) => a - b); // smallest to largest
console.log(numbers); // => [1, 5, 10, 100]

numbers.sort((a, b) => b - a); // largest to smallest
console.log(numbers); // => [100, 10, 5, 1]
```

The rule: if `a - b` is negative, `a` comes first. Sorting **text** works fine
by default (alphabetical order):

```js
const names = ["Sam", "Alex", "Jo"];
names.sort();
console.log(names); // => ["Alex", "Jo", "Sam"]
```

> Warning: `sort` changes (mutates) the original array. If you want to keep the
> original, copy it first (more on copying below).

## slice vs splice

These names look alike but do very different things.

### slice — copy a part (does NOT change the original)

`slice(start, end)` returns a **new array** with items from `start` up to (but
not including) `end`.

```js
const fruits = ["apple", "banana", "cherry", "date"];

const part = fruits.slice(1, 3);
console.log(part);   // => ["banana", "cherry"]
console.log(fruits); // => ["apple", "banana", "cherry", "date"] (unchanged)
```

A nice trick: `slice()` with no arguments makes a full copy of the array.

```js
const copy = fruits.slice();
```

### splice — cut/insert (DOES change the original)

`splice(start, count)` removes `count` items starting at `start`. It changes the
original array and returns the removed items.

```js
const fruits = ["apple", "banana", "cherry", "date"];

const removed = fruits.splice(1, 2);
console.log(removed); // => ["banana", "cherry"]
console.log(fruits);  // => ["apple", "date"]  (changed!)
```

You can also insert items with `splice`:

```js
const fruits = ["apple", "date"];
fruits.splice(1, 0, "banana", "cherry"); // remove 0, add 2 items at index 1
console.log(fruits); // => ["apple", "banana", "cherry", "date"]
```

**Remember:** `slice` is safe (copies). `splice` is destructive (edits in
place). The extra "p" in s**p**lice can remind you it changes things.

## concat — Join Arrays

`concat` joins arrays into a **new array**. It does not change the originals.

```js
const a = [1, 2];
const b = [3, 4];

const joined = a.concat(b);
console.log(joined); // => [1, 2, 3, 4]
console.log(a);      // => [1, 2]  (unchanged)
```

## join — Array to String

`join` turns an array into a single string. You choose the "glue" between items.

```js
const words = ["I", "love", "JavaScript"];

console.log(words.join(" "));  // => "I love JavaScript"
console.log(words.join("-"));  // => "I-love-JavaScript"
console.log(words.join(""));   // => "IloveJavaScript"
```

## Method Chaining

Because `map`, `filter`, and `slice` each return a **new array**, you can call
another method right after. Calling methods one after another is called
**chaining**.

```js
const numbers = [1, 2, 3, 4, 5, 6];

const result = numbers
  .filter((n) => n % 2 === 0) // keep evens: [2, 4, 6]
  .map((n) => n * 10)         // multiply:   [20, 40, 60]
  .reduce((sum, n) => sum + n, 0); // add up: 120

console.log(result); // => 120
```

Read a chain from top to bottom. Each line takes the result of the line above.
Chaining keeps code short and easy to read.

## The Immutability Idea

**Immutable** means "does not change". Methods like `map`, `filter`, `slice`,
and `concat` do **not** change the original array. They give you a **new** one.
This is good: your original data stays safe.

```js
const original = [1, 2, 3];
const changed = original.map((n) => n + 1);

console.log(original); // => [1, 2, 3]  (safe)
console.log(changed);  // => [2, 3, 4]  (new array)
```

A few methods **do** change the original: `push`, `pop`, `shift`, `unshift`,
`sort`, `splice`, and `reverse`. When in doubt, copy first with `slice()` or the
spread operator (`[...arr]`, covered in a later lesson).

## Common Mistakes

- **Using `map` when you mean `forEach`.** If you do not use the new array, use
  `forEach`.
- **Forgetting `map`/`filter` return a new array.** You must store the result:
  `const out = arr.map(...)`.
- **Sorting numbers without a compare function.** `sort()` alone sorts as text.
  Use `arr.sort((a, b) => a - b)`.
- **Mixing up `slice` and `splice`.** `slice` copies; `splice` edits the
  original.
- **Forgetting the starting value in `reduce`.** Always pass it, like the `0` in
  `reduce((t, n) => t + n, 0)`.
- **Filter callback not returning true/false.** The test must give a yes/no
  answer.

## Exercises

1. Given `const prices = [10, 25, 5, 40]`, use `map` to add 10% tax to each price
   (multiply by `1.1`). Print the new array.
2. Given `const ages = [12, 18, 7, 21, 16]`, use `filter` to keep only ages 18
   and over. Then use `length` to print how many adults there are.
3. Given `const cart = [3, 5, 2]`, use `reduce` to find the total. Then chain
   `map` and `reduce` on `[1, 2, 3, 4]` to get the sum of each number squared.
4. Given `const scores = [55, 90, 72, 88]`, use `find` to get the first score
   over 80, `some` to check if any score is below 60, and `every` to check if all
   scores are passing (60 or more).

## Key Takeaways

- **map** transforms each item and returns a new array of the same size.
- **filter** keeps items that pass a test and returns a new (maybe smaller)
  array.
- **reduce** combines all items into one value using an accumulator.
- **forEach** runs code for each item but returns nothing.
- **find/findIndex** get the first match (or its index); **some/every** ask
  about any/all items.
- **includes/indexOf** check for a value (yes-no / position).
- **sort** reorders in place; use `(a, b) => a - b` for numbers.
- **slice** copies safely; **splice** edits the original.
- **concat** joins arrays; **join** turns an array into a string.
- Chain methods together; prefer **immutable** methods that return new arrays.

---

[← Back to Course Index](../README.md) | [Next: Objects →](03-objects.md)
