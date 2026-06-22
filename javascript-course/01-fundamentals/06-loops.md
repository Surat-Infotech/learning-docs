# Loops: `for`, `while`, `for...of`

Imagine you need to print "Hello" 100 times. Writing 100 lines would be painful.
Loops solve this. A **loop** repeats a block of code as many times as you need.
Instead of writing the same thing over and over, you write it once and tell the
loop how many times to run it.

## What You'll Learn

- Why loops exist and what problem they solve
- The `for` loop (init, condition, update)
- The `while` loop and the `do...while` loop
- `for...of` to go over arrays
- `for...in` to go over object keys, and how it differs from `for...of`
- `break` and `continue`
- The danger of infinite loops
- Nested loops, briefly
- A classic **FizzBuzz** example

## Why Loops Exist

Computers are great at repeating tasks. A loop lets you do something repeatedly
without copying code. The repeated run is called an **iteration** (one trip
through the loop).

```js
// Without a loop (tedious and rigid):
console.log("Hello");
console.log("Hello");
console.log("Hello");

// With a loop (clean and flexible):
for (let i = 0; i < 3; i++) {
  console.log("Hello");
}
// => Hello
// => Hello
// => Hello
```

## The `for` Loop

The `for` loop is the most common loop. It has three parts inside its
parentheses, separated by semicolons:

1. **Init** — runs once at the start (usually to create a counter).
2. **Condition** — checked before each iteration; the loop runs while it is true.
3. **Update** — runs after each iteration (usually to change the counter).

```js
for (let i = 1; i <= 5; i++) {
  console.log(i);
}
// => 1
// => 2
// => 3
// => 4
// => 5
```

Reading this in plain English: "Start `i` at 1. While `i` is 5 or less, print
`i`, then add 1 to `i`."

Here `i++` means "add 1 to `i`" (a short form of `i = i + 1`). The name `i` is a
common choice for a counter, short for "index."

You can loop over an array using its index positions:

```js
let fruits = ["apple", "banana", "cherry"];

for (let i = 0; i < fruits.length; i++) {
  console.log(fruits[i]);
}
// => apple
// => banana
// => cherry
```

Notice we start at `0` because array positions start at `0`, and we stop at
`fruits.length` (which is `3` here, so the last valid index is `2`).

## The `while` Loop

A `while` loop repeats **as long as a condition is true**. Use it when you do not
know in advance how many times you need to loop.

```js
let count = 1;

while (count <= 3) {
  console.log(count);
  count++; // VERY important: change the counter, or the loop never ends
}
// => 1
// => 2
// => 3
```

You must change something inside the loop so the condition eventually becomes
`false`. Otherwise the loop runs forever (see the infinite loop warning below).

## The `do...while` Loop

A `do...while` loop is like `while`, but it checks the condition **after** running
the block. This means the block always runs **at least once**, even if the
condition is false from the start.

```js
let number = 10;

do {
  console.log("This runs at least once.");
} while (number < 5);
// => This runs at least once.
```

Use `do...while` when you need to run the body once before checking, like asking a
user for input and then deciding whether to ask again.

## `for...of` — Looping Over Arrays

`for...of` is a cleaner way to go through the **values** of an array. You do not
need a counter or an index — it hands you each item directly.

```js
let colors = ["red", "green", "blue"];

for (let color of colors) {
  console.log(color);
}
// => red
// => green
// => blue
```

Compare this to the classic `for` loop: `for...of` is shorter and harder to get
wrong because there is no index math.

## `for...in` — Looping Over Object Keys

`for...in` goes through the **keys** (the property names) of an object.

```js
let person = {
  name: "Ana",
  age: 28,
  city: "Lisbon",
};

for (let key in person) {
  console.log(key + ": " + person[key]);
}
// => name: Ana
// => age: 28
// => city: Lisbon
```

Here `key` is each property name, and `person[key]` gets the value for that key.

### The Difference: `for...of` vs `for...in`

This trips up many beginners, so remember it clearly:

- **`for...of`** gives you the **values**. Use it for **arrays**.
- **`for...in`** gives you the **keys**. Use it for **objects**.

```js
let arr = ["a", "b", "c"];

for (let item of arr) {
  console.log(item); // => a, b, c   (the values)
}

for (let index in arr) {
  console.log(index); // => 0, 1, 2   (the keys/positions, as text)
}
```

