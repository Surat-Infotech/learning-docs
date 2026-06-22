# Severity vs Priority

Every bug report asks you to set two values that beginners constantly confuse:
**severity** and **priority**. They are *not* the same thing, and a classic
interview question is to give an example where they differ. Get this clear once and
you'll classify bugs like a pro.

## What You'll Learn

- What severity means
- What priority means
- Why they're independent
- The four classic combinations (with examples)

## Severity — How Bad Is the Damage?

**Severity** measures the **technical impact** of the bug — how badly it breaks the
software. It's about the *defect itself*, regardless of business concerns.

Typical levels:

| Severity | Meaning | Example |
| --- | --- | --- |
| **Critical** | App crashes / data loss / total blocker | Login is completely down; nobody can use the app |
| **Major (High)** | Big feature broken, no workaround | Checkout fails for all credit cards |
| **Minor (Medium)** | Feature issue with a workaround | Sort button works only after refresh |
| **Trivial (Low)** | Cosmetic, no functional impact | A typo on the About page |

Severity is usually set by **QA** (you), based on what you observed.

## Priority — How Soon Must We Fix It?

**Priority** measures the **business urgency** — how quickly it should be fixed
relative to everything else. It's about *scheduling*, and it considers things like
deadlines, customer visibility, and brand impact.

| Priority | Meaning |
| --- | --- |
| **High (P1)** | Fix immediately / before release |
| **Medium (P2)** | Fix soon, in the normal queue |
| **Low (P3)** | Fix when there's time |

Priority is usually decided by the **product owner / manager** (often with QA
input), because it depends on business context.

## Why They're Independent

Here's the key insight: **a bug's technical severity and its business urgency can
disagree.** That's why we track them separately. The four combinations:

```text
                  HIGH PRIORITY                 LOW PRIORITY
              ┌───────────────────────┬───────────────────────┐
   HIGH       │ Login is down for     │ Rare crash only when   │
   SEVERITY   │ everyone.             │ you do 12 weird steps  │
              │ Fix NOW.              │ no real user does.     │
              ├───────────────────────┼───────────────────────┤
   LOW        │ CEO's name misspelled │ Typo on a rarely-seen  │
   SEVERITY   │ on the homepage.      │ help page.             │
              │ Trivial bug, but fix  │ Fix whenever.          │
              │ urgently (embarrass-  │                        │
              │ ing & very visible).  │                        │
              └───────────────────────┴───────────────────────┘
```

### The two famous "they differ" cases

**High severity, low priority:**
A crash that happens only under a bizarre sequence of steps no real user would ever
do. *Technically* severe (it crashes!), but *not urgent* (almost nobody hits it).

**Low severity, high priority:**
The company name or CEO's name is misspelled on the homepage. *Technically* trivial
(just text), but *extremely urgent* — it's embarrassing and seen by everyone, so fix
it before anything else.

These two examples are the heart of the concept. If you remember them, you
understand severity vs priority.

## Quick Comparison

| | Severity | Priority |
| --- | --- | --- |
| **Measures** | Technical impact | Business urgency |
| **Question** | How badly does it break things? | How soon must we fix it? |
| **Set by** | QA / tester | Product owner / manager |
| **Depends on** | The defect itself | Business context, deadlines, visibility |

## Why Tracking Both Helps

When developers plan their work, they look at *both*. A high-severity, high-priority
bug jumps the queue. A high-severity but low-priority bug can wait. Two numbers give
a far better picture than one — they let the team fix the *right* things in the
*right* order.

## Exercises

1. Define severity and priority in one sentence each.
2. Classify each bug's likely severity AND priority:
   a. The payment page crashes for all users.
   b. A typo in the footer copyright year.
   c. The company logo is broken on the homepage.
   d. A crash that needs 15 unusual steps to trigger.
3. Give your *own* example of a high-severity, low-priority bug.
4. Why is it useful to track severity and priority separately instead of one
   combined "importance" score?

Next: [Working with Developers and Triage](04-working-with-developers.md)
