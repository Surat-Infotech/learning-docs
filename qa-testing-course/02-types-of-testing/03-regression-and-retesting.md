# Regression Testing and Retesting

Software is never "finished." Developers keep adding features and fixing bugs. Every
change risks **breaking something that used to work**. Two kinds of testing guard
against this: **retesting** (did the fix work?) and **regression testing** (did the
fix break anything else?). Beginners mix these up — let's nail the difference.

## What You'll Learn

- What retesting is and when you do it
- What regression testing is and why it's unavoidable
- The clear difference between the two
- How teams keep regression manageable

## Retesting — "Is the bug actually fixed?"

When you report a bug and a developer fixes it, you **retest**: run the *same* test
that failed before, on the *new* build, to confirm it now passes.

- **Goal:** confirm the specific fix worked.
- **Uses:** the exact steps from the original bug report.
- **Always** done after a bug is marked fixed.

> You reported "clicking Save shows an error." Dev says it's fixed. You click Save
> again on the new build → it saves correctly. ✅ Retest passed.

## Regression Testing — "Did the change break anything else?"

Code is interconnected. Fixing or adding one thing can accidentally break a
*different*, unrelated thing. **Regression testing** re-runs tests on existing,
previously-working features to make sure they *still* work after a change.

- **Goal:** confirm nothing that used to work is now broken.
- **Uses:** existing test cases for features *around* the change (and often the
  whole app).
- Done after *any* change: new feature, bug fix, configuration change.

> A dev fixes the Save button. You also re-check the Edit, Delete, and Export
> buttons nearby — because the fix touched shared code. That's regression testing.

The word "regression" means *going backward* — software that *regresses* has slipped
back to a broken state.

## The Key Difference

| | Retesting | Regression Testing |
| --- | --- | --- |
| **Question** | Is *this* bug fixed? | Did the change break *anything else*? |
| **Scope** | The one fixed defect | Existing features around (or across) the app |
| **Test data** | Same as the failed test | Existing/previous test cases |
| **When** | After a specific fix | After any change |
| **Finds** | Whether the fix worked | New, unexpected breakages |

A simple way to remember: **retest the fix, regress the rest.**

## Why Regression Is a Big Deal

Regression testing is one of the **largest, most repetitive** jobs in QA. Every
release, you may re-run hundreds of old tests "just to be sure." This is exactly
why:

- It's tedious and error-prone to do entirely by hand every time.
- It's the **#1 candidate for automation** — computers happily re-run 500 tests
  every night without getting bored. (More on this in
  [Module 5](../05-tools-and-process/05-intro-to-automation.md).)
- Smart teams keep a **regression suite**: a curated set of important tests run
  before every release.

## How Teams Keep It Manageable

You can't re-run *everything* forever, so teams prioritize:

- **Risk-based:** test the most critical and most-used features first (checkout,
  login).
- **Change-based:** focus on areas near the code that changed (remember
  [defect clustering](../01-testing-fundamentals/04-principles-of-testing.md)).
- **Smoke first:** a quick "are the basics alive?" check before deeper regression
  (next lesson).
- **Automate the boring parts:** let tools handle the repetitive re-runs.

## Exercises

1. Define retesting and regression testing in one sentence each.
2. A developer fixes a bug in the shopping cart. List: (a) the retest you'd run,
   (b) three regression checks around it.
3. Why is regression testing the most common candidate for automation?
4. Your release has 600 test cases but only two hours to test. Which do you run
   first, and how do you decide?

Next: [Smoke and Sanity Testing](04-smoke-and-sanity.md)
