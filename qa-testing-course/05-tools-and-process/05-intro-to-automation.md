# Intro to Test Automation

You've learned manual testing — the foundation of all QA. At some point you'll hear
about **automation**: making computers run tests for you. This lesson is a *bridge*,
not a coding tutorial. The goal is to understand **what** automation is, **when** it
helps, **when it doesn't**, and how it relates to the manual skills you've built. You
do *not* need to write code to understand these ideas.

## What You'll Learn

- What test automation actually is
- What's worth automating (and what isn't)
- Why manual skills come *first*
- The names of common automation tools (for awareness)
- A sane path if you want to grow into automation later

## What Is Test Automation?

**Test automation** is using software to run your tests automatically instead of a
human doing it by hand. You write instructions once (in code or a low-code tool),
and the computer repeats them — clicking, typing, and checking results — as often as
you like, day or night.

```text
   Manual:     You click through 50 login tests.   (~1 hour, every release)
   Automated:  A script clicks through 50 login     (~2 minutes, runs nightly,
               tests for you.                         never gets bored or tired)
```

## What's Worth Automating

Automation shines for tests that are **repetitive, stable, and run often**:

- ✅ **Regression suites** — the big set of "make sure old stuff still works" tests
  you run every release. (The #1 automation target — recall
  [Module 2](../02-types-of-testing/03-regression-and-retesting.md).)
- ✅ **Smoke tests** — quick "is the build alive?" checks on every new build.
- ✅ **Repetitive, data-heavy tests** — the same flow with 100 different inputs.
- ✅ **Stable features** — things that don't change much, so the script doesn't keep
  breaking.

## What's *Not* Worth Automating

Automation is not a cure-all. Keep these **manual**:

- ❌ **Exploratory testing** — needs human curiosity and judgment; a script can't be
  curious.
- ❌ **Usability** — only a human can tell if something *feels* confusing or ugly.
- ❌ **Brand-new, changing features** — the UI shifts daily; the script breaks daily.
  Automate *after* it stabilizes.
- ❌ **Tests run only once** — writing automation costs time; don't pay it for a
  one-off.

The smart split: **automate the boring, repetitive checks; keep humans on the
thinking-heavy testing.** They complement each other.

## Why Manual Skills Come First

This is the key message of the whole course:

> **You cannot automate a test you don't understand.**

Automation just *executes* a test — you still have to *design* it: know the
requirements, pick the right cases (equivalence partitioning, boundary values),
define expected results, and judge what matters. Those are the **manual skills you
just learned**. A person who can't write a good test case will write bad automated
tests, faster. Manual testing is the foundation; automation is a power tool built on
top.

## The Tools (Just for Awareness)

You don't need to learn these now — just recognize the names:

| Tool | What it automates | Note |
| --- | --- | --- |
| **Selenium** | Web browsers | The classic, widely used |
| **Playwright** | Web browsers | Modern, fast, popular |
| **Cypress** | Web browsers | Developer-friendly |
| **Appium** | Mobile apps | For iOS/Android |
| **Postman/Newman** | APIs | Automate the API checks from the last-but-one lesson |

Most use a programming language (often **JavaScript** or **Python**). Some newer
**low-code/no-code** tools (e.g., Katalon, Testable record-and-playback tools) let
you automate with less coding.

## A Sane Growth Path (If You Want It)

You can have a great career in **manual QA** and never automate. But if you want to
grow into automation later, a gentle path:

1. **Master manual testing first** (you're doing it — this course).
2. **Learn API testing** with Postman (you got an intro — it's low-friction).
3. **Pick up basic programming** — a little JavaScript or Python goes a long way.
4. **Learn one tool** — Playwright or Cypress are great modern starting points.
5. **Automate your own regression suite** on a practice project.

Your repo already has a [JavaScript course](../../javascript-course/) — a natural
next step if you decide to head toward automation.

## The Big Picture

```text
   STRONG MANUAL QA  ──┬──▶  stay manual (a real, valued career)
   (this course)       │
                       └──▶  add automation (manual skills + a tool + some code)
```

Automation is an *addition* to your skills, not a replacement for them. Everything
you learned here remains the foundation.

## Exercises

1. In your own words, what is test automation?
2. Give two types of testing that are *great* to automate and two that should stay
   manual — and explain why.
3. Why is it true that "you can't automate a test you don't understand"?
4. If you wanted to move toward automation in a year, what would your first three
   steps be?

Next: [Module 6 — Mastery](../06-mastery/01-building-a-qa-portfolio.md)
