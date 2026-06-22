# QA Interview Preparation

You've built the skills and a portfolio — now let's get you ready for the interview.
QA interviews are surprisingly predictable: the same core concepts and the same
"how would you test X?" exercises come up again and again. This lesson collects the
most common questions, the famous trick exercise, and how to answer like someone who
actually understands testing (which, after this course, you do).

## What You'll Learn

- The most common QA interview questions
- How to answer the classic "how would you test a ___?" exercise
- Behavioral questions and how to approach them
- Smart questions to ask *them*

## Core Concept Questions (Know These Cold)

Every one of these is covered in this course. Be able to answer each in a clear
sentence or two:

| Question | Where you learned it |
| --- | --- |
| Difference between QA, QC, and testing? | [Module 0](../00-getting-started/01-what-is-qa.md) |
| Verification vs validation? | [Module 1](../01-testing-fundamentals/03-verification-vs-validation.md) |
| Severity vs priority? (give an example where they differ!) | [Module 4](../04-defect-management/03-severity-vs-priority.md) |
| Smoke vs sanity testing? | [Module 2](../02-types-of-testing/04-smoke-and-sanity.md) |
| Retesting vs regression testing? | [Module 2](../02-types-of-testing/03-regression-and-retesting.md) |
| The defect life cycle? | [Module 4](../04-defect-management/01-defect-life-cycle.md) |
| The STLC phases? | [Module 1](../01-testing-fundamentals/02-stlc.md) |
| Functional vs non-functional testing? | [Module 2](../02-types-of-testing/01-functional-testing.md) |
| Black vs white vs grey box? | [Module 2](../02-types-of-testing/06-black-grey-white-box.md) |
| What goes in a good bug report? | [Module 4](../04-defect-management/02-writing-bug-reports.md) |

> **Tip:** for the "vs" questions, always have a one-line *example* ready. "Low
> severity, high priority? A typo in the company logo on the homepage." Examples
> prove you *understand* it, not just memorized it.

## The Famous Exercise: "How Would You Test a ___?"

This is the signature QA interview question. They'll name something — sometimes
software, sometimes a real-world object — and watch *how you think*. Classics:

- "How would you test a login page?"
- "How would you test a pen?"
- "How would you test an elevator?"
- "How would you test a vending machine?"

They are **not** looking for one right answer. They're looking for a **structured,
thorough thought process**. Use a framework out loud:

```text
   1. Clarify   — "Who uses it? What's it for? Any requirements?"
   2. Functional — Does it do its core job? (happy paths)
   3. Negative   — What if used wrong? (unhappy paths, edge cases)
   4. Non-functional — Performance, usability, safety, durability?
   5. Boundaries — Limits, extremes, maximums.
```

**Example — "test a pen":**

- *Clarify:* What kind? Who uses it? What surface?
- *Functional:* Does it write? In the right color? Smoothly?
- *Negative:* Does it write upside down? On wet paper? After being dropped?
- *Non-functional:* Comfortable to hold? How long does the ink last? Does it leak?
- *Boundaries:* Write continuously until ink runs out; extreme temperatures.

Notice this is just the **[tester's
mindset](../00-getting-started/02-the-testers-mindset.md)** applied out loud. You
already own this skill — now narrate it.

## Behavioral Questions

They'll also probe how you *work*:

- "Tell me about a bug you found and how you reported it." → Use a portfolio example.
- "A developer says your bug isn't a bug. What do you do?" → Go to the requirement,
  focus on user impact, stay professional
  ([Module 4](../04-defect-management/04-working-with-developers.md)).
- "How do you decide what to test when there's no time to test everything?" →
  Risk-based prioritization; critical/most-used features first
  ([principle 2](../01-testing-fundamentals/04-principles-of-testing.md)).
- "You're told to ship but testing isn't finished. What do you do?" → Communicate
  the *risk* clearly and factually; let the team decide with eyes open.

Structure behavioral answers with **STAR**: **S**ituation, **T**ask, **A**ction,
**R**esult.

## Questions to Ask *Them*

Interviews go both ways. Asking good questions signals maturity:

- "What does the QA process look like here — Agile? Where does QA fit in the sprint?"
- "What tools do you use for test management and bug tracking?"
- "Is testing mostly manual, automated, or both?"
- "What does the Definition of Done include?"

## Final Tips

- **Think out loud.** For the "how would you test" questions, your *process* is the
  answer.
- **Use real examples** from your portfolio — concrete beats theoretical.
- **Be honest about gaps.** "I haven't used Selenium, but I understand automation's
  role and I learn tools quickly" is a strong, honest answer.
- **Show curiosity.** Enthusiasm for breaking things and improving quality is
  exactly the QA personality teams want.

## Exercises

1. Write one-sentence answers (with an example) for: severity vs priority, smoke vs
   sanity, verification vs validation.
2. Out loud, answer "How would you test an ATM?" using the five-step framework.
3. Prepare a STAR-format story about a bug you found (use a portfolio bug).
4. Write three questions *you'd* ask an interviewer about their QA process.

Next: [Career Paths and Certifications](03-career-paths-and-certifications.md)
