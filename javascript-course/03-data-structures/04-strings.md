# Strings and Template Literals

A **string** is text. Any time you work with words, names, messages, or
sentences, you are using strings. You met strings early in the course; now we go
deeper into the useful things you can do with them.

Strings are like a row of characters. In fact, you can treat a string a bit like
a read-only array of letters.

## What You'll Learn

- How to create strings (three kinds of quotes)
- That strings are **immutable** (cannot be changed in place)
- The `length` of a string
- Reading a character by position (`charAt`, brackets)
- Common methods: `toUpperCase`, `toLowerCase`, `trim`, `includes`, `indexOf`,
  `slice`, `substring`, `replace`, `replaceAll`, `split`, `repeat`,
  `startsWith`, `endsWith`, `padStart`
- **Template literals** with backticks and `${}`, including multi-line text
- Escaping special characters

## Creating Strings

You can use single quotes, double quotes, or backticks. The first two work the
same. Backticks are special (more on that later).

```js
const a = "Hello";   // double quotes
const b = 'Hello';   // single quotes
const c = `Hello`;   // backticks (template literal)

console.log(a, b, c); // => Hello Hello Hello
```

Pick one style and be consistent. Many people use double quotes or backticks.

## Strings Are Immutable

**Immutable** means "cannot be changed". You cannot change a single letter inside
a string. Trying to do so does nothing.

```js
let word = "cat";
word[0] = "b"; // this silently fails
console.log(word); // => "cat"  (still "cat")
```

So how do we "change" strings? We **make a new string** from the old one. Every
string method below returns a **new** string and leaves the original alone.

```js
let word = "cat";
let newWord = "b" + word.slice(1); // build a new string
console.log(newWord); // => "bat"
console.log(word);    // => "cat"  (unchanged)
```

## Length

`length` tells you how many characters are in the string (spaces count too).

```js
const name = "Sam";
console.log(name.length); // => 3

const sentence = "I code";
console.log(sentence.length); // => 6  (the space counts)
```

## Indexing and charAt

Like arrays, strings use **0-based** positions. The first letter is at index `0`.

```js
const word = "hello";

console.log(word[0]);       // => "h"
console.log(word[1]);       // => "e"
console.log(word.charAt(0)); // => "h"  (same idea, method form)
```

`word[0]` and `word.charAt(0)` do almost the same thing. Brackets are shorter and
very common.

## Common String Methods

Remember: these all return a **new** string (or a number/boolean). The original
is never changed.

### toUpperCase and toLowerCase

```js
console.log("hello".toUpperCase()); // => "HELLO"
console.log("HELLO".toLowerCase()); // => "hello"
```

### trim — remove spaces at the ends

```js
const messy = "   hello   ";
console.log(messy.trim()); // => "hello"  (spaces removed from both ends)
```

This is great for cleaning up user input from a form.

### includes — does it contain something?

```js
console.log("hello world".includes("world")); // => true
console.log("hello world".includes("bye"));   // => false
```

### indexOf — where is it?

Returns the position of the first match, or `-1` if not found.

```js
console.log("hello".indexOf("l")); // => 2
console.log("hello".indexOf("z")); // => -1
```

### slice — cut out a part

`slice(start, end)` returns characters from `start` up to (not including) `end`.

```js
const text = "JavaScript";

console.log(text.slice(0, 4)); // => "Java"
console.log(text.slice(4));    // => "Script"  (to the end)
console.log(text.slice(-6));   // => "Script"  (negative counts from the end)
```

### substring — similar to slice

`substring(start, end)` is much like `slice`. The main difference is that
`substring` does not understand negative numbers (it treats them as `0`). For
beginners, prefer `slice`.

```js
const text = "JavaScript";
console.log(text.substring(0, 4)); // => "Java"
```

### replace and replaceAll

`replace` swaps the **first** match. `replaceAll` swaps **every** match.

```js
const text = "I like cats. Cats are great.";

console.log(text.replace("cats", "dogs"));
// => "I like dogs. Cats are great."  (only the first, lowercase one)

console.log("a-b-c".replaceAll("-", " "));
// => "a b c"  (all dashes replaced)
```

### split — string to array

`split` breaks a string into an array using a separator.

```js
const csv = "Sam,Alex,Jo";
console.log(csv.split(",")); // => ["Sam", "Alex", "Jo"]

const sentence = "I love code";
console.log(sentence.split(" ")); // => ["I", "love", "code"]

console.log("abc".split("")); // => ["a", "b", "c"]  (into letters)
```

`split` is the opposite of the array `join` method.

### repeat — copy the string

```js
console.log("ha".repeat(3)); // => "hahaha"
console.log("=".repeat(5));  // => "====="
```

