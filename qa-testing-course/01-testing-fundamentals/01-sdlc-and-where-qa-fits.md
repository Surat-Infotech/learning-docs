# The SDLC and Where QA Fits

Software doesn't appear out of nowhere. It moves through a series of stages, from
"someone has an idea" to "people are using it." That journey is called the
**Software Development Life Cycle (SDLC)**. To be a good tester, you need to know
the stages — because QA is involved in *every single one*, not just at the end.

## What You'll Learn

- The classic stages of building software
- Where testing happens in each stage (hint: everywhere)
- The difference between Waterfall and Agile, in plain terms
- Why "testing at the end" is a trap

## The Stages of the SDLC

```text
  1. Requirements  →  2. Design  →  3. Development  →  4. Testing  →  5. Deployment  →  6. Maintenance
     (what to build)   (how)         (build it)         (check it)     (ship it)         (keep it healthy)
```

1. **Requirements** — Decide *what* to build and *why*. Talk to users and
   stakeholders. Write down what the software must do.
2. **Design** — Plan *how* to build it: screens, data, structure, technical
   approach.
3. **Development** — Programmers write the actual code.
4. **Testing** — Check that what was built matches what was wanted, and find bugs.
5. **Deployment** — Release the software to real users.
6. **Maintenance** — Fix issues, add improvements, keep it running.

## Where Does QA Fit? (Everywhere)

Beginners think QA only happens in stage 4. That's the trap. Good QA contributes
to **every** stage:

| Stage | What a tester does here |
| --- | --- |
| **Requirements** | Ask questions, spot vague or contradictory requirements, think about edge cases *before* anything is built. |
| **Design** | Review the plan for testability and missing scenarios. |
| **Development** | Prepare test cases and test data so you're ready the moment code lands. |
| **Testing** | Run tests, report bugs, retest fixes. |
| **Deployment** | Run smoke tests to confirm the release didn't break basics. |
| **Maintenance** | Regression-test new fixes; make sure changes don't break old features. |

The earlier QA gets involved, the cheaper the bugs. A confusing requirement caught
in stage 1 is a five-minute chat. The same confusion caught in stage 4 means code
gets rewritten. This is the **"shift left"** idea from
[Module 0](../00-getting-started/01-what-is-qa.md): move testing thinking earlier.

## Waterfall vs Agile (The Two Big Styles)

You'll hear these words constantly. Here's the plain version.

### Waterfall

Do each stage **fully**, in order, before starting the next — like a waterfall
flowing down. Requirements are *all* written, then *all* design, then *all* coding,
then *all* testing.

- **Good for:** projects with fixed, well-understood requirements.
- **Risk for QA:** testing is squeezed at the very end, so bugs are found late and
  there's pressure to rush.

### Agile

Build in small repeated cycles called **sprints** (often 1–2 weeks). Each sprint
produces a small working piece, which is tested *during* the sprint. Then you
repeat.

- **Good for:** projects where requirements change and feedback matters.
- **Great for QA:** testers are part of the team from day one, testing continuously
  instead of at the end.

```text
  Waterfall:  [Req]→[Design]→[Code]→[Test]→[Ship]      (test once, at the end)

  Agile:      Sprint 1: [plan→build→test→review]
              Sprint 2: [plan→build→test→review]        (test every sprint)
              Sprint 3: [plan→build→test→review]
```

Most modern teams use Agile (often a flavor called **Scrum**). We cover the QA
role in Agile in detail in
[Module 5](../05-tools-and-process/02-agile-testing-and-scrum.md).

## Exercises

1. List the six SDLC stages from memory, then write one QA activity for each.
2. In your own words, why is finding a bug in the Requirements stage cheaper than
   finding it after Deployment?
3. A team writes *all* requirements, builds for three months, then tests for one
   week before launch. Which style is this, and what's the biggest risk?
4. Why does Agile tend to suit QA better than Waterfall?

Next: [The Software Testing Life Cycle (STLC)](02-stlc.md)
