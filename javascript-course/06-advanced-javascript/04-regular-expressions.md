# Regular Expressions

A **regular expression** (or **regex** for short) is a small pattern that
describes what some text should look like. With a regex you can ask questions
like "Does this string contain a number?" or "Find every email in this text" or
"Replace all the extra spaces." It is like a search tool with superpowers.

Fair warning up front: regex can look like cat-walked-on-the-keyboard nonsense at
first (`/^\d{3}-\d{4}$/`). We will go gently, one symbol at a time, and keep every
example practical. You do not need to memorize everything — just understand the
pieces.

## What You'll Learn

- What a regex is (a pattern to match text)
- Two ways to create one: `/pattern/` and `new RegExp()`
- Testing with `test()` and finding with `match()`
- Common pattern pieces: `\d \w \s . * + ? ^ $ [] ()`
- Flags: `g`, `i`, `m`
- Using regex with string methods: `replace`, `split`, `match`, `matchAll`
- Practical examples: rough email check, find all numbers, squash whitespace

---

## What Is a Regex?

A regex is a **pattern**. You compare it against a string to see if (and where)
the string matches that shape. Think of it like a cookie cutter you press onto
dough: the cutter is the pattern, and you find every place in the dough that fits
the shape.

---

## Creating a Regex

There are two ways to write one.

**1. Literal syntax** — the pattern goes between slashes `/ /`:

```js
const pattern = /cat/;
```

**2. The `RegExp` constructor** — useful when the pattern is built from a string
(for example, from user input):

```js
const word = "cat";
const pattern = new RegExp(word);
```

Both make the same kind of object. Use the `/slashes/` form most of the time; use
`new RegExp()` when you need to build the pattern dynamically.

---

## Testing With `test()`

`test()` returns `true` or `false`: does the string contain a match?

```js
const pattern = /cat/;

console.log(pattern.test("I have a cat"));  // => true
console.log(pattern.test("I have a dog"));  // => false
```

Simple and great for yes/no checks like form validation.

---

## Finding With `match()`

`match()` is a **string** method. It returns the matched text (or `null` if
nothing matched).

```js
const text = "I have a cat";
console.log(text.match(/cat/)); // => ['cat', index: 9, ...]

console.log("no pets here".match(/cat/)); // => null
```

We will see how to get *all* matches in a moment.

---

## Common Pattern Pieces

These special symbols are the heart of regex. Learn these and you can read most
patterns.

### Character Shortcuts

| Symbol | Means | Example match |
|--------|-------|---------------|
| `\d` | any **digit** (0–9) | `7` |
| `\w` | any **word** character (letter, digit, `_`) | `a`, `9`, `_` |
| `\s` | any **whitespace** (space, tab, newline) | ` ` |
| `.`  | **any** single character (except newline) | `x`, `5`, `?` |

```js
console.log(/\d/.test("abc123")); // => true  (there is a digit)
console.log(/\s/.test("hello"));  // => false (no whitespace)
```

### Quantifiers (How Many?)

| Symbol | Means |
|--------|-------|
| `*` | zero or more |
| `+` | one or more |
| `?` | zero or one (optional) |

```js
console.log(/\d+/.test("year 2024")); // => true  (one or more digits)
console.log(/colou?r/.test("color")); // => true  (the u is optional)
console.log(/colou?r/.test("colour"));// => true
```

### Anchors (Where?)

| Symbol | Means |
|--------|-------|
| `^` | start of the string |
| `$` | end of the string |

```js
console.log(/^Hello/.test("Hello world")); // => true  (starts with Hello)
console.log(/world$/.test("Hello world")); // => true  (ends with world)
console.log(/^Hi$/.test("Hi there"));      // => false (not the WHOLE string)
```

Using `^` and `$` together means "the entire string must match this pattern."

### Sets `[]` and Groups `()`

- `[ ]` means "any **one** of these characters." `[abc]` matches `a`, `b`, or `c`.
  A range works too: `[a-z]`, `[0-9]`.
- `( )` groups parts together, so a quantifier can apply to the whole group, and
  so you can capture that part.

```js
console.log(/[aeiou]/.test("sky"));   // => false (no vowels)
console.log(/[0-9]+/.test("id-42"));  // => true  (digits present)
console.log(/(ha)+/.test("hahaha"));  // => true  ("ha" repeated)
```

You can also negate a set with `^` *inside* the brackets: `[^0-9]` means "any
character that is **not** a digit."

---

## Flags

Flags go after the closing slash and change how the regex behaves.

| Flag | Meaning |
|------|---------|
| `g` | **global** — find all matches, not just the first |
| `i` | **insensitive** — ignore upper/lowercase |
| `m` | **multiline** — `^` and `$` match each line, not just the whole string |

