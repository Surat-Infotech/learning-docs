# Clean Code and Naming

Code is read far more often than it is written. You (and others) will come back to your code weeks later and need to understand it fast. **Clean code** means writing code that is easy to read and understand. It's a skill that makes you a much better developer. This lesson is full of small before-and-after examples.

## What You'll Learn

- Why readable code matters
- How to choose good names
- Why functions should be small and do one thing
- How to avoid magic numbers
- How to avoid deep nesting with early returns
- The DRY rule (don't repeat yourself)
- How to write helpful comments
- Why consistent formatting matters (Prettier and ESLint)

## Why Readable Code Matters

Computers can run messy code just fine. But *humans* have to read and change it. Clean code:

- Is easier to understand and fix.
- Has fewer bugs, because problems are easier to spot.
- Is friendlier for teammates (and future you).

Writing clean code is a kindness to the person who reads it next, which is usually you.

## Good Naming

Names are one of the most important parts of clean code. A good name tells you exactly what something is.

**Rules for good names:**

- Be **clear and descriptive**.
- Use **camelCase** for variables and functions (first word lowercase, then each new word capitalized, like `userAge`).
- Avoid **single letters**, except for short loop counters like `i`.

```js
// Bad: what do these even mean?
let d = 5;
let x = "Ava";

// Good: the names explain themselves
let daysLeft = 5;
let firstName = "Ava";
```

A good name removes the need for a comment. When you read `daysLeft`, you instantly know what it holds.

## Functions Should Do One Thing

A function should do **one job** and have a name that describes that job. Small, focused functions are easier to read, test, and reuse.

```js
// Bad: this function does too many different things
function handleUser(user) {
  console.log("Saving user");
  // ...save to database...
  console.log("Sending email");
  // ...send welcome email...
  console.log("Logging activity");
  // ...write to log...
}

// Good: split into small, clear functions
function saveUser(user) {
  // ...save to database...
}

function sendWelcomeEmail(user) {
  // ...send email...
}

function logActivity(user) {
  // ...write to log...
}
```

If you find yourself describing a function with the word "and" (it saves the user *and* sends an email *and* logs activity), that's a sign to split it up.

## Avoid Magic Numbers

A **magic number** is a number in your code with no explanation. Readers can't tell what it means or why it's there. Replace it with a named **constant** (a value with a clear name that doesn't change).

```js
// Bad: what is 0.08? What is 18?
function getTotal(price) {
  return price + price * 0.08;
}

function canVote(age) {
  return age >= 18;
}

// Good: named constants explain the meaning
const TAX_RATE = 0.08;
const VOTING_AGE = 18;

function getTotal(price) {
  return price + price * TAX_RATE;
}

function canVote(age) {
  return age >= VOTING_AGE;
}
```

Now the code reads almost like English, and if the tax rate changes, you update it in one place.

## Avoid Deep Nesting with Early Returns

Deeply nested code (lots of `if` blocks inside each other) is hard to follow. An **early return** means you check for problems first and exit the function early, which keeps the main logic flat and clear.

```js
// Bad: deeply nested and hard to read
function getDiscount(user) {
  if (user) {
    if (user.isActive) {
      if (user.isMember) {
        return 20;
      }
    }
  }
  return 0;
}

// Good: handle the exits first, then the main path
function getDiscount(user) {
  if (!user) return 0;
  if (!user.isActive) return 0;
  if (!user.isMember) return 0;

  return 20;
}
```

The second version is much easier to read top to bottom. Each check stands alone.

## DRY: Don't Repeat Yourself

**DRY** stands for "Don't Repeat Yourself." If you copy and paste the same code in many places, fixing a bug means fixing it everywhere. Instead, put repeated code into one function and reuse it.

```js
// Bad: the same greeting logic repeated
console.log("Hello, Ava! Welcome back.");
console.log("Hello, Ben! Welcome back.");
console.log("Hello, Cara! Welcome back.");

// Good: write it once, reuse it
function greet(name) {
  console.log(`Hello, ${name}! Welcome back.`);
}

greet("Ava");
greet("Ben");
greet("Cara");
```

Now if you want to change the greeting, you change it in one place.

## Comments: Explain Why, Not What

Good code explains *what* it does through clear names. So comments should usually explain *why* you did something, not repeat what the code already shows.

```js
// Bad: this comment just repeats the code
// add 1 to count
count = count + 1;

// Good: this explains WHY, which the code can't show
// We start pages at 1, not 0, because users count from 1
let pageNumber = 1;
```

Use comments for surprising decisions, tricky workarounds, or important reasons. Don't use them to describe obvious code.

## Consistent Formatting

**Formatting** is how your code is spaced and laid out: indentation, line breaks, quotes, and so on. Consistent formatting makes code easier to scan. The good news: tools do this for you automatically.

- **Prettier** automatically formats your code so it looks neat and consistent.
- **ESLint** checks your code for common mistakes and bad patterns, and can warn you about them.

You don't have to format by hand or argue about style. Set up these tools once, and they keep your whole project tidy.

## Common Mistakes

- **Short, vague names** like `x`, `temp`, or `data2`. Be descriptive.
- **Giant functions** that do many things. Split them up.
- **Magic numbers** scattered around. Use named constants.
- **Deep nesting** that's hard to follow. Use early returns.
- **Copy-pasting code.** Reuse with functions instead.
- **Comments that repeat the code** instead of explaining the reason.

## Exercises

1. Take this line: `let n = 7;` and rename the variable so its meaning is clear (decide what it represents).
2. Find a magic number in your own code (or invent one) and replace it with a named constant.
3. Rewrite a nested `if` block using early returns to make it flat and readable.
4. Spot some repeated code in a small program and pull it into one reusable function.

## Key Takeaways

- Clean code is **easy for humans to read**, which means fewer bugs.
- Use **clear, descriptive names** in camelCase; avoid single letters.
- Functions should **do one thing** and stay small.
- Replace **magic numbers** with named constants.
- Use **early returns** to avoid deep nesting.
- Follow **DRY**: write repeated logic once and reuse it.
- Comments should explain **why**, not what.
- Use **Prettier** and **ESLint** to keep formatting consistent automatically.

---

[← Back to Testing Basics](./05-testing.md) | [Back to Course Index](../README.md) | [Next: A Taste of TypeScript →](./07-typescript-awareness.md)
