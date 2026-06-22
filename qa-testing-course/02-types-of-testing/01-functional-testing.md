# Functional Testing

Functional testing answers the most basic question about software: **"Does it do
what it's supposed to do?"** If a button says "Save," does it save? If you enter a
wrong password, do you get blocked? This is the bread and butter of manual QA — most
of your day-to-day testing is functional testing.

## What You'll Learn

- What functional testing is (and isn't)
- How to turn a requirement into functional tests
- The difference between functional and non-functional testing
- A worked example you can copy

## What Is Functional Testing?

Functional testing checks **features and behavior against requirements**. You give
the software an input, perform an action, and check the output matches what was
expected.

```text
   Input  →  Action  →  Expected Output
   "wrong password" → click Login → "error: incorrect password" shown,
                                     user NOT logged in
```

You don't care *how* the code works inside — only that the **behavior** is correct.
(That outside-only view is called **black box testing**, covered in
[a later lesson](06-black-grey-white-box.md).)

## Functional vs Non-Functional (the big split)

Every type of testing falls into one of two families:

| | Functional | Non-Functional |
| --- | --- | --- |
| **Question** | *What* does it do? | *How well* does it do it? |
| **Examples** | Login works, search returns results, payment processes | Speed, security, usability, works on mobile |
| **Example check** | "Clicking Buy creates an order" | "The order page loads in under 2 seconds" |

This lesson covers functional. The next one covers non-functional.

## From Requirement to Test

The skill is turning a plain requirement into concrete checks. Take this
requirement:

> **R1:** Users log in with an email and password. Correct credentials go to the
> dashboard. Wrong credentials show an error.

Functional tests that flow from it:

| # | What you do | Expected result |
| --- | --- | --- |
| 1 | Enter correct email + correct password, click Login | Dashboard opens |
| 2 | Enter correct email + wrong password | Error message; stays on login |
| 3 | Enter unregistered email | Error message; stays on login |
| 4 | Leave both fields blank, click Login | Validation prompts to fill fields |
| 5 | Enter email in the wrong format (`bob@`) | "Please enter a valid email" |

Notice test 1 is the **happy path**; tests 2–5 are the **unhappy paths**. A good
functional test set covers both. (Remember the
[tester's mindset](../00-getting-started/02-the-testers-mindset.md): go off the
happy path.)

## Common Things Functional Testing Checks

- **User interface** — buttons, links, and menus do what their labels say.
- **Inputs and forms** — valid input accepted, invalid input rejected with helpful
  messages.
- **Business rules** — e.g., "orders over $50 get free shipping" actually applies.
- **Workflows** — multi-step journeys (sign up → verify email → log in) work end to
  end.
- **Data** — what you save is what you see later; deletes really delete.
- **Permissions** — a regular user can't reach admin-only pages.

## A Word on Coverage

You can't test every input ([principle 2](../01-testing-fundamentals/04-principles-of-testing.md)),
so functional testing leans on **design techniques** to pick the most valuable
cases — equivalence partitioning, boundary values, and more. Those get a full
lesson in
[Module 3](../03-test-design-and-documentation/03-test-design-techniques.md). For
now, just practice translating requirements into clear input → action → expected
result checks.

## Exercises

1. In one sentence each, define functional and non-functional testing.
2. Requirement: *"Users can search products by name. Matching products are shown;
   if none match, a 'No results' message appears."* Write **five** functional
   tests as an input → action → expected-result table.
3. For a "Reset Password" feature, list three happy-path and three unhappy-path
   tests.
4. Classify each as functional or non-functional: (a) the page loads in 1 second,
   (b) clicking Delete removes the item, (c) the app works on an iPhone, (d) a
   coupon code reduces the price correctly.

Next: [Non-Functional Testing](02-non-functional-testing.md)
