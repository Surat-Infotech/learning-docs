# Operators and Expressions

Programs do more than store data — they *work* with it. To add, compare, and
combine values, you use **operators**. An operator is a symbol that performs an
action, like `+` for adding. When you put values and operators together, you make
an **expression**.

## What You'll Learn

- What an expression is
- Arithmetic operators: `+ - * / % **`
- Assignment operators: `= += -=` and more
- Comparison operators and why `===` is safer than `==`
- Logical operators: `&& || !`
- The ternary operator (a short `if`)
- Operator precedence (which operator runs first)
- Joining strings with `+`

## What Is an Expression?

An **expression** is any piece of code that produces a value. If you can imagine
JavaScript "calculating it down to one value," it is an expression.

```js
5 + 3; // an expression that produces 8
"hi"; // an expression that produces the string "hi"
age > 18; // an expression that produces true or false
```

You usually use the result of an expression by storing it in a variable or
printing it:

```js
let total = 5 + 3;
console.log(total); // => 8
```

## Arithmetic Operators

These do math. They work just like a calculator.

| Operator | Meaning | Example | Result |
| --- | --- | --- | --- |
| `+` | add | `4 + 2` | `6` |
| `-` | subtract | `4 - 2` | `2` |
| `*` | multiply | `4 * 2` | `8` |
| `/` | divide | `4 / 2` | `2` |
| `%` | remainder (modulo) | `5 % 2` | `1` |
| `**` | power (exponent) | `2 ** 3` | `8` |

```js
console.log(7 + 3); // => 10
console.log(7 - 3); // => 4
console.log(7 * 3); // => 21
console.log(7 / 2); // => 3.5
console.log(7 % 2); // => 1   (7 divided by 2 leaves a remainder of 1)
console.log(2 ** 4); // => 16  (2 to the power of 4)
```

The **remainder operator** `%` (also called modulo) gives what is left over after
division. It is very handy for checking if a number is even or odd:

```js
console.log(10 % 2); // => 0   (even numbers leave no remainder)
console.log(11 % 2); // => 1   (odd numbers leave a remainder of 1)
```

## Assignment Operators

You already know `=`, which puts a value into a variable. There are also
shortcuts that do math *and* assign in one step.

```js
let score = 10;

score += 5; // same as: score = score + 5
console.log(score); // => 15

score -= 3; // same as: score = score - 3
console.log(score); // => 12

score *= 2; // same as: score = score * 2
console.log(score); // => 24

score /= 4; // same as: score = score / 4
console.log(score); // => 6
```

These are just a faster way to write a common pattern. `count += 1` means "add 1
to count and store it back."

## Comparison Operators

These compare two values and always produce a boolean (`true` or `false`).

| Operator | Meaning |
| --- | --- |
| `===` | strictly equal (same value AND same type) |
| `!==` | strictly not equal |
| `==` | loosely equal (avoid — explained below) |
| `!=` | loosely not equal (avoid) |
| `>` | greater than |
| `<` | less than |
| `>=` | greater than or equal |
| `<=` | less than or equal |

```js
console.log(5 > 3); // => true
console.log(5 < 3); // => false
console.log(5 >= 5); // => true
console.log(4 <= 2); // => false
```

### Always use `===` instead of `==`

This is one of the most important habits in JavaScript.

- `===` checks if two values are equal **and** the same type. This is strict and
  predictable.
- `==` checks equality but first tries to convert the types to match. This causes
  confusing results.

```js
console.log(5 === 5); // => true
console.log(5 === "5"); // => false (number vs string — different types)

console.log(5 == "5"); // => true  (== converts "5" to 5 first — confusing!)
console.log(0 == false); // => true  (more confusion)
console.log(0 === false); // => false (clear and correct)
```

**Rule: always use `===` and `!==`.** They mean exactly what they say. You will
learn *why* `==` is so confusing in the next lesson on coercion.

## Logical Operators

Logical operators combine or flip boolean values. They power decisions.

