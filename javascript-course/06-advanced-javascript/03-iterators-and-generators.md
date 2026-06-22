# Iterators and Generators

You already loop over arrays and strings with `for...of`. Have you ever wondered
*how* JavaScript knows how to step through them one item at a time? The answer is
a built-in rule called the **iterator protocol**. Once you understand it, you can
make your *own* things loopable, and you can build clever value-producers called
**generators**.

Do not worry — these sound fancy, but the core idea is simple: something that can
hand you items one at a time, on demand.

## What You'll Learn

- What **iterable** means (things you can loop with `for...of`)
- The **iterator protocol**: `next()` returning `{ value, done }`
- How to make your **own iterable**
- **Generator functions** (`function*`) and **`yield`**
- Pausing and resuming a function
- Producing sequences (an id generator, an infinite counter)
- **Spreading** a generator into an array

---

## What Does "Iterable" Mean?

An **iterable** is anything you can loop over with `for...of`. Arrays, strings,
`Map`s, and `Set`s are all iterable out of the box.

```js
for (const letter of "hi") {
  console.log(letter);
}
// => h
// => i
```

Behind the scenes, `for...of` asks the iterable: "Give me your iterator." Then it
keeps asking that iterator for the next item until there are no more.

---

## The Iterator Protocol

An **iterator** is an object with one important method: `next()`. Every time you
call `next()`, it returns an object with two fields:

- `value` — the next item.
- `done` — `false` if there is more, `true` when finished.

Think of a **vending machine**. Each time you press the button (`next()`), one
snack drops out (`value`). When the machine is empty, it tells you it is done
(`done: true`).

Let's see it by hand on an array:

```js
const numbers = [10, 20];
const iterator = numbers[Symbol.iterator](); // get the iterator

console.log(iterator.next()); // => { value: 10, done: false }
console.log(iterator.next()); // => { value: 20, done: false }
console.log(iterator.next()); // => { value: undefined, done: true }
```

`Symbol.iterator` is the special "key" where an object stores its iterator-maker.
You rarely call this by hand — `for...of` does it for you — but seeing it makes
the magic clear.

---

## Making Your Own Iterable

To make your own object loopable, give it a `[Symbol.iterator]` method that
returns an iterator (an object with a `next()` method).

Here is a `range` object that counts from `start` to `end`:

```js
const range = {
  start: 1,
  end: 4,

  [Symbol.iterator]() {
    let current = this.start;
    const end = this.end;

    return {
      next() {
        if (current <= end) {
          return { value: current++, done: false };
        }
        return { value: undefined, done: true };
      },
    };
  },
};

for (const n of range) {
  console.log(n);
}
// => 1
// => 2
// => 3
// => 4
```

This works, but writing `next()` and tracking `current` and `done` by hand is
fiddly. Good news: **generators** do all of this for you automatically.

---

## Generator Functions and `yield`

A **generator** is a special function that can **pause** in the middle and later
**resume** right where it left off. You write it with a star: `function*`. Inside,
you use the keyword **`yield`** to hand out a value and pause.

```js
function* greetings() {
  yield "Hello";
  yield "Hi";
  yield "Hey";
}

const g = greetings(); // does NOT run the body yet!
console.log(g.next()); // => { value: 'Hello', done: false }
console.log(g.next()); // => { value: 'Hi', done: false }
console.log(g.next()); // => { value: 'Hey', done: false }
console.log(g.next()); // => { value: undefined, done: true }
```

Notice the generator already returns objects shaped like `{ value, done }` — it
*is* an iterator automatically. So you can loop it with `for...of`:

```js
for (const word of greetings()) {
  console.log(word);
}
// => Hello
// => Hi
// => Hey
```

---

## Pausing and Resuming

This is the special power of generators. Calling a generator does **not** run its
code. Each `next()` runs the function until the next `yield`, then **freezes**
everything — variables and all — until you call `next()` again.

