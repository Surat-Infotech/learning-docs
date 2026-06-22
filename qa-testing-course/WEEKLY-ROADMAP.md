# QA Testing Course — 4-Week Intensive Roadmap (~20 hours/week)

This roadmap turns the [QA Testing Course](README.md) into a fast, **4-week
intensive** — a bootcamp-style sprint for finishing the whole course in a month.

- **Time budget:** about 20 hours per week (a near-full-time commitment)
- **Total length:** 4 weeks (~80 hours)
- **Split:** About **30% reading** and **70% hands-on** — at this pace you read a
  module and immediately practice it on a real app the same day.

> **How to read this plan.** Each week covers a big chunk of the course plus a
> hands-on project that ties it together. Pick **one real app** at the start (a shop,
> a to-do app, a booking site) and test it all month — by Week 4 it becomes your
> capstone. Save every artifact in a `qa-portfolio` folder; it's your job-hunting
> portfolio by the end.

**Weekly rhythm (suggested):** ~4 hours a day, 5 days a week (or spread across 6–7
lighter days). Read in the morning, practice in the afternoon — the intensity is the
point: concepts stick when you apply them immediately.

> ⚠️ **This is an aggressive pace.** If 20 hours/week isn't realistic, use the gentler
> plans instead — the same content also maps to a 6-week (~10 hrs/week) or 4-month
> (~5 hrs/week) schedule.

---

## Week-at-a-Glance

| Week | Theme                              | Modules            | Milestone                                   |
| ---- | ---------------------------------- | ------------------ | ------------------------------------------- |
| 1    | Foundations + Types of Testing     | Module 0, 1, 2     | Tester's mindset + first bug log            |
| 2    | Test Design & Documentation        | Module 3           | A full test suite + test plan + RTM         |
| 3    | Defects, Tools & Process           | Module 4, 5        | Bug-report pack in a real tracker           |
| 4    | Mastery + Final Project            | Module 6           | End-to-end QA capstone + portfolio          |

---

## Week 1 — Foundations and Types of Testing

**Goal:** Absorb the mindset, the core vocabulary, and every major kind of testing —
then start finding real bugs immediately.

**Study (read across the week):**

