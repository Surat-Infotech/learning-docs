# Test Plans and Test Strategy

Before a team starts testing seriously, someone writes a **test plan** — the
document that answers *what* we'll test, *how*, *who* does it, *when*, and *what
could go wrong*. As a junior tester you won't write the whole thing on day one, but
you'll *contribute* to it and *work from* it constantly. Understanding it makes you
look senior fast.

## What You'll Learn

- What a test plan is and why it exists
- The standard sections of a test plan
- The difference between test plan and test strategy
- The idea of scope: what's in, what's out

## What Is a Test Plan?

A **test plan** is a document that describes the **testing approach for a specific
project or release**. It's the map for the whole testing effort — so everyone agrees
on what "done" looks like *before* the work starts.

Think of it as the answer to: *"How are we going to test this thing, and how will we
know we're finished?"*

## The Standard Sections

Real templates vary, but most test plans cover these:

| Section | What it answers |
| --- | --- |
| **Objective** | Why are we testing? What's the goal of this release? |
| **Scope** | What *will* and *won't* be tested (see below) |
| **Test approach** | What types of testing (functional, regression, etc.) and how |
| **Test environment** | What hardware, browsers, devices, data we need |
| **Roles & responsibilities** | Who does what |
| **Schedule** | When testing starts and ends; key milestones |
| **Entry & exit criteria** | When we can start, and when we can call it done |
| **Deliverables** | What we'll produce (test cases, reports, bug logs) |
| **Risks & mitigation** | What could derail testing, and our backup plan |
| **Tools** | What software we'll use (test management, bug tracking) |

## Scope: In and Out

The most important — and most skipped — part is defining **scope** clearly:

- **In scope:** "We will test login, checkout, and search on Chrome and Safari."
- **Out of scope:** "We will **not** test on Internet Explorer, and we will **not**
  performance-test this release."

Why bother stating what you *won't* do? Because it prevents the dreaded *"wait, why
didn't QA test that?!"* after launch. Out-of-scope items are a deliberate,
documented decision — not an oversight. This protects you and sets expectations.

## Test Plan vs Test Strategy

These overlap and people argue about the exact line. The simple version:

| | Test Strategy | Test Plan |
| --- | --- | --- |
| **Level** | Organization / high-level | Specific project or release |
| **Changes** | Rarely (a long-lived guideline) | Per project |
| **Content** | The overall *philosophy* and standards for testing | The concrete *who/what/when* for this release |
| **Owner** | Senior QA / management | Test lead for the project |

Roughly: **strategy** is the company's general testing approach; the **plan** applies
that approach to *this* project. Many smaller teams fold both into one document and
just call it the test plan — don't lose sleep over the distinction.

## Risks and Mitigation (a junior-friendly highlight)

Every plan should name what could go wrong and the backup plan. Examples:

- **Risk:** the test environment might not be ready in time.
  **Mitigation:** prepare test cases and data in advance so we're ready the moment
  it is.
- **Risk:** a key feature arrives late, squeezing test time.
  **Mitigation:** prioritize critical test cases first (risk-based testing).

Thinking about risk shows maturity and is a great thing to mention in interviews.

## Your Realistic Role as a Junior

You probably won't author the whole plan early on. But you will:

- **Read it** to understand scope and priorities.
- **Estimate** how long your test cases will take.
- **Flag risks** you notice ("we have no test devices for Android").
- **Report progress** against it.

Knowing the structure means you can contribute intelligently from week one.

## Exercises

1. List six sections you'd expect to find in a test plan.
2. Why is documenting **out-of-scope** items just as important as in-scope ones?
3. Explain the difference between a test strategy and a test plan in your own words.
4. Write one **risk** and a sensible **mitigation** for a project where the test
   environment is shared with developers.

Next: [Requirements Traceability Matrix](05-traceability-matrix.md)