### startsWith and endsWith

These return `true` or `false`.

```js
const file = "report.pdf";

console.log(file.startsWith("report")); // => true
console.log(file.endsWith(".pdf"));     // => true
console.log(file.endsWith(".txt"));     // => false
```

### padStart — pad the start to a length

`padStart(targetLength, padString)` adds characters to the **front** until the
string reaches the target length. Great for things like clock times.

```js
console.log("5".padStart(2, "0"));   // => "05"
console.log("42".padStart(5, "*"));  // => "***42"
```

(There is also `padEnd`, which pads the end the same way.)

## Template Literals

A **template literal** uses backticks `` ` `` instead of quotes. It has two
superpowers.

### 1. Insert values with ${}

Inside backticks, you can drop in any value or expression using `${ }`. This is
called **interpolation** and it is much cleaner than joining with `+`.

```js
const name = "Sam";
const age = 30;

// old way with + :
const old = "Hi " + name + ", you are " + age;

// template literal way:
const greeting = `Hi ${name}, you are ${age}`;

console.log(greeting); // => "Hi Sam, you are 30"
```

You can even do math or call methods inside `${ }`:

```js
const price = 20;
console.log(`Total: $${price * 1.1}`);          // => "Total: $22"
console.log(`Shout: ${"hello".toUpperCase()}`); // => "Shout: HELLO"
```

### 2. Multi-line strings

With backticks, you can write text across several lines, and the line breaks are
kept.

```js
const message = `Dear Sam,

Thank you for joining.
See you soon!`;

console.log(message);
// => Dear Sam,
// =>
// => Thank you for joining.
// => See you soon!
```

With normal quotes, you would need `\n` for new lines (explained next).

## Escaping Characters

Sometimes a character has a special meaning, or you cannot type it directly. You
use a **backslash** `\` to "escape" it.

```js
// a quote inside the same kind of quotes:
const text1 = "She said \"hi\"";
console.log(text1); // => She said "hi"

// new line and tab:
console.log("Line 1\nLine 2"); // \n means new line
// => Line 1
// => Line 2

console.log("Name:\tSam"); // \t means tab
// => Name:   Sam

// a real backslash:
console.log("C:\\Users\\Sam"); // => C:\Users\Sam
```

Common escapes:
- `\n` — new line
- `\t` — tab
- `\"` — a double quote
- `\'` — a single quote
- `\\` — a backslash

A simple way to avoid quote-escaping: use the **other** quote type, or use
backticks.

```js
const easy = 'She said "hi"'; // single quotes around double quotes, no escaping
console.log(easy); // => She said "hi"
```

## Common Mistakes

- **Trying to change a letter in place.** Strings are immutable. Build a new
  string instead.
- **Forgetting strings are 0-indexed.** The first letter is at index `0`.
- **Mixing up `slice` and `substring` with negatives.** `slice` handles negative
  numbers; `substring` does not. Prefer `slice`.
- **Using `replace` and expecting all matches.** `replace` changes only the
  first. Use `replaceAll` for every match.
- **Using `+` for long messages.** Template literals (`` `Hi ${name}` ``) are
  cleaner and easier to read.
- **Forgetting that methods return a new string.** You must store the result:
  `text = text.trim()`.

## Exercises

1. Create a string `name = "  Ada Lovelace  "`. Trim it, make it uppercase, and
   print its length. Print the first character using brackets and `charAt`.
2. Take `const email = "user@example.com"`. Use `split("@")` to get the username
   and domain. Then use `endsWith` to check if it ends with `.com`.
3. Use a template literal to build a multi-line invoice that shows a name and a
   total, with the total formatted as `$${price * 1.1}`.
4. Take `const phrase = "the cat sat on the mat"`. Replace all `"the"` with
   `"a"` using `replaceAll`, then `split` the result into an array of words and
   print how many words there are.

## Key Takeaways

- A **string** is text, made with `" "`, `' '`, or backticks `` ` ``.
- Strings are **immutable**: methods return a **new** string; the original never
  changes.
- `length` counts characters; access letters with `str[0]` or `charAt(0)`.
- Useful methods: `toUpperCase`, `toLowerCase`, `trim`, `includes`, `indexOf`,
  `slice`, `replace`/`replaceAll`, `split`, `repeat`, `startsWith`, `endsWith`,
  `padStart`.
- **Template literals** (backticks) let you insert values with `${ }` and write
  multi-line text.
- Use `\` to **escape** special characters like `\n`, `\t`, and quotes.

---

[← Back to Course Index](../README.md) | [Next: Sets and Maps →](05-sets-and-maps.md)