- Module 0 — [What Is QA](00-getting-started/01-what-is-qa.md),
  [The Tester's Mindset](00-getting-started/02-the-testers-mindset.md)
- Module 1 — [SDLC](01-testing-fundamentals/01-sdlc-and-where-qa-fits.md),
  [STLC](01-testing-fundamentals/02-stlc.md),
  [Verification vs Validation](01-testing-fundamentals/03-verification-vs-validation.md),
  [The Seven Principles](01-testing-fundamentals/04-principles-of-testing.md),
  [Levels of Testing](01-testing-fundamentals/05-levels-of-testing.md)
- Module 2 — all six lessons (functional, non-functional, regression/retesting,
  smoke/sanity, exploratory/ad-hoc, black/grey/white box)

**Practice / Project:**

- Build a one-page **cheat sheet**: QA vs QC, verification vs validation, the seven
  principles, the STLC phases, the testing levels.
- Choose your app for the month. Run a **smoke test** of its critical paths, then a
  **functional test** of one feature (input → action → expected result).
- Run two 30-minute **exploratory sessions** with written charters.
- Start a **bug log** spreadsheet of everything you find.

**By the end of the week you can:** explain core QA concepts and perform the main
testing types — and you have a running bug log.

---

## Week 2 — Test Design and Documentation

**Goal:** Learn to design and document tests properly, and produce your first
professional test suite.

**Study:**

- Module 3 — [Scenarios vs Cases](03-test-design-and-documentation/01-test-scenarios-vs-cases.md),
  [Writing Good Test Cases](03-test-design-and-documentation/02-writing-test-cases.md),
  [Design Techniques](03-test-design-and-documentation/03-test-design-techniques.md),
  [Test Plans](03-test-design-and-documentation/04-test-plans.md),
  [Traceability Matrix](03-test-design-and-documentation/05-traceability-matrix.md)

**Practice / Project:**

- Write **4–6 test scenarios** for your app, then expand them into **20–25 detailed
  test cases** (with the standard fields).
- Apply the **design techniques**: equivalence partitioning + boundary values on an
  input field, and a **decision table** for a rule-based feature.
- Draft a one-page **test plan** (objective, scope in/out, approach, risks).
- Build a **Requirements Traceability Matrix** linking requirements to cases —
  confirm nothing is untested.

**By the end of the week you can:** turn requirements into a well-designed, traceable
test suite — and you have a test plan + RTM for your portfolio.

---

## Week 3 — Defects, Tools, and Process

**Goal:** Report and track bugs like a professional, and get hands-on with the tools
and processes used in real QA jobs.

**Study:**

- Module 4 — [Defect Life Cycle](04-defect-management/01-defect-life-cycle.md),
  [Bug Reports](04-defect-management/02-writing-bug-reports.md),
  [Severity vs Priority](04-defect-management/03-severity-vs-priority.md),
  [Working with Developers](04-defect-management/04-working-with-developers.md)
- Module 5 — [Test Management Tools](05-tools-and-process/01-test-management-tools.md),
  [Agile/Scrum](05-tools-and-process/02-agile-testing-and-scrum.md),
  [API Testing Basics](05-tools-and-process/03-api-testing-basics.md),
  [Cross-Browser & Mobile](05-tools-and-process/04-cross-browser-and-mobile.md),
  [Intro to Automation](05-tools-and-process/05-intro-to-automation.md)

**Practice / Project:**

- Execute your Week 2 test cases (**Pass / Fail / Blocked**). For every failure, write
  a **complete bug report** with **severity + priority** and a screenshot.
- Produce a **bug-report pack** of 5+ polished reports — include one case where
  severity and priority differ.
- Recreate three reports as tickets in **free Jira** and move them across a board.
- Send a few **Postman** API requests (valid + invalid) and do a **cross-browser +
  responsive** check of one page.

**By the end of the week you can:** report and classify defects professionally, and
use a tracker, Postman, and cross-browser tools.

---

## Week 4 — Mastery and Final Project

**Goal:** Turn your skills into a portfolio, prep for interviews, and complete a full
end-to-end QA cycle as your capstone.

**Study:**

- Module 6 — [Building a QA Portfolio](06-mastery/01-building-a-qa-portfolio.md),
  [Interview Preparation](06-mastery/02-interview-preparation.md),
  [Career Paths](06-mastery/03-career-paths-and-certifications.md),
  [Final Project](06-mastery/04-final-project.md)

**Practice / Project — the capstone:**

- Complete the [Final Project](06-mastery/04-final-project.md): run a **full manual
  QA cycle** on your app — test plan, scenarios, **15–25 test cases**, RTM, executed
  results, bug reports, an exploratory session, cross-browser checks, and a **test
  summary report** with a ship / don't-ship recommendation.
- Organize everything into a `qa-portfolio` folder (ideally on **GitHub** — use the
  repo's [Git course](../git-course/) if you need a refresher).
- Do a **mock interview** out loud, including "how would you test a ___?"

**By the end of the week you can:** demonstrate the entire manual QA process with real
artifacts, talk confidently in an interview, and know your next career step.

---

## After the Roadmap

- **Polish your portfolio** and start applying for junior QA roles.
- **Keep practicing** — every app you use is free practice.
- **Pick a direction:** API testing → automation (your repo's
  [JavaScript course](../javascript-course/) is a great next step), or a specialty
  like performance, security, or mobile. See
  [Career Paths](06-mastery/03-career-paths-and-certifications.md).
- **Consider ISTQB Foundation** if you want a recognized certificate — this course
  maps closely to its syllabus.

Happy testing! 🐛🔍
