# Final Project: Test a Real Application End to End

This is where everything comes together. You'll take a real application and put it
through a **complete, professional QA process** — from understanding requirements to
a final test summary report. Do this well and you'll have both **proof you can do
the job** and the centerpiece of your [portfolio](01-building-a-qa-portfolio.md).

## The Goal

Run a full manual QA cycle on a real app, producing every key artifact a working
tester creates. By the end you'll have a folder of documents that demonstrates the
entire skill set from this course.

## Choose Your Application

Pick **one** app to test thoroughly. Good choices:

- A **public web app** you use (a to-do app, a shop, a booking site, a note app).
- A **practice testing site** built for QA learners (search "practice website for QA
  testing" — several exist with deliberate bugs).
- A **small open-source app** you can run.

> ⚠️ Keep it to testing a normal user could do (functional, usability,
> compatibility). No security attacks or load testing on services you don't own.

Pick something with real features — login, forms, a multi-step flow like checkout or
booking. You need enough surface area to test.

## The Deliverables

Produce these artifacts, in order. Each maps to a part of the course.

### 1. Test Plan (Module 3)

A one-to-two page plan:

- **Objective** — what you're testing and why.
- **Scope** — what's in and what's out (be explicit about out-of-scope).
- **Approach** — which test types (functional, usability, compatibility...).
- **Environment** — browsers/devices you'll use.
- **Risks** — what could limit your testing.

### 2. Requirements List + Test Scenarios (Modules 1 & 3)

- Write down the **requirements** as you understand them (infer them from the app if
  none are given — a real skill).
- Turn them into **test scenarios** (high-level "what to test").

### 3. Test Cases (Module 3)

- Expand your scenarios into **detailed test cases** with the standard fields (ID,
  title, preconditions, steps, data, expected result).
- Aim for **15–25 cases** covering happy paths *and* unhappy paths.
- Use the **design techniques** (equivalence partitioning, boundary values, a
  decision table for any rule-based feature).

### 4. Requirements Traceability Matrix (Module 3)

- A table linking each requirement to its test case IDs. Prove full coverage — and
  spot any gaps.

### 5. Test Execution + Bug Reports (Modules 2 & 4)

- **Run** your test cases. Mark each **Pass / Fail / Blocked**.
- For every failure, write a complete **bug report** (template from
  [Module 4](../04-defect-management/02-writing-bug-reports.md)) with steps, expected
  vs actual, environment, screenshots, and **severity + priority**.
- Run at least one **exploratory session** (charter + notes) and log anything it
  surfaces.

### 6. Cross-Browser / Responsive Check (Module 5)

- Test key pages in at least two browsers and at a phone screen size. Document any
  compatibility/layout bugs with screenshots.

### 7. (Bonus) API Check (Module 5)

- If the app has an accessible API, run a few **Postman** checks (valid + invalid
  requests) and note the results and status codes.

### 8. Test Summary Report (Module 1 — STLC closure)

A one-page wrap-up:

- What you tested (scope) and what you didn't.
- Numbers: cases run, passed, failed, blocked.
- Bugs found, grouped by severity.
- Your **overall assessment**: would you recommend shipping? Why or why not?
- Lessons learned.

## Suggested Timeline

```text
   Day 1-2:  Test plan + requirements + scenarios.
   Day 3-4:  Write test cases + RTM.
   Day 5-6:  Execute cases; log bugs with screenshots.
   Day 7:    Exploratory session + cross-browser/responsive checks.
   Day 8:    Test summary report; polish and publish to your portfolio (Git).
```

## Definition of Done for This Project

- [ ] Test plan written, with explicit scope.
- [ ] Requirements captured and turned into scenarios.
- [ ] 15–25 detailed test cases using design techniques.
- [ ] RTM linking requirements to cases (no untested requirement).
- [ ] All cases executed and marked Pass/Fail/Blocked.
- [ ] Every failure has a complete, reproducible bug report with severity + priority.
- [ ] At least one exploratory session documented.
- [ ] Cross-browser + responsive checks done with screenshots.
- [ ] Test summary report with a clear ship / don't-ship recommendation.
- [ ] Everything organized in a `qa-portfolio` folder, ideally on GitHub.

## What You'll Have Achieved

When you finish, you will have personally executed the **entire manual QA life
cycle** — the same process used on real teams — and have the documents to prove it.
That is exactly what gets a junior tester hired.

## Where to Go Next

- Polish this into your [portfolio](01-building-a-qa-portfolio.md).
- Prep with the [interview lesson](02-interview-preparation.md).
- Consider the next step on your chosen
  [career path](03-career-paths-and-certifications.md) — perhaps API testing, then
  automation via the [JavaScript course](../../javascript-course/).

Congratulations — you've completed the QA Testing course. Now go break some
software. 🐛🔨

Back to: [Course Home](../README.md)
