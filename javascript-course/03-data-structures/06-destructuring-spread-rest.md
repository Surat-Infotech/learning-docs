# Destructuring, Spread, and Rest

This lesson covers three modern features that make working with arrays and
objects much shorter and cleaner. They share a special syntax, so we learn them
together.

- **Destructuring** pulls values out of arrays and objects into variables.
- **Spread** (`...`) copies or merges arrays and objects.
- **Rest** (`...`) gathers many values into one array.

These show up constantly in real JavaScript code. Once they click, your code gets
a lot tidier.

## What You'll Learn

- Array destructuring
- Object destructuring, with renaming and default values
- Nested destructuring
- The spread operator for copying and merging
- Spread to pass arguments to a function
- The rest parameter in functions (`...args`)
- Rest in destructuring
- Swapping variables with destructuring
- Immutable update patterns: `[...arr, x]` and `{ ...obj, key: val }`

## Array Destructuring

Normally you pull items out of an array by index, one at a time.

```js
const colors = ["red", "green", "blue"];

const first = colors[0];
const second = colors[1];
console.log(first, second); // => "red" "green"
```

**Destructuring** does this in one line. You write a pattern with `[ ]` on the
**left** of the `=`. The names get matched by position.

```js
const colors = ["red", "green", "blue"];

const [first, second] = colors;
console.log(first, second); // => "red" "green"
```

You can skip items by leaving a gap (an empty comma slot):

```js
const colors = ["red", "green", "blue"];

const [, , third] = colors; // skip the first two
console.log(third); // => "blue"
```

You can also give a **default** value, used when the item is missing
(`undefined`):

```js
const colors = ["red"];

const [a, b = "white"] = colors;
console.log(a, b); // => "red" "white"  (b had no value, so the default is used)
```

## Object Destructuring

For objects, you match by **key name**, not position. The pattern uses `{ }`.

```js
const person = { name: "Sam", age: 30 };

const { name, age } = person;
console.log(name, age); // => "Sam" 30
```

The variable names must match the keys.

### Renaming

You can give the variable a different name with a colon: `key: newName`.

```js
const person = { name: "Sam", age: 30 };

const { name: fullName } = person;
console.log(fullName); // => "Sam"
// console.log(name);  // error: there is no variable called "name"
```

### Defaults

Provide a default for keys that might be missing.

```js
const settings = { theme: "dark" };

const { theme, fontSize = 14 } = settings;
console.log(theme, fontSize); // => "dark" 14  (fontSize used its default)
```

### Renaming and defaults together

```js
const settings = { theme: "dark" };

const { fontSize: size = 14 } = settings;
console.log(size); // => 14
```

A common real use: pulling fields out of a function's object parameter.

```js
function greet({ name, age }) {
  return `Hi ${name}, age ${age}`;
}

console.log(greet({ name: "Sam", age: 30 })); // => "Hi Sam, age 30"
```

## Nested Destructuring

You can reach into nested arrays and objects by nesting the pattern.

```js
const user = {
  name: "Sam",
  address: { city: "Pune", zip: "411001" },
};

const {
  address: { city },
} = user;

console.log(city); // => "Pune"
```

Nested arrays work the same way:

```js
const grid = [[1, 2], [3, 4]];
const [[a, b]] = grid;
console.log(a, b); // => 1 2
```

> Tip: Nested destructuring is powerful but can get hard to read. If it looks
> confusing, it is fine to do it in two simple steps instead.

## The Spread Operator (...)

The **spread operator** is three dots `...`. It "spreads out" the items of an
array or the properties of an object. It is great for copying and merging.

### Copy an array

```js
const original = [1, 2, 3];
const copy = [...original];

copy.push(4);
console.log(copy);     // => [1, 2, 3, 4]
console.log(original); // => [1, 2, 3]  (original is safe)
```

This makes a **new** array, so changing the copy does not touch the original.

### Merge arrays

```js
const a = [1, 2];
const b = [3, 4];

const merged = [...a, ...b];
console.log(merged); // => [1, 2, 3, 4]
```

### Copy and merge objects

```js
const base = { name: "Sam", age: 30 };

const copy = { ...base };
const updated = { ...base, age: 31, city: "Pune" };

console.log(copy);    // => { name: "Sam", age: 30 }
console.log(updated); // => { name: "Sam", age: 31, city: "Pune" }
```

When two objects have the same key, the **later** one wins. That is how `age: 31`
overrides the original `age: 30`.

## Spread to Pass Arguments

You can spread an array into separate arguments when calling a function.

```js
function add(x, y, z) {
  return x + y + z;
}

const numbers = [1, 2, 3];

console.log(add(...numbers)); // => 6  (same as add(1, 2, 3))
```

A handy example: `Math.max` wants separate numbers, not an array.

