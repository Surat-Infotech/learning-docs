# QA Testing Course — 6-Week Weekly Roadmap (10 hours/week)

This roadmap turns the [QA Testing Course](README.md) into a clear weekly plan.

- **Time budget:** 10 hours per week
- **Total length:** 6 weeks (~60 hours)
- **Split:** About **40% reading** and **60% hands-on** — practicing on real apps,
  writing test cases, and finding bugs.

> **How to read this plan.** Each week lists the lessons to study and a hands-on
> project. Pick a real app you use (a shop, a to-do app, a booking site) and test it
> as you learn — you'll find real bugs, which is exactly the point. Save your best
> work in a `qa-portfolio` folder; by Week 6 it becomes your job-hunting portfolio.

**Weekly rhythm (suggested):** 4–5 short sessions of ~2 hours. Consistent practice
beats marathons — testing is a skill that grows by *doing*.

---

## Week-at-a-Glance

| Week | Focus                          | Lessons             | Weekly Project                              |
| ---- | ------------------------------ | ------------------- | ------------------------------------------- |
| 1    | Mindset + Fundamentals         | Module 0 + Module 1 | Test-Concept Notes + First Bug Hunt         |
| 2    | Types of Testing               | Module 2            | Classify & Run Tests on a Real App          |
| 3    | Test Design & Documentation    | Module 3            | Write Test Cases + a Traceability Matrix    |
| 4    | Defect Management              | Module 4            | A Polished Bug Report Pack                  |
| 5    | Tools & Process                | Module 5            | Set Up Jira/Postman + Cross-Browser Testing |
| 6    | Mastery + Final Project        | Module 6            | Full End-to-End QA of a Real App            |

---

## Week 1 — Mindset and Fundamentals

**Goal:** Understand what QA is and absorb the core vocabulary every tester needs.

**Study:**

- Module 0 — [What Is QA](00-getting-started/01-what-is-qa.md),
  [The Tester's Mindset](00-getting-started/02-the-testers-mindset.md)
- Module 1 — all five lessons (SDLC, STLC, verification vs validation, the seven
  principles, levels of testing)

**Practice / Project:**

- Keep a one-page **cheat sheet** in your own words: QA vs QC, verification vs
  validation, the seven principles, the STLC phases.
- **First bug hunt:** pick any app and spend an hour just *using it like a curious
  tester* — go off the happy path. Write down every odd thing you notice (no formal
  report yet).

**By the end of the week you can:** explain what QA is, where it fits in the SDLC,
and the fundamental principles — and you've started looking at software with a
tester's eyes.

---

## Week 2 — Types of Testing

**Goal:** Know the major kinds of testing and when each applies.

**Study:**

- Module 2 — all six lessons (functional, non-functional, regression/retesting,
  smoke/sanity, exploratory/ad-hoc, black/grey/white box)

**Practice / Project:**

- On your chosen app, do a quick **smoke test** of the critical paths.
- Run a **functional test** of one feature (write input → action → expected result).
- Run a 30-minute **exploratory session** with a written charter; keep notes.
- For five things you tested, label each: functional or non-functional? black or
  grey box?

**By the end of the week you can:** identify and perform the main testing types, and
explain how they differ (great interview material).

---

## Week 3 — Test Design and Documentation

**Goal:** Write the documents testers live by, using proper design techniques.

**Study:**

- Module 3 — all five lessons (scenarios vs cases, writing test cases, design
  techniques, test plans, RTM)

**Practice / Project:**

- Choose one feature of your app. Write **4–6 test scenarios**, then expand them into
  **10–15 detailed test cases** (with the standard fields).
- Apply **design techniques**: use equivalence partitioning + boundary values on an
  input field, and build a **decision table** for a rule-based feature.
- Build a small **Requirements Traceability Matrix** linking requirements to your
  cases. Confirm every requirement has at least one test.

**By the end of the week you can:** turn requirements into well-designed, traceable
test cases — the daily core skill of manual QA.

---

## Week 4 — Defect Management

**Goal:** Find, report, and classify bugs like a professional.

**Study:**

- Module 4 — all four lessons (defect life cycle, bug reports, severity vs priority,
  working with developers)

**Practice / Project:**

- Execute the test cases you wrote in Week 3. Mark each **Pass / Fail / Blocked**.
- For every failure (and any bug from your exploration), write a **complete bug
  report** using the template: title, steps, expected vs actual, environment,
  **severity + priority**, and a **screenshot**.
- Produce a **bug report pack** of 5+ real, polished reports. Practice setting
  severity and priority — including one case where they differ.

**By the end of the week you can:** write reproducible bug reports developers
respect, and classify defects correctly. These reports go straight into your
portfolio.

---

## Week 5 — Tools and Process

**Goal:** Get comfortable with the tools and team processes used in real QA jobs.

**Study:**

- Module 5 — all five lessons (test management & bug tracking tools, Agile/Scrum, API
  testing basics, cross-browser & mobile, intro to automation)

**Practice / Project:**

- Sign up for a **free Jira** account; recreate three of your Week 4 bug reports as
  tickets and move them across a board.
- Install **Postman** and run a few **API requests** against any free public API
  (send valid and invalid requests; read the status codes).
- Do a **cross-browser + responsive** check of one page: test it in two browsers and
  at a phone screen size (use DevTools device mode). Screenshot any issues.

**By the end of the week you can:** use a bug tracker, do basic API testing, test
across browsers/devices, and explain where QA fits in Agile/Scrum.

---

## Week 6 — Mastery and Final Project

**Goal:** Turn your skills into a portfolio and complete a full QA cycle.

**Study:**

- Module 6 — all four lessons (portfolio, interview prep, career paths, final
  project)

**Practice / Project — the capstone:**

- Complete the [Final Project](06-mastery/04-final-project.md): run a **full manual
  QA cycle** on a real app, producing every artifact — test plan, scenarios, test
  cases, RTM, executed results, bug reports, exploratory notes, cross-browser checks,
  and a **test summary report** with a ship / don't-ship recommendation.
- Organize everything into a `qa-portfolio` folder (ideally on **GitHub** — use the
  repo's [Git course](../git-course/) if you need a refresher).
- Do a mock interview: answer the common questions from the
  [interview lesson](06-mastery/02-interview-preparation.md) out loud, including
  "how would you test a ___?"

**By the end of the week you can:** demonstrate the entire manual QA process with
real artifacts, talk confidently in an interview, and know your next career step.

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
