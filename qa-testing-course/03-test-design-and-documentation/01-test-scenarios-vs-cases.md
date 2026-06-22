# Test Scenarios vs Test Cases

When you're handed a feature to test, you don't just start clicking. You break the
work down into **scenarios** (the big "what should I test?" ideas) and then into
**test cases** (the precise step-by-step checks). Beginners confuse these two —
let's make the difference crisp, because almost everything in this module builds on
it.

## What You'll Learn

- What a test scenario is
- What a test case is
- How scenarios break down into cases
- When to use each

## Test Scenario — The "What"

A **test scenario** is a high-level idea of *what* to test — a single line
describing functionality from the user's point of view. It answers *"what should we
verify?"* without spelling out the steps.

> **Scenario:** "Verify that a user can log in with valid credentials."
> **Scenario:** "Verify that login fails with an invalid password."
> **Scenario:** "Verify the user can reset a forgotten password."

Scenarios are quick to write, easy to review, and great for making sure you've
covered the big picture before sweating the details.

## Test Case — The "How"

A **test case** is the detailed, step-by-step version of a scenario. It spells out
*exactly* what to do, what data to use, and what result to expect — so precisely
that **anyone** could run it and get the same result.

A scenario becomes one *or more* test cases. Here's the login scenario expanded:

| Field | Value |
| --- | --- |
| **Test Case ID** | TC-001 |
| **Title** | Login with valid email and password |
| **Preconditions** | User account `bob@example.com` exists and is active |
| **Steps** | 1. Go to the login page. 2. Enter `bob@example.com`. 3. Enter the correct password. 4. Click **Login**. |
| **Test Data** | `bob@example.com` / `Correct123!` |
| **Expected Result** | Dashboard opens; "Welcome, Bob" is shown |

(We'll fully break down how to write cases like this in the
[next lesson](02-writing-test-cases.md).)

## One Scenario → Many Cases

A single scenario usually spawns several test cases — one per situation worth
checking.

```text
   Scenario: "Verify user login"
        │
        ├── TC-001: valid email + valid password        → success
        ├── TC-002: valid email + wrong password         → error
        ├── TC-003: unregistered email                   → error
        ├── TC-004: empty fields                         → validation prompt
        └── TC-005: badly formatted email                → validation message
```

The scenario is the *theme*; the cases are the *specific experiments*.

## Comparison

| | Test Scenario | Test Case |
| --- | --- | --- |
| **Detail** | High-level, one line | Detailed, step-by-step |
| **Answers** | *What* to test | *How* to test it, exactly |
| **Includes steps?** | No | Yes |
| **Includes test data?** | No | Yes |
| **Speed to write** | Fast | Slower |
| **Best for** | Coverage at a glance, planning | Execution, repeatability |

## When to Use Each

- **Scenarios first** — brainstorm coverage quickly, get the list reviewed, make
  sure nothing big is missing.
- **Cases next** — turn the agreed scenarios into precise, runnable steps.
- **Tight on time?** Some Agile teams work mostly from scenarios plus exploratory
  testing, writing full cases only for critical or complex flows.

Think of it like travel: the **scenario** is "visit the museum, the park, and the
old town." The **test cases** are the turn-by-turn directions to each one.

## Exercises

1. Define test scenario and test case in one sentence each.
2. Write **four** test scenarios for a "search products" feature.
3. Pick one of your scenarios and expand it into a detailed test case (ID, title,
   preconditions, steps, data, expected result).
4. Why might a fast-moving Agile team rely more on scenarios than fully written
   cases?

Next: [Writing Good Test Cases](02-writing-test-cases.md)
