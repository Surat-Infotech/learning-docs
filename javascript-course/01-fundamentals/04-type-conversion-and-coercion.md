# Type Conversion and Coercion

Sometimes a value is the wrong type for what you need. You might have the text
`"42"` but need the number `42`. Changing a value from one type to another is
called **type conversion**. Sometimes *you* do it on purpose, and sometimes
JavaScript does it automatically behind your back. This lesson explains both, so
the strange results stop being a mystery.

## What You'll Learn

- **Explicit conversion**: changing types on purpose with `Number()`, `String()`,
  `Boolean()`, `parseInt()`, and `parseFloat()`
- **Implicit coercion**: when JavaScript changes types automatically
- Why `"5" + 1` and `"5" - 1` give very different results
- **Truthy** and **falsy** values, and the full list of falsy values
- Why `==` is confusing and `===` is safe

## Two Ways Types Change

1. **Explicit conversion (type casting):** you write code that clearly changes
   the type. This is safe and easy to read.
2. **Implicit coercion:** JavaScript silently changes a type for you during an
   operation. This is where surprises come from.

## Explicit Conversion

### `Number()` — turn something into a number

```js
console.log(Number("42")); // => 42
console.log(Number("3.14")); // => 3.14
console.log(Number("hello")); // => NaN   (not a valid number)
console.log(Number("")); // => 0     (empty string becomes 0)
console.log(Number(true)); // => 1
console.log(Number(false)); // => 0
```

### `String()` — turn something into text

```js
console.log(String(42)); // => "42"
console.log(String(true)); // => "true"
console.log(String(null)); // => "null"
```

### `Boolean()` — turn something into true or false

```js
console.log(Boolean(1)); // => true
console.log(Boolean(0)); // => false
console.log(Boolean("hi")); // => true
console.log(Boolean("")); // => false
```

(We explain which values become `false` in the truthy/falsy section below.)

### `parseInt()` and `parseFloat()` — pull a number out of text

These are special: they read a string from the start and grab the number part,
even if other characters follow.

- `parseInt()` reads a **whole number** (integer).
- `parseFloat()` reads a number that **can have decimals**.

```js
console.log(parseInt("42px")); // => 42   (stops at "px")
console.log(parseInt("3.99")); // => 3    (integers only — drops the decimal)
console.log(parseFloat("3.99")); // => 3.99 (keeps the decimal)
console.log(parseInt("hello")); // => NaN  (no number at the start)
```

Notice the difference from `Number()`:

```js
console.log(Number("42px")); // => NaN   (whole string must be a valid number)
console.log(parseInt("42px")); // => 42    (grabs the number it can find)
```

## Implicit Coercion

**Implicit coercion** is when JavaScript changes a type *automatically* during an
operation. The most common place this happens is with the `+` operator and other
math operators.

### The `+` operator prefers strings

If either side of `+` is a string, JavaScript converts the other side to a string
and **joins** them.

```js
console.log("5" + 1); // => "51"  (1 becomes "1", then joined)
console.log("5" + true); // => "5true"
console.log(1 + "5"); // => "15"
```

### Other math operators prefer numbers

The operators `-`, `*`, `/`, and `%` have no "join" job, so they convert strings
to numbers instead.

```js
console.log("5" - 1); // => 4   ("5" becomes 5, then subtract)
console.log("5" * 2); // => 10
console.log("10" / "2"); // => 5
console.log("abc" - 1); // => NaN ("abc" cannot become a number)
```

This is the classic confusing pair to remember:

```js
console.log("5" + 1); // => "51"  (+ joins as text)
console.log("5" - 1); // => 4     (- converts to numbers)
```

Same numbers, different operator, completely different result. That is coercion.

## Truthy and Falsy Values

In places that expect a yes/no answer (like an `if` condition), JavaScript treats
every value as either **truthy** (acts like `true`) or **falsy** (acts like
`false`).

### The complete list of falsy values

