# Test Management and Bug Tracking Tools

In a real job, you won't keep test cases in a notebook or report bugs by shouting
across the room. Teams use **tools** to organize tests, track bugs, and report
progress. You don't need to master every tool — they all share the same concepts.
This lesson introduces the categories and the big names so you walk into a job
already knowing the landscape.

## What You'll Learn

- The two main tool categories: test management and bug tracking
- The popular tools you'll hear about (Jira, TestRail, Zephyr)
- What a workflow in these tools looks like
- Why a spreadsheet is a fine place to start

## Two Categories of Tool

| Category | Job | Examples |
| --- | --- | --- |
| **Test management** | Store and organize test cases; record runs; report coverage | TestRail, Zephyr, Xray, qase |
| **Bug / issue tracking** | Log, assign, and track defects through their life cycle | Jira, Bugzilla, GitHub Issues, Linear |

Some tools (like Jira with plugins) try to do both. In many companies you'll use
**Jira for bugs** and a dedicated tool like **TestRail for test cases**, linked
together.

## Bug Tracking: Jira (the one you'll almost certainly meet)

**Jira** is the most common issue tracker in the industry. A bug in Jira is a
"ticket" (or "issue") that carries all the fields from your
[bug report](../04-defect-management/02-writing-bug-reports.md) — title, steps,
severity, priority, attachments — and moves through statuses on a **board**:

```text
   ┌─────────┐   ┌─────────────┐   ┌──────────┐   ┌──────────┐
   │ To Do   │──▶│ In Progress │──▶│ In Review│──▶│   Done   │
   │ (New)   │   │ (dev fixing)│   │ (QA test)│   │ (Closed) │
   └─────────┘   └─────────────┘   └──────────┘   └──────────┘
```

Notice this board *is* the [defect life
cycle](../04-defect-management/01-defect-life-cycle.md) made visual. You drag a
ticket across columns as it progresses. Each team configures its own columns, but
the idea is universal.

## Test Management: TestRail (a representative example)

A test management tool lets you:

- **Organize** test cases into folders/suites (e.g., "Login," "Checkout").
- **Write** cases with the standard fields (steps, expected result, priority).
- **Run** a "test run" — go through a set of cases marking each Pass/Fail/Blocked.
- **Report** — see pass rates, coverage, and what's left, often with a
  [traceability matrix](../03-test-design-and-documentation/05-traceability-matrix.md)
  generated automatically.

```text
   Test Suite: Checkout
     ├── TC-101  Add item to cart          [ Pass ]
     ├── TC-102  Apply coupon              [ Fail ] → linked Jira bug BUG-456
     ├── TC-103  Pay with credit card      [ Pass ]
     └── TC-104  Pay with expired card     [Blocked]
   Run summary: 2 passed, 1 failed, 1 blocked  (50% pass)
```

When a case fails, you link it to a bug ticket — connecting your test tool and your
bug tracker.

## A Typical Daily Workflow

1. Open the **test management tool**, start a **test run** for today's build.
2. Execute test cases one by one, marking **Pass/Fail/Blocked**.
3. For each **Fail**, create a bug in **Jira** with full details and link it to the
   failed case.
4. Update the team in **stand-up** on progress and blockers.
5. When developers mark bugs **Fixed**, **retest** and close or reopen them.

## You Can Start with a Spreadsheet

Don't feel you must learn expensive tools before you can test. A simple spreadsheet
with columns — *ID, Title, Steps, Expected, Actual, Status* — is a perfectly valid
way to manage test cases while learning, and many small teams genuinely do this.
**The tool is just a container for the concepts you already learned.** Master the
concepts; the tools take an afternoon to pick up.

## Tips for Learning Tools on the Job

- Most tools offer **free trials or free tiers** (Jira, TestRail, qase) — practice
  on a personal project.
- The vocabulary transfers: "issue," "ticket," "status," "board," "test run" mean
  the same thing everywhere.
- Focus interviews on **concepts**, not memorizing menus. "I've used Jira and
  TestRail; I can pick up any similar tool" is exactly the right answer.

## Exercises

1. Name the two categories of QA tools and give one example of each.
2. Map a Jira board's columns (To Do → In Progress → In Review → Done) to the
   defect life cycle statuses.
3. In a test management tool, what does a "test run" do, and what three statuses do
   you mark cases with?
4. Create a 5-row test-case spreadsheet for a feature of your choice, with the
   standard columns.

Next: [Agile Testing and the QA Role in Scrum](02-agile-testing-and-scrum.md)
