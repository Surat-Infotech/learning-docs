# Building a QA Portfolio

You've learned the craft — now you need to *prove* it. Employers hiring a junior
tester don't expect years of experience, but they do want evidence you can actually
do the work: write clear test cases, find real bugs, and report them well. A small
**portfolio** does exactly that, and it makes you stand out from applicants who only
have a résumé. The good news: you can build one with no job and no permission.

## What You'll Learn

- Why a portfolio beats a résumé bullet for a tester
- What to put in a QA portfolio
- How to create artifacts using *real, public* apps
- How to present it

## Why a Portfolio?

Anyone can write "good attention to detail" on a CV. A portfolio *shows* it. When a
hiring manager sees a crisp bug report and a well-structured test plan you actually
wrote, the conversation changes from "can you do this?" to "when can you start?" It's
the closest thing to on-the-job evidence you can produce before getting the job.

## What to Include

Aim for a handful of polished artifacts, not a mountain of mediocre ones:

1. **A test plan** for a real app — scope, approach, risks (from
   [Module 3](../03-test-design-and-documentation/04-test-plans.md)).
2. **A set of test cases** for one feature — well-structured, with the standard
   fields.
3. **Real bug reports** — bugs you actually found in real apps, written to the
   [template](../04-defect-management/02-writing-bug-reports.md) with screenshots.
4. **An RTM** — a small traceability matrix linking requirements to your test cases.
5. **An exploratory testing charter + notes** from a session you ran.
6. **(Bonus) API testing** — a few Postman checks with notes on what you verified.
7. **(Bonus) Cross-browser/responsive findings** — screenshots of a layout bug
   across browsers or screen sizes.

Quality over quantity. Three excellent, real artifacts beat twenty sloppy ones.

## The Secret: Test Real, Public Apps

You don't need a company to give you software. The world is full of apps with bugs:

- Pick a **public website or app** you use (a shop, a to-do app, a booking site).
- **Treat it as your assignment.** Write scenarios, then test cases, then go
  hunting.
- **Find real bugs** — they're everywhere: broken layouts on mobile, bad error
  messages, validation that lets through garbage, slow pages.
- **Document everything** exactly as you would on the job.

Some teams even publish *intentionally buggy* practice apps designed for testers to
hone skills on — search for "practice websites for QA testing." They're perfect for
portfolio material.

> ⚠️ **Ethics note:** Functional and usability testing of a public site as a *normal
> user* is fine. Do **not** attempt security attacks, load/stress testing, or
> anything that could harm a service you don't own or have permission to test. Keep
> it to the kind of testing a regular user could do.

## How to Present It

- **A simple doc or PDF** per artifact is perfectly fine to start.
- **GitHub** is even better — and your learning repo already lives there. A
  `qa-portfolio` folder with Markdown files shows you can use the tools developers
  use. (You took the [Git course](../../git-course/) — use it!)
- **A short intro page:** who you are, what's inside, and a one-line summary of each
  artifact.
- **Screenshots matter** — visual evidence of bugs is compelling.

## A Mini Plan to Build It This Month

```text
   Week 1: Pick an app. Write a 1-page test plan + 10 test cases for one feature.
   Week 2: Execute the cases. Find and document 5 real bugs with screenshots.
   Week 3: Run an exploratory session (charter + notes). Build a small RTM.
   Week 4: Polish everything, put it on GitHub, write the intro page.
```

That's a genuine, employer-credible portfolio in four weeks of part-time effort.

## Exercises

1. Why does a portfolio persuade an employer more than a résumé bullet?
2. List five artifacts you'll put in your QA portfolio.
3. Choose a real public app right now. Write three test scenarios and find one real
   bug, documented with the bug-report template.
4. Set up a `qa-portfolio` folder (in a Git repo) and add your first artifact.

Next: [QA Interview Preparation](02-interview-preparation.md)