There are only a handful of falsy values. Memorize this short list — everything
**not** on it is truthy.

```js
Boolean(false); // => false
Boolean(0); // => false
Boolean(-0); // => false
Boolean(0n); // => false   (bigint zero)
Boolean(""); // => false   (empty string)
Boolean(null); // => false
Boolean(undefined); // => false
Boolean(NaN); // => false
```

So the falsy values are: **`false`, `0`, `-0`, `0n`, `""` (empty string),
`null`, `undefined`, and `NaN`.** That is the whole list.

### Everything else is truthy

```js
Boolean("hello"); // => true
Boolean("0"); // => true   (a non-empty string, even "0"!)
Boolean(42); // => true
Boolean(-5); // => true
Boolean([]); // => true   (an empty array is still truthy)
Boolean({}); // => true   (an empty object is still truthy)
```

Two surprises beginners hit often: the string `"0"` is truthy (it is not empty),
and an empty array `[]` or empty object `{}` is truthy.

## Why `==` Is Confusing and `===` Is Safe

Now you can understand the warning from the operators lesson.

The `==` operator uses **coercion**: before comparing, it converts the two values
to a matching type. This creates results that are hard to predict.

```js
console.log(0 == ""); // => true   (both coerced toward each other)
console.log(0 == "0"); // => true
console.log("" == "0"); // => false  (wait, what?!)
console.log(null == undefined); // => true
console.log(1 == true); // => true
console.log(0 == false); // => true
```

Look at those first three lines: `0 == ""` is true and `0 == "0"` is true, but
`"" == "0"` is false. That is not consistent and is very hard to reason about.

The `===` operator does **no** coercion. It only returns `true` when the values
are the same type *and* the same value. The results are always predictable.

```js
console.log(0 === ""); // => false
console.log(0 === "0"); // => false
console.log(1 === true); // => false
console.log(null === undefined); // => false
console.log(5 === 5); // => true
```

**Rule: always use `===` and `!==`.** If you need to convert a type, do it
explicitly first (for example with `Number(...)`), then compare with `===`.

```js
let input = "5";
console.log(Number(input) === 5); // => true  (clear and safe)
```

## Common Mistakes

- **Joining when you meant to add.** `"5" + 1` is `"51"`. Convert with
  `Number(...)` first if you want math.
- **Expecting `Number("42px")` to work.** It returns `NaN`. Use `parseInt(...)`
  to pull a number out of mixed text.
- **Forgetting `parseInt` drops decimals.** Use `parseFloat` to keep them.
- **Thinking `"0"` or `[]` is falsy.** Both are truthy. Only the listed values are
  falsy.
- **Using `==` to "be flexible."** It causes inconsistent results. Convert
  explicitly, then use `===`.

## Exercises

1. Convert the string `"123"` to a number two different ways, and convert the
   number `123` to a string. Print the `typeof` each result to confirm.
2. Predict the output of `"10" + 5` and `"10" - 5` before running them, then run
   them and check if you were right.
3. Write down the full list of falsy values from memory, then test each one with
   `Boolean(...)` to confirm it returns `false`.
4. Compare `"" == 0` and `"" === 0`. Explain in plain English why the first is
   `true` and the second is `false`.

## Key Takeaways

- Type conversion changes a value from one type to another.
- Convert explicitly with `Number()`, `String()`, `Boolean()`, `parseInt()`, and
  `parseFloat()`.
- `parseInt`/`parseFloat` pull a number out of text; `Number()` needs the whole
  string to be a valid number.
- Implicit coercion is automatic: `+` joins as text, while `- * / %` convert to
  numbers.
- The only falsy values are `false`, `0`, `-0`, `0n`, `""`, `null`, `undefined`,
  and `NaN`; everything else is truthy.
- `==` uses coercion and is unpredictable; always use `===` for safe, clear
  comparisons.

---

[← Back to Course Index](../README.md) | [Next: Conditionals →](05-conditionals.md)
