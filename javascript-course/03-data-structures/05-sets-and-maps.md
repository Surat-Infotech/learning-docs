# Sets and Maps

You already know arrays and objects. JavaScript has two more handy collections:
**Set** and **Map**. They were added to solve common problems more cleanly.

- A **Set** is a bag of values where every value is **unique** (no duplicates).
- A **Map** is like an object, but its keys can be **anything**, and it remembers
  the order you added things.

You will not need these every day, but in the right spot they make your code
shorter and clearer.

## What You'll Learn

- What a Set is and how to use `add`, `has`, `delete`, `size`
- Removing duplicates by converting an array to a Set and back
- Looping over a Set
- What a Map is and how to use `set`, `get`, `has`, `delete`, `size`
- Looping over a Map with `for...of`
- When to choose a Map instead of a plain object

## Set: A Collection of Unique Values

A **Set** stores values, but it **never keeps duplicates**. Think of a guest
list: each person appears only once.

### Creating a Set

```js
const colors = new Set();
console.log(colors); // => Set(0) {}

// you can start it from an array:
const nums = new Set([1, 2, 3]);
console.log(nums); // => Set(3) { 1, 2, 3 }
```

### add — put a value in

If the value is already there, nothing happens (no duplicates).

```js
const colors = new Set();

colors.add("red");
colors.add("blue");
colors.add("red"); // ignored, already in the set

console.log(colors); // => Set(2) { "red", "blue" }
```

### has — is the value in the set?

```js
const colors = new Set(["red", "blue"]);

console.log(colors.has("red"));   // => true
console.log(colors.has("green")); // => false
```

### delete — remove a value

```js
const colors = new Set(["red", "blue"]);

colors.delete("red");
console.log(colors); // => Set(1) { "blue" }
```

### size — how many values

A Set uses `size`, not `length`.

```js
const colors = new Set(["red", "blue"]);
console.log(colors.size); // => 2
```

## Remove Duplicates: Array <-> Set

The most popular use of a Set is removing duplicate values from an array. You
turn the array into a Set (duplicates vanish), then turn it back into an array.

```js
const numbers = [1, 2, 2, 3, 3, 3, 4];

// array -> set (drops duplicates) -> array
const unique = [...new Set(numbers)];

console.log(unique); // => [1, 2, 3, 4]
```

The `[...]` part is the **spread operator**, which spreads the Set's values into
a new array. (We cover spread fully in the next lesson.) You could also use
`Array.from(new Set(numbers))`, which does the same thing.

## Looping Over a Set

You can use `for...of` or `forEach`.

```js
const colors = new Set(["red", "blue", "green"]);

for (const color of colors) {
  console.log(color);
}
// => "red"
// => "blue"
// => "green"

colors.forEach((color) => console.log(color));
// => same output
```

A Set keeps values in the order you added them.

## Map: Key/Value With Any Key

A **Map** stores key/value pairs, like an object. The big difference: in a Map,
the **key can be any type** — a string, a number, even an object. Objects only
allow string (and symbol) keys.

### Creating a Map

```js
const ages = new Map();
console.log(ages); // => Map(0) {}

// you can start from an array of [key, value] pairs:
const scores = new Map([
  ["Sam", 90],
  ["Alex", 85],
]);
console.log(scores); // => Map(2) { "Sam" => 90, "Alex" => 85 }
```

### set — add or change a pair

`set(key, value)` adds a new pair, or updates the value if the key exists.

```js
const ages = new Map();

ages.set("Sam", 30);
ages.set("Alex", 25);
ages.set("Sam", 31); // updates Sam

console.log(ages); // => Map(2) { "Sam" => 31, "Alex" => 25 }
```

### get — read a value by key

```js
const ages = new Map([["Sam", 30]]);

console.log(ages.get("Sam"));   // => 30
console.log(ages.get("Nobody")); // => undefined
```

### has — does the key exist?

```js
const ages = new Map([["Sam", 30]]);

console.log(ages.has("Sam"));  // => true
console.log(ages.has("Alex")); // => false
```

