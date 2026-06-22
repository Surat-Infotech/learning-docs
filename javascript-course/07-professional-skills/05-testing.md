# Testing Basics

How do you know your code actually works? You could click around and check by hand every time. But that gets slow and easy to forget. **Testing** means writing code that checks your code automatically. It helps you catch bugs early and change your code without fear. This lesson keeps it gentle and practical.

## What You'll Learn

- Why testing is worth your time
- The difference between manual and automated testing
- What a unit test is
- What a test runner does (Jest and Vitest)
- The AAA pattern: Arrange, Act, Assert
- How to write a simple function and a test for it
- The idea of test-driven development
- What is worth testing

## Why Test?

Tests give you two big gifts:

1. **Catch bugs early.** A test can find a mistake before your users do.
2. **Change code safely.** When you improve your code later, tests tell you instantly if you broke something. This is called confidence.

Without tests, every change is a gamble. With tests, you can improve your code knowing a safety net is watching your back.

## Manual vs Automated Testing

There are two ways to test:

- **Manual testing** — you run the app yourself and click around to check things. This is fine for a quick look, but it's slow and you might forget steps.
- **Automated testing** — you write code that checks your code. You run it with one command, and it tests everything in seconds, every time.

Manual testing is like proofreading an essay yourself. Automated testing is like having a spell-checker that runs instantly. You still do some manual testing, but automated tests save huge amounts of time.

## What Is a Unit Test?

A **unit test** checks one small piece of your code on its own, usually a single function. "Unit" means a small, independent part.

For example, if you have a function that adds two numbers, a unit test checks: "When I give it 2 and 3, does it return 5?" That's it. Small and focused.

Unit tests are the most common type of test and the best place to start.

## What Is a Test Runner?

A **test runner** is a tool that finds your test files, runs them, and tells you which passed and which failed. You don't have to build this yourself.

Two popular test runners for JavaScript are:

- **Jest** — very common, made by the team behind React.
- **Vitest** — newer and fast, popular with modern projects.

They work in very similar ways, so learning one helps you understand the other. You install them with npm (which you learned about earlier).

## The AAA Pattern

Good tests follow a simple three-step shape called **AAA**:

1. **Arrange** — set up the data you need.
2. **Act** — call the function you're testing.
3. **Assert** — check that the result is what you expected.

This pattern keeps every test clear and easy to read. Set up, run, check.

## Writing a Function and a Test

Let's write a simple **pure function** and test it. A pure function always returns the same result for the same input and doesn't change anything outside itself. Pure functions are the easiest to test.

Here is our function:

```js
// add.js
// Takes two numbers and returns their sum
export function add(a, b) {
  return a + b;
}
```

Now here is a test for it. The exact words `test`, `expect`, and `toBe` come from the test runner.

```js
// add.test.js
import { add } from "./add.js";

// test(description, function that does the checking)
test("add returns the sum of two numbers", () => {
  // Arrange: set up our inputs
  const a = 2;
  const b = 3;

  // Act: run the function
  const result = add(a, b);

  // Assert: check the result is what we expect
  expect(result).toBe(5);
});
```

Let's read the assert line: `expect(result).toBe(5)` means "I expect `result` to be `5`." If it is, the test passes. If not, the test fails and shows you what went wrong.

You would run this with a command like:

```bash
# Run all your tests
npm test
```

The runner then prints something like "1 test passed." If a test fails, it shows the expected value versus the actual value, helping you fix the bug.

## Test-Driven Development (Briefly)

**Test-driven development** (TDD) is a style where you write the *test first*, before the actual code. The cycle is:

1. Write a test for what you want (it fails, because the code doesn't exist yet).
2. Write just enough code to make the test pass.
3. Clean up the code, keeping the test green.

TDD feels strange at first, but it forces you to think clearly about what your code should do. You don't have to use it now, but it's good to know the idea exists.

## What to Test

You don't need to test everything. Focus your energy on:

- **Important logic.** Calculations, rules, and anything with steps that could go wrong.
- **Edge cases.** What happens with zero, empty values, or very large numbers?
- **Bugs you've fixed.** Write a test so the same bug can't come back.

You usually don't need to test simple things like a one-line getter, or code from other libraries (they have their own tests). Start small. Even a few tests are far better than none.

## Common Mistakes

- **Testing too much at once.** Keep each test focused on one thing.
- **Writing tests that depend on each other.** Each test should stand alone.
- **Skipping the Assert step.** A test with no `expect` checks nothing.
- **Only testing the "happy path."** Also test empty and unusual inputs.
- **Giving up because it feels slow at first.** It gets fast quickly and saves time later.

## Exercises

1. Write a pure function `multiply(a, b)` that returns the product of two numbers.
2. Write a test for `multiply` using the AAA pattern and `expect(...).toBe(...)`.
3. Write a function `isEven(n)` that returns `true` if a number is even. Write two tests: one for an even number, one for an odd number.
4. Try TDD: write a failing test for a `reverseString` function first, then write the function to make it pass.

## Key Takeaways

- Tests **catch bugs early** and let you **change code safely**.
- **Manual** testing is by hand; **automated** testing is by code and much faster.
- A **unit test** checks one small piece of code, usually a function.
- A **test runner** like Jest or Vitest finds and runs your tests.
- Follow the **AAA pattern**: Arrange, Act, Assert.
- Use `expect(result).toBe(value)` to check outcomes.
- **TDD** means writing the test before the code.
- Test important logic and edge cases first; you don't need to test everything.

---

[← Back to LocalStorage](./04-localstorage.md) | [Back to Course Index](../README.md) | [Next: Clean Code and Naming →](./06-clean-code.md)
