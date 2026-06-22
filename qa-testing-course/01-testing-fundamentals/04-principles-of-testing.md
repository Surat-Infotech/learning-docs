# The Seven Principles of Testing

Over decades, the testing community boiled hard-won lessons down to **seven
principles**. They're famous (they appear on the ISTQB certification) and, more
importantly, they're *true* — they'll save you from rookie mistakes. Learn what
each one really means, not just the slogan.

## What You'll Learn

- The seven classic testing principles
- A plain-English meaning and a consequence for each
- Why "test everything" is impossible — and what to do instead

## 1. Testing Shows the Presence of Defects, Not Their Absence

Testing can prove bugs **exist**. It can never prove there are **none**. If you run
100 tests and all pass, that doesn't mean the software is bug-free — it means
*those 100 tests* didn't find anything.

> **Consequence:** Never say "I tested it, so it's bug-free." Say "the tests I ran
> passed." Stay humble.

## 2. Exhaustive Testing Is Impossible

You cannot test every possible input, combination, and path. A form with 10 fields
has billions of input combinations. Testing them all would take longer than the
universe has existed.

> **Consequence:** You must **prioritize** and use smart **techniques** (Module 3)
> to test the *right* things, not *everything*.

## 3. Early Testing Saves Time and Money

Start testing activities as early as possible — even at the requirements stage.
The earlier a defect is found, the cheaper it is to fix. (This is "shift left.")

> **Consequence:** Don't wait for finished code. Review requirements and designs
> now.

## 4. Defects Cluster Together

Bugs are not spread evenly. A small number of modules usually contain most of the
defects. (Often roughly 80% of bugs come from 20% of the code — the Pareto
principle.)

> **Consequence:** When you find a buggy area, **test it harder** — there are
> probably more bugs nearby.

## 5. Beware the Pesticide Paradox

If you run the *same* tests over and over, they stop finding new bugs — just like
using the same pesticide stops killing evolved bugs. The tests still pass, but new
problems slip by.

> **Consequence:** **Review and refresh** your tests regularly. Add new cases,
> vary your data, do some exploratory testing.

## 6. Testing Is Context-Dependent

You don't test a banking app the way you test a game. A medical device demands far
more rigor than a marketing landing page. The context decides how much and what
kind of testing.

> **Consequence:** Adapt your approach to the risk and domain. There's no single
> "correct" amount of testing.

## 7. Absence-of-Errors Is a Fallacy

Software can be 100% bug-free against the spec and **still be useless** if it
doesn't meet users' real needs. Finding and fixing lots of bugs doesn't help if you
built the wrong thing. (This connects directly to
[validation](03-verification-vs-validation.md).)

> **Consequence:** A working, bug-free product that nobody wants is still a
> failure. Test against *user needs*, not just the spec.

## Quick Reference

| # | Principle | One-liner |
| --- | --- | --- |
| 1 | Presence, not absence | Tests find bugs; they can't prove there are none. |
| 2 | Exhaustive testing is impossible | You can't test everything — prioritize. |
| 3 | Early testing | Find bugs early; they're cheaper. |
| 4 | Defect clustering | Bugs gather in a few spots — test those harder. |
| 5 | Pesticide paradox | Same tests stop finding bugs — refresh them. |
| 6 | Context-dependent | Test according to risk and domain. |
| 7 | Absence-of-errors fallacy | Bug-free but wrong product is still a failure. |

## Exercises

1. From memory, list all seven principles.
2. A teammate says, "All tests passed, so the app has no bugs." Which principle do
   they misunderstand, and what should they say instead?
3. You found four bugs in the checkout module and none in the help page. Which
   principle tells you where to focus next, and what do you do?
4. Give your own example of principle 7 (bug-free but wrong).

Next: [The Levels of Testing](05-levels-of-testing.md)