### delete — remove a pair

```js
const ages = new Map([["Sam", 30], ["Alex", 25]]);

ages.delete("Sam");
console.log(ages); // => Map(1) { "Alex" => 25 }
```

### size — how many pairs

Like a Set, a Map uses `size`.

```js
const ages = new Map([["Sam", 30], ["Alex", 25]]);
console.log(ages.size); // => 2
```

### A key that is not a string

This is something objects cannot do cleanly:

```js
const userRoles = new Map();
const user = { name: "Sam" };

userRoles.set(user, "admin"); // the key is an object!

console.log(userRoles.get(user)); // => "admin"
```

## Looping Over a Map

Use `for...of`. Each loop gives you a `[key, value]` pair, which you can
**destructure** into two variables.

```js
const scores = new Map([
  ["Sam", 90],
  ["Alex", 85],
]);

for (const [name, score] of scores) {
  console.log(`${name}: ${score}`);
}
// => "Sam: 90"
// => "Alex: 85"
```

You can also loop just the keys or just the values:

```js
for (const name of scores.keys()) {
  console.log(name); // => "Sam", then "Alex"
}

for (const score of scores.values()) {
  console.log(score); // => 90, then 85
}
```

## When to Use Map vs a Plain Object

Both store key/value data. Here is a simple guide.

**Use a plain object when:**
- Your keys are fixed, known strings (like `name`, `age`).
- You are modeling one "thing" with named fields.
- You want simple syntax: `obj.name`.

**Use a Map when:**
- Keys are not all strings (numbers, objects, etc.).
- Keys are created at runtime and you do not know them ahead of time.
- You add and remove pairs often, like a lookup table or cache.
- You need to know the size quickly (`map.size`) or keep insertion order
  reliably.
- You want to loop over entries easily with `for...of`.

A quick example where Map shines: counting how many times each word appears.

```js
const words = ["a", "b", "a", "c", "a", "b"];
const counts = new Map();

for (const word of words) {
  const current = counts.get(word) || 0; // 0 if not seen yet
  counts.set(word, current + 1);
}

console.log(counts); // => Map(3) { "a" => 3, "b" => 2, "c" => 1 }
```

## Common Mistakes

- **Using `length` on a Set or Map.** They use `size`, not `length`.
- **Expecting `add` to allow duplicates.** A Set silently ignores repeats.
- **Using object access on a Map.** Use `map.get(key)` and `map.set(key, value)`,
  not `map.key`.
- **Forgetting to destructure in a Map loop.** `for...of` gives `[key, value]`
  pairs; use `for (const [k, v] of map)`.
- **Thinking two equal-looking objects are the same Set value or Map key.** They
  are only the same if they are literally the same object in memory.

## Exercises

1. Create a Set from the array `[3, 1, 3, 2, 1, 1]` to get the unique values.
   Print the result as an array and print its `size`.
2. Build a Set of your three favorite foods. Use `has` to check if "pizza" is in
   it, then `delete` one food and print the new size.
3. Create a Map of three students to their grades. Use `get` to print one grade,
   update a grade with `set`, then loop over all pairs with `for...of` and print
   `name: grade`.
4. Given an array of letters with repeats, use a Map to count how many times each
   letter appears (like the word-count example).

## Key Takeaways

- A **Set** holds **unique** values; use `add`, `has`, `delete`, and `size`.
- Remove array duplicates with `[...new Set(array)]`.
- A **Map** holds key/value pairs where **keys can be any type**; use `set`,
  `get`, `has`, `delete`, and `size`.
- Sets and Maps use `size`, not `length`.
- Loop with `for...of`; for a Map, destructure each `[key, value]` pair.
- Use a **plain object** for fixed named fields; use a **Map** for dynamic keys,
  non-string keys, or frequent add/remove.

---

[← Back to Course Index](../README.md) | [Next: Destructuring, Spread, and Rest →](06-destructuring-spread-rest.md)