```js
function* steps() {
  console.log("Step 1 running");
  yield "a";
  console.log("Step 2 running");
  yield "b";
  console.log("Step 3 running");
}

const s = steps();
s.next(); // logs "Step 1 running", returns { value: 'a', done: false }
s.next(); // logs "Step 2 running", returns { value: 'b', done: false }
s.next(); // logs "Step 3 running", returns { value: undefined, done: true }
```

Think of it like a movie you can pause. The scene freezes exactly where you
stopped, and pressing play continues from that frame.

---

## Producing Sequences

Generators shine when producing a series of values, especially when the series is
long or even endless.

### An ID Generator

Need unique IDs that never repeat? A generator remembers the last one for you.

```js
function* idMaker() {
  let id = 1;
  while (true) {     // looks infinite, but it pauses at yield
    yield id++;
  }
}

const ids = idMaker();
console.log(ids.next().value); // => 1
console.log(ids.next().value); // => 2
console.log(ids.next().value); // => 3
```

The `while (true)` loop would normally freeze your program forever. But because
`yield` pauses the function, it only runs when you ask for the next value. Safe
and handy.

### An Infinite Counter (Step By Step)

```js
function* counter(step = 1) {
  let n = 0;
  while (true) {
    yield n;
    n += step;
  }
}

const evens = counter(2);
console.log(evens.next().value); // => 0
console.log(evens.next().value); // => 2
console.log(evens.next().value); // => 4
```

> Careful: never do `for...of` over an infinite generator without a `break`, or
> your program will loop forever.

---

## Spreading a Generator

Because generators are iterable, you can spread a **finite** one into an array
with `...`:

```js
function* range(start, end) {
  for (let i = start; i <= end; i++) {
    yield i;
  }
}

const nums = [...range(1, 5)];
console.log(nums); // => [1, 2, 3, 4, 5]
```

You can also use them anywhere an iterable is accepted, like `Array.from`:

```js
console.log(Array.from(range(3, 6))); // => [3, 4, 5, 6]
```

> Reminder: only spread generators that *end*. Spreading an infinite generator
> will try to collect endless values and crash.

---

## Common Mistakes

- **Forgetting the star.** A generator must be `function*`, not `function`.
- **Expecting the body to run on call.** `myGen()` only creates the generator. The
  body runs as you call `next()`.
- **Reading `.value` without checking `.done`.** When `done` is `true`, `value` is
  usually `undefined`.
- **Spreading or `for...of` over an infinite generator** with no `break`. This
  hangs your program.
- **Reusing a finished generator.** Once `done` is `true`, it stays done. Make a
  new one to start over.

---

## Exercises

1. Write a generator `function* countdown(n)` that yields numbers from `n` down to
   `1`. Loop it with `for...of` and print each.

2. Make a custom iterable object `weekdays` so that `for...of weekdays` prints
   "Mon", "Tue", "Wed", "Thu", "Fri" (use a `[Symbol.iterator]` method, or build a
   generator and assign it).

3. Write an infinite `function* fibonacci()` generator that yields the Fibonacci
   sequence (1, 1, 2, 3, 5, 8, ...). Use `next()` to print the first 8 numbers.
   Be careful not to loop forever.

4. Use the spread operator to turn a finite `range(10, 15)` generator into an
   array and log it.

---

## Key Takeaways

- An **iterable** is anything you can loop with `for...of`.
- An **iterator** has a `next()` method returning `{ value, done }`.
- You can make custom iterables with a `[Symbol.iterator]` method — but it is
  tedious.
- **Generators** (`function*` + `yield`) build iterators for you automatically.
- Generators can **pause and resume**, which makes endless sequences safe.
- You can **spread** finite generators into arrays, but never infinite ones.

---

[← Back to OOP](./02-oop-classes-prototypes.md) | [Next: Regular Expressions →](./04-regular-expressions.md)
