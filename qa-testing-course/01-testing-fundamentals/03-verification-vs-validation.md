# Verification vs Validation

These two words sound almost identical and are constantly confused — including in
job interviews. Learn the difference once, with a simple memory trick, and you'll
never mix them up again.

## What You'll Learn

- What verification means and what validation means
- A one-line trick to tell them apart forever
- Concrete examples of each
- Why a product can pass one and fail the other

## The Two Questions

| | Verification | Validation |
| --- | --- | --- |
| **The question** | "Are we building the product **right**?" | "Are we building the **right** product?" |
| **Checks against** | Specifications, designs, requirements | Real user needs |
| **How** | Reviews, walkthroughs, inspections (no running the app) | Actual testing — running the software |
| **When** | Earlier (documents, designs, code) | Later (the working product) |
| **Question it answers** | Did we build it according to the plan? | Does the thing we built actually solve the user's problem? |

## The Memory Trick

> **Verification** = "Did we build it **right**?" (right = correctly, per the spec)
> **Validation** = "Did we build the **right thing**?" (right = the thing users
> actually need)

Or shorter: **Verification checks the documents, Validation checks the product.**

## Examples That Make It Click

**Example 1 — A login form**

- *Verification:* Read the design doc. It says the password field should hide
  characters with dots. You review the built form against the doc — yes, it hides
  characters. ✅ Verified.
- *Validation:* You actually log in as a real user would. It works, it's fast, the
  error messages make sense. ✅ Validated.

**Example 2 — Where they disagree (the important case)**

A team is asked to build a tool to help nurses log patient vitals. They build it
*exactly* to the written spec — every field, every button, pixel-perfect.

- **Verification: PASS** — it matches the spec perfectly.
- **Validation: FAIL** — when real nurses try it, it takes 12 taps to log one
  reading, so nobody uses it. The spec itself was wrong.

This is the whole point: **you can build something perfectly to spec and still
build the wrong thing.** Verification alone is not enough. You need real users (or
testers standing in for them) to validate that it solves the actual problem.

## A Tiny Bit of Process Vocabulary

Verification is usually done through **static** techniques (you don't run the
software):

- **Review** — people read a document or design and give feedback.
- **Walkthrough** — the author guides the team through the work.
- **Inspection** — a formal, structured review against a checklist.

Validation is done through **dynamic** techniques — you *run* the software and
observe it. That's most of what this course teaches.

## Exercises

1. Without looking, write the one-line memory trick for each term.
2. Label each as verification or validation:
   a. Reading a requirements document to check it's complete.
   b. Logging into the finished app to confirm it works.
   c. A team reviewing a screen design before any code is written.
   d. A real customer trying a beta and giving feedback.
3. Describe a situation where software passes verification but fails validation.
4. Why is a static review (no running the app) still valuable testing work?

Next: [The Seven Principles of Testing](04-principles-of-testing.md)