```js
console.log(/cat/i.test("My CAT is here")); // => true  (case ignored)

const text = "cat cat cat";
console.log(text.match(/cat/));   // => ['cat', ...] just the first
console.log(text.match(/cat/g));  // => ['cat', 'cat', 'cat'] all of them
```

You can combine flags: `/cat/gi` is global **and** case-insensitive.

---

## Using Regex With String Methods

Regex teams up with several string methods.

### `replace()`

Swap matched text for something else. With the `g` flag it replaces all matches.

```js
const text = "I like cats. Cats are great.";
console.log(text.replace(/cats/gi, "dogs"));
// => I like dogs. dogs are great.
```

### `split()`

Break a string apart wherever the pattern matches.

```js
const csv = "apple,banana;cherry orange";
// split on comma, semicolon, OR space
console.log(csv.split(/[,; ]/));
// => ['apple', 'banana', 'cherry', 'orange']
```

### `match()` and `matchAll()`

`match()` with `g` gives you an array of matched strings. `matchAll()` gives you
richer results (including positions and groups), which you loop over.

```js
const text = "Order 12 and 345 items";

console.log(text.match(/\d+/g)); // => ['12', '345']

for (const m of text.matchAll(/\d+/g)) {
  console.log(m[0], "at index", m.index);
}
// => 12 at index 6
// => 345 at index 13
```

---

## Practical Examples

### 1. Roughly Validate an Email

A *perfect* email regex is famously huge. For everyday checks, a simple "rough"
pattern is enough: some characters, an `@`, more characters, a dot, and an ending.

```js
const emailPattern = /^[\w.-]+@[\w.-]+\.\w+$/;

console.log(emailPattern.test("amy@example.com")); // => true
console.log(emailPattern.test("not-an-email"));    // => false
console.log(emailPattern.test("a@b.co"));          // => true
```

Reading it: start (`^`), one or more word/dot/dash chars, an `@`, more of those,
a literal dot (`\.`), one or more word chars, end (`$`).

### 2. Find All Numbers in Text

```js
const report = "We sold 25 units in week 3 and 100 in week 4.";
console.log(report.match(/\d+/g)); // => ['25', '3', '100', '4']
```

### 3. Replace Extra Whitespace

Squash runs of spaces/tabs into a single space and trim the ends:

```js
const messy = "Hello     world   from    JS";
const clean = messy.replace(/\s+/g, " ").trim();
console.log(clean); // => Hello world from JS
```

`\s+` means "one or more whitespace characters," and `g` makes it fix every run.

---

## A Gentle Warning

Regex can get complicated fast, and a too-clever pattern is hard to read and easy
to get wrong. A few friendly rules:

- Start simple. Build the pattern up piece by piece.
- Test it with real examples (and tricky edge cases).
- Add a comment explaining what a non-obvious pattern does.
- For very complex needs, sometimes plain string methods are clearer.
- Online regex testers are your friend — they highlight matches as you type.

---

## Common Mistakes

- **Forgetting the `g` flag** when you want *all* matches. Without it you only get
  the first.
- **Forgetting to escape special characters.** A real dot is `\.`; a plain `.`
  matches *any* character.
- **Confusing `match` (string method) with `test` (regex method).**
  `text.match(/x/)` vs `/x/.test(text)`.
- **Expecting `^...$` and partial match to be the same.** `^Hi$` means the whole
  string is exactly "Hi"; `Hi` alone matches anywhere inside.
- **Building patterns from user input without care.** Special characters in input
  can break or change your pattern.

---

## Exercises

1. Write a regex that tests whether a string contains at least one digit. Try it
   on `"abc"` and `"abc5"`.

2. Use `match()` with the `g` flag to pull every word that starts with a capital
   letter out of `"The Quick Brown fox Jumps"`. (Hint: `[A-Z]\w*`)

3. Use `replace()` to turn all commas in `"1,000,000"` into nothing, giving
   `"1000000"`.

4. Write a rough phone-number checker for the format `123-456-7890` using `\d` and
   `{3}` style counts. (Tip: `{3}` means "exactly 3.") Test a valid and an invalid
   number.

---

## Key Takeaways

- A **regex** is a pattern that describes text.
- Create one with `/pattern/flags` or `new RegExp("pattern", "flags")`.
- `test()` returns true/false; `match()` / `matchAll()` find matches.
- Learn the building blocks: `\d \w \s .` and `* + ?` and `^ $ [] ()`.
- Flags change behavior: `g` (all), `i` (ignore case), `m` (multiline).
- Regex pairs with `replace`, `split`, `match`, and `matchAll`.
- Keep patterns simple, test them, and comment the tricky ones.

---

[← Back to Iterators and Generators](./03-iterators-and-generators.md) | [Next: Functional Programming Ideas →](./05-functional-programming.md)