Short memory tip: **of** = the actual values; **in** = the keys/indexes.

## `break` and `continue`

These two keywords give you extra control inside a loop.

- **`break`** stops the loop completely and exits right away.
- **`continue`** skips the rest of the current iteration and jumps to the next one.

```js
// break: stop when we find the number 3
for (let i = 1; i <= 5; i++) {
  if (i === 3) {
    break;
  }
  console.log(i);
}
// => 1
// => 2
```

```js
// continue: skip even numbers, print the rest
for (let i = 1; i <= 5; i++) {
  if (i % 2 === 0) {
    continue; // skip the console.log for even numbers
  }
  console.log(i);
}
// => 1
// => 3
// => 5
```

## Warning: Infinite Loops

An **infinite loop** is a loop whose condition never becomes false. It runs
forever, freezing your program or browser tab. This usually happens when you
forget to update the counter.

```js
// DO NOT RUN THIS — it never ends!
let i = 0;
while (i < 5) {
  console.log(i);
  // bug: we never do i++, so i stays 0 and i < 5 is always true
}
```

Always make sure the loop moves toward its stopping condition. If a loop seems
stuck, check that the counter is changing.

## Nested Loops (Briefly)

You can put a loop inside another loop. The inner loop runs completely for **each**
step of the outer loop. This is useful for grids or tables.

```js
for (let row = 1; row <= 2; row++) {
  for (let col = 1; col <= 3; col++) {
    console.log(`row ${row}, col ${col}`);
  }
}
// => row 1, col 1
// => row 1, col 2
// => row 1, col 3
// => row 2, col 1
// => row 2, col 2
// => row 2, col 3
```

Nested loops can get slow if both run many times, so use them thoughtfully.

## Example: FizzBuzz

FizzBuzz is a classic practice problem. The rules: count from 1 to 15. For
multiples of 3 print `"Fizz"`, for multiples of 5 print `"Buzz"`, for multiples of
both print `"FizzBuzz"`, otherwise print the number. It combines loops,
conditionals, and the `%` (remainder) operator.

```js
for (let i = 1; i <= 15; i++) {
  if (i % 3 === 0 && i % 5 === 0) {
    console.log("FizzBuzz");
  } else if (i % 3 === 0) {
    console.log("Fizz");
  } else if (i % 5 === 0) {
    console.log("Buzz");
  } else {
    console.log(i);
  }
}
// => 1
// => 2
// => Fizz
// => 4
// => Buzz
// => Fizz
// => 7
// => 8
// => Fizz
// => Buzz
// => 11
// => Fizz
// => 13
// => 14
// => FizzBuzz
```

Notice the order: we check "both" first. If we checked `i % 3` first, `15` would
print `"Fizz"` and never reach the `"FizzBuzz"` case.

## Common Mistakes

- **Forgetting to update the counter**, causing an infinite loop.
- **Off-by-one errors.** Using `<=` vs `<` changes how many times the loop runs.
  For arrays, loop while `i < array.length`.
- **Mixing up `for...of` and `for...in`.** Use `for...of` for array values and
  `for...in` for object keys.
- **Putting the FizzBuzz checks in the wrong order.** Check the combined condition
  before the single ones.
- **Confusing `break` and `continue`.** `break` exits the loop; `continue` skips
  to the next iteration.

## Exercises

1. Use a `for` loop to print the numbers from 10 down to 1 (counting backwards).
2. Use a `for...of` loop to print each item in an array of your three favorite
   foods.
3. Make an object with three key/value pairs, then use `for...in` to print each
   key and its value.
4. Write a loop that prints numbers 1 to 20 but uses `continue` to skip every
   multiple of 4.

## Key Takeaways

- Loops repeat code so you do not write the same thing many times.
- A `for` loop has init, condition, and update parts.
- `while` repeats while a condition is true; `do...while` always runs at least
  once.
- `for...of` loops over array values; `for...in` loops over object keys.
- `break` exits a loop; `continue` skips to the next iteration.
- Always move toward the stopping condition to avoid infinite loops.
- FizzBuzz combines loops, conditionals, and the `%` operator.

---

[← Back to Course Index](../README.md)
