# Agile Testing and the QA Role in Scrum

Most modern teams build software using **Agile**, usually a flavor called **Scrum**.
As a tester, you won't be a separate department that tests "at the end" — you'll be
an embedded member of a small team, testing continuously. This lesson explains how
Agile works and exactly where you fit, so you can walk into a stand-up and feel at
home.

## What You'll Learn

- What Agile and Scrum actually mean, in plain terms
- The key Scrum ceremonies (meetings) and your role in each
- What QA does across a sprint
- The "three amigos" and "definition of done"

## Agile in One Paragraph

Instead of building the *whole* product over months and testing at the end
(Waterfall), Agile builds in **short repeating cycles**, delivering a small working
piece each time and adjusting based on feedback. The most popular framework for doing
this is **Scrum**, which organizes work into fixed-length **sprints** (commonly two
weeks).

## The Building Blocks of Scrum

- **Product Backlog** — the master list of everything to build, as "user stories."
- **User Story** — a small feature from the user's view: *"As a shopper, I want to
  save items to a wishlist so I can buy them later."*
- **Sprint** — a fixed time-box (e.g., two weeks) in which the team builds and tests
  a chosen set of stories.
- **Sprint Backlog** — the stories chosen for the current sprint.
- **Increment** — the working, tested software produced by the sprint.

## The Scrum Ceremonies (and Your Role)

```text
   Sprint Planning ──▶ [ Daily Stand-ups for ~2 weeks ] ──▶ Sprint Review ──▶ Retrospective
        │                        │                              │                  │
   pick stories            sync each day                 demo the work       improve process
```

| Ceremony | What happens | Your role as QA |
| --- | --- | --- |
| **Sprint Planning** | Team picks stories for the sprint | Ask clarifying questions; estimate test effort; raise testability concerns *early* |
| **Daily Stand-up** | 15-min sync: done / doing / blockers | Report testing progress and blockers ("waiting on build for story 4") |
| **Sprint Review** | Demo finished work to stakeholders | Confirm what's genuinely *done and tested* |
| **Retrospective** | What went well / what to improve | Suggest process improvements ("bugs found late — let's test stories sooner") |

The daily stand-up is just three questions: *What did I do yesterday? What will I do
today? What's blocking me?* Keep it short.

## What QA Does Across a Sprint

The magic of Agile QA is that testing happens **throughout**, not at the end:

```text
   Day 1-2:  Planning. Read stories, ask questions, draft test scenarios.
   Day 3-7:  Devs build story by story. You write test cases, prep test data.
             As each story is coded, you TEST it (don't wait!).
   Day 8-9:  Bugs get fixed; you retest. Regression-test the increment.
   Day 10:   Review/demo. Only truly tested work is called "done."
```

The anti-pattern to avoid: all the testing piling up in the last two days ("mini-
waterfall inside the sprint"). Push to test each story **as soon as it's ready**.

## Two Phrases You'll Hear Constantly

### "Definition of Done" (DoD)

A shared checklist for when a story is *truly* finished — not just "code written."
A typical DoD includes: code complete, **tested**, no open critical bugs, reviewed,
documented. QA is central to the DoD: a story isn't "done" until it's tested. Use
the DoD to resist pressure to call untested work complete.

### "Three Amigos"

Before building a story, three perspectives meet: **business** (product owner),
**development** (developer), and **testing** (QA). Together they clarify what the
story means and what "correct" looks like *before* coding starts. This is "shift
left" in action — you catch unclear requirements before a line of code exists. As QA,
your job here is to ask the "what if?" questions
([the tester's mindset](../00-getting-started/02-the-testers-mindset.md)).

## Why Agile Suits QA

- You're involved from **day one**, not handed finished code at the end.
- Bugs are found **early**, while they're cheap to fix.
- You have a **voice** in planning and process.
- Continuous testing beats a frantic test-everything-at-the-end crunch.

## Exercises

1. Explain Scrum to a friend in three sentences (sprint, user story, increment).
2. List the four main Scrum ceremonies and your QA role in each.
3. What is a "Definition of Done," and why is QA central to it?
4. What is the "three amigos" practice, and which testing principle does it embody?

Next: [API Testing Basics](03-api-testing-basics.md)
