# Exploratory and Ad-hoc Testing

Not all testing follows a script. Sometimes the best bugs are found by simply
*using* the software with curiosity, going wherever your instincts lead. This is
**exploratory testing**, and it's where the [tester's
mindset](../00-getting-started/02-the-testers-mindset.md) really shines. It's also
where many testers have the most fun.

## What You'll Learn

- What exploratory testing is and why it's powerful
- How it differs from scripted testing
- What ad-hoc testing is
- How to do exploratory testing in a focused, professional way (not random
  clicking)

## Scripted vs Exploratory

**Scripted testing** = follow pre-written test cases step by step. Predictable,
repeatable, good for coverage and regression.

**Exploratory testing** = you *simultaneously* learn the app, design tests in your
head, and run them — all at once, guided by what you discover. You decide your next
move based on what just happened.

| | Scripted | Exploratory |
| --- | --- | --- |
| **Plan** | Written in advance | Created on the fly |
| **Strength** | Coverage, repeatability | Finding the *unexpected* |
| **Feels like** | Following a recipe | Investigating like a detective |
| **Best for** | Regression, requirements coverage | New features, "what did we miss?" |

Good QA uses **both**. Scripts make sure you cover the known; exploration finds the
unknown.

## Why Exploratory Testing Finds Great Bugs

Scripted tests can only find the problems someone *anticipated* when writing them.
Exploratory testing finds the problems **nobody thought of** — exactly the ones that
embarrass teams in production. By reacting to the software's actual behavior, you go
places a pre-written script never would.

## Ad-hoc Testing

**Ad-hoc testing** is the most informal kind: no plan, no documentation, just
poking at the software randomly to try to break it. It's even less structured than
exploratory testing.

- Exploratory testing is *structured improvisation* — guided, thoughtful, often
  noted down.
- Ad-hoc testing is *unstructured* — "let me just mess with this and see."

Ad-hoc has its place (a quick gut-check), but exploratory testing is the more
*professional, repeatable-ish* version — and the one you should lean on.

## Doing It Right: Structured Exploration

"Exploratory" doesn't mean "aimless." Pros keep it focused so the time is
accountable. A common technique is the **session-based** approach:

1. **Pick a charter** — a clear mission for the session.
   > *"Explore the checkout flow with invalid payment data for 45 minutes."*
2. **Time-box it** — set a fixed duration (e.g., 30–60 minutes).
3. **Explore and take notes** — record what you tried, what you found, and any
   questions, as you go.
4. **Report** — summarize bugs, risks, and areas that need deeper testing.

This gives you the freedom of exploration *and* a record of what was covered.

## Practical Tips for Exploring

- **Follow the smell.** Something looked slightly off? Dig there — bugs cluster.
- **Vary your data.** Long names, symbols, emoji, zero, negatives, foreign
  characters.
- **Interrupt things.** Refresh mid-action, hit Back, lose connection, double-click.
- **Change the order.** Do steps in an unexpected sequence.
- **Use real-user empathy.** "What would a confused or rushing user do here?"
- **Take screenshots** of anything weird — you'll need them for bug reports
  (Module 4).

## Exercises

1. Explain the difference between scripted and exploratory testing in your own
   words.
2. Write an exploratory **charter** for testing a photo-upload feature, and
   time-box it.
3. Pick any app and do a 20-minute session-based exploration. Keep notes; list
   every bug, oddity, or confusing moment you find.
4. When would you prefer scripted testing over exploratory testing?

Next: [Black, Grey, and White Box Testing](06-black-grey-white-box.md)
