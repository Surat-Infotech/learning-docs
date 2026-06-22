# Requirements Traceability Matrix (RTM)

How do you *prove* you tested everything the customer asked for? You connect each
**requirement** to the **test cases** that check it. That connection, laid out in a
simple table, is the **Requirements Traceability Matrix (RTM)**. It's how teams
guarantee no requirement slips through untested.

## What You'll Learn

- What traceability means and why it matters
- What an RTM looks like
- How it catches gaps in coverage
- How to read and contribute to one

## The Problem RTM Solves

A project has, say, 40 requirements and 200 test cases. How do you answer these
questions with confidence?

- "Did we write a test for **every** requirement?"
- "This requirement changed — which tests do I need to update?"
- "This test failed — which customer requirement is now at risk?"

Without a map connecting requirements to tests, you're guessing. The RTM **is** that
map.

## What Traceability Means

**Traceability** = the ability to *trace* a link between two things — here, between a
**requirement** and the **test case(s)** that verify it. If every requirement traces
to at least one test, you have **full coverage**. If a requirement traces to *no*
test, you've found a dangerous gap.

## What an RTM Looks Like

At its simplest, it's a table linking requirement IDs to test case IDs:

| Requirement ID | Requirement | Test Case IDs | Status |
| --- | --- | --- | --- |
| R-01 | User can log in with valid credentials | TC-001, TC-002 | ✅ Pass |
| R-02 | Login fails with wrong password | TC-003 | ✅ Pass |
| R-03 | User can reset a forgotten password | TC-010, TC-011 | ⚠️ 1 Fail |
| R-04 | User can update their email address | *(none)* | ❌ **No tests!** |

Look at **R-04** — it has *no* test cases. The RTM just caught a coverage gap: a
requirement nobody wrote a test for. That's the whole point.

## How It Catches Gaps (Two Directions)

The matrix lets you check coverage **both ways**:

- **Forward (requirements → tests):** every requirement should have at least one
  test. A requirement with no test = a hole in coverage.
- **Backward (tests → requirements):** every test should trace to a requirement. A
  test linked to *nothing* might be testing something nobody asked for (wasted
  effort) — or hint at a missing requirement.

```text
   Requirements  ───────────▶  Test Cases
        R-01  ───────────────▶  TC-001, TC-002     ✅ covered
        R-02  ───────────────▶  TC-003             ✅ covered
        R-03  ───────────────▶  TC-010, TC-011     ✅ covered
        R-04  ───────────────▶  (nothing)          ❌ GAP
                                TC-099 ◀── traces to no requirement ❓
```

## Why It's Worth the Effort

- **Coverage proof.** You can *show* stakeholders that every requirement is tested.
- **Impact analysis.** A requirement changes → instantly see which tests to update.
- **Failure tracing.** A test fails → instantly see which business requirement is at
  risk, to judge severity.
- **Audits & compliance.** In regulated industries (medical, finance), an RTM is
  often *required*.

## Your Realistic Role

You may not build the RTM from scratch early on, but you will:

- **Link** your new test cases to the right requirement IDs.
- **Spot gaps** — "R-04 has no tests, should I write some?"
- **Update** the matrix when requirements or tests change.

Many test management tools (TestRail, Jira plugins — see
[Module 5](../05-tools-and-process/01-test-management-tools.md)) generate the RTM
automatically once you link cases to requirements. So in practice you often just
maintain the *links*, and the tool builds the matrix.

## Exercises

1. In one sentence, what does an RTM let you prove?
2. Given requirements R-01…R-05 and the fact that R-03 has no linked test cases,
   what does that tell you, and what would you do?
3. Explain forward vs backward traceability.
4. Build a small 3-row RTM for a "search products" feature: invent three
   requirements and link plausible test case IDs to each.

Next: [Module 4 — Defect Management](../04-defect-management/01-defect-life-cycle.md)