```js
const scores = [82, 95, 67];
console.log(Math.max(...scores)); // => 95
```

## The Rest Parameter (...args)

The **rest parameter** is also three dots, but it does the **opposite** of
spread. In a function, it gathers all the leftover arguments into one array.

```js
function sumAll(...numbers) {
  // "numbers" is a real array of every argument passed
  return numbers.reduce((total, n) => total + n, 0);
}

console.log(sumAll(1, 2, 3));    // => 6
console.log(sumAll(10, 20));     // => 30
console.log(sumAll());           // => 0
```

You can take some named parameters first, then gather the rest:

```js
function intro(first, ...others) {
  console.log("First:", first);
  console.log("Others:", others);
}

intro("Sam", "Alex", "Jo");
// => First: Sam
// => Others: ["Alex", "Jo"]
```

The rest parameter must be **last** in the parameter list.

> Spread vs rest: same three dots, opposite jobs. **Spread** unpacks an array
> into pieces. **Rest** packs pieces into an array. Context tells them apart.

## Rest in Destructuring

You can use rest inside destructuring to collect "everything else".

With arrays:

```js
const numbers = [1, 2, 3, 4, 5];

const [first, second, ...others] = numbers;
console.log(first);  // => 1
console.log(second); // => 2
console.log(others); // => [3, 4, 5]
```

With objects (collect the remaining properties into a new object):

```js
const person = { name: "Sam", age: 30, city: "Pune" };

const { name, ...rest } = person;
console.log(name); // => "Sam"
console.log(rest); // => { age: 30, city: "Pune" }
```

## Swapping Variables

Destructuring gives a neat trick: swap two variables in one line, no temporary
variable needed.

```js
let a = 1;
let b = 2;

[a, b] = [b, a];

console.log(a, b); // => 2 1
```

## Immutable Update Patterns

**Immutable** means "do not change the original; make a new copy with the
change". This is a very common style, especially in modern apps. Spread makes it
easy.

### Add to an array (without changing it)

```js
const list = [1, 2, 3];

const withFour = [...list, 4]; // add at the end
const withZero = [0, ...list]; // add at the start

console.log(withFour); // => [1, 2, 3, 4]
console.log(withZero); // => [0, 1, 2, 3]
console.log(list);     // => [1, 2, 3]  (untouched)
```

### Update an object field (without changing it)

```js
const user = { name: "Sam", age: 30 };

const olderUser = { ...user, age: 31 };

console.log(olderUser); // => { name: "Sam", age: 31 }
console.log(user);      // => { name: "Sam", age: 30 }  (untouched)
```

### Remove an item from an array (immutably)

Use `filter` to keep everything except the one you want gone:

```js
const list = [1, 2, 3, 4];

const withoutThree = list.filter((n) => n !== 3);
console.log(withoutThree); // => [1, 2, 4]
console.log(list);         // => [1, 2, 3, 4]  (untouched)
```

These patterns keep your original data safe, which prevents many hard-to-find
bugs.

## Common Mistakes

- **Mixing up array and object destructuring.** Use `[ ]` for arrays (by
  position) and `{ }` for objects (by key name).
- **Wrong key name in object destructuring.** The variable name must match the
  key, unless you rename with `key: newName`.
- **Putting the rest parameter not last.** `function f(...args, x)` is an error.
- **Thinking spread copies deeply.** Spread makes a **shallow** copy. Nested
  objects/arrays inside are still shared. For simple data this is usually fine.
- **Confusing spread and rest.** Spread unpacks; rest gathers. They look the
  same but do opposite things.

## Exercises

1. Given `const rgb = [255, 128, 0]`, use array destructuring to make variables
   `red`, `green`, and `blue`. Then add a default `alpha = 1`.
2. Given `const book = { title: "JS", author: "Sam" }`, destructure `title` and
   rename `author` to `writer`. Add a default `year = 2024`.
3. Write a function `multiplyAll(...nums)` that returns the product of all its
   arguments using `reduce`. Test it with several numbers.
4. Start with `const todo = { id: 1, text: "Learn", done: false }`. Using spread,
   create a new object that is the same but with `done` set to `true`. Confirm the
   original is unchanged.

## Key Takeaways

- **Destructuring** pulls values into variables: `[a, b] = arr` (by position) and
  `{ x, y } = obj` (by key).
- Object destructuring supports **renaming** (`key: newName`) and **defaults**
  (`key = value`).
- **Spread** (`...`) copies and merges arrays/objects and passes array items as
  function arguments.
- **Rest** (`...args`) gathers many arguments or leftover items into one array.
  It must come last.
- Swap variables with `[a, b] = [b, a]`.
- For **immutable updates**, use `[...arr, x]` and `{ ...obj, key: val }` to make
  a changed copy without touching the original.

---

[← Back to Course Index](../README.md)