- `&&` is **AND**: true only if both sides are true.
- `||` is **OR**: true if at least one side is true.
- `!` is **NOT**: flips true to false and false to true.

```js
console.log(true && true); // => true
console.log(true && false); // => false

console.log(false || true); // => true
console.log(false || false); // => false

console.log(!true); // => false
console.log(!false); // => true
```

A real-world example: a user can enter if they are an adult **and** have a ticket.

```js
let isAdult = true;
let hasTicket = true;
console.log(isAdult && hasTicket); // => true (both conditions met)
```

## The Ternary Operator

The **ternary operator** is a short way to choose between two values based on a
condition. It uses `?` and `:`.

The shape is: `condition ? valueIfTrue : valueIfFalse`.

```js
let age = 20;
let status = age >= 18 ? "adult" : "minor";
console.log(status); // => adult
```

Read it as a question: "Is `age >= 18`? If yes, use `"adult"`; if no, use
`"minor"`." It is a compact version of an `if/else`, which you will meet in the
conditionals lesson.

## Operator Precedence

When an expression has several operators, JavaScript decides which to run first.
This order is called **precedence**. It mostly follows normal math rules:
multiply and divide happen before add and subtract.

```js
console.log(2 + 3 * 4); // => 14 (3 * 4 happens first, then + 2)
```

Use parentheses `()` to control the order yourself. Code inside parentheses runs
first. When in doubt, add parentheses — they make your intent clear.

```js
console.log((2 + 3) * 4); // => 20 (the addition is forced first)
```

A handy short list, from runs-first to runs-last:

1. `()` grouping
2. `**` power
3. `*` `/` `%`
4. `+` `-`
5. comparison (`>`, `<`, `===`, etc.)
6. `&&`
7. `||`

## String Concatenation With `+`

The `+` operator has a second job: when used with text, it **joins** strings
together. This is called **concatenation**.

```js
let first = "John";
let last = "Doe";
console.log(first + " " + last); // => John Doe
```

Be careful: if one side is a string, `+` joins instead of adding.

```js
console.log("5" + 3); // => "53"  (joined as text, not added)
console.log(5 + 3); // => 8     (added as numbers)
```

For building text with variables, template literals (backticks) are often
cleaner than many `+` signs:

```js
let name = "Sara";
let age = 9;
console.log(`${name} is ${age} years old`); // => Sara is 9 years old
```

## Common Mistakes

- **Using `==` instead of `===`.** Always use `===` to avoid surprise type
  conversions.
- **Forgetting precedence.** `2 + 3 * 4` is `14`, not `20`. Use parentheses when
  unsure.
- **Accidental string joining.** `"5" + 3` gives `"53"`, not `8`. Check your
  types.
- **Confusing `=` and `===`.** `=` assigns a value; `===` compares two values.
- **Reading a ternary as an `if` that does an action.** A ternary chooses a
  *value*; it is best for simple either/or choices.

## Exercises

1. Use `%` to print whether the numbers `4`, `7`, and `10` are even or odd (think
   about what `number % 2` gives you).
2. Start with `let price = 100;`. Use `+=`, `-=`, and `*=` to change it three
   times, printing the value after each step.
3. Compare `7 === "7"` and `7 == "7"`. Print both and explain in your own words
   why they differ.
4. Write a ternary that stores `"pass"` if a `grade` variable is `50` or more, and
   `"fail"` otherwise. Test it with a few values.

## Key Takeaways

- An expression is any code that produces a value.
- Arithmetic operators (`+ - * / % **`) do math; `%` gives the remainder.
- Assignment shortcuts like `+=` do math and store the result in one step.
- Comparison operators return booleans; always prefer `===` and `!==` over `==`
  and `!=`.
- Logical operators `&& || !` combine and flip booleans.
- The ternary `condition ? a : b` picks one of two values.
- Precedence decides the order operators run; parentheses let you control it.
- `+` joins strings, which can cause accidental text when mixing types.

---

[← Back to Course Index](../README.md) | [Next: Type Conversion and Coercion →](04-type-conversion-and-coercion.md)
