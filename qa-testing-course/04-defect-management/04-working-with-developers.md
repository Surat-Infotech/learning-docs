# Working with Developers and Triage

QA and development are sometimes painted as rivals — testers "attacking" the
developers' work. That's a toxic and false picture. You're on the **same team** with
the **same goal**: a great product. How you communicate decides whether QA is a
respected partner or a resented gatekeeper. This lesson is about the human side of
the job, plus how bug **triage** works.

## What You'll Learn

- The right mindset for QA–developer collaboration
- How to report bugs without creating friction
- What "triage" is and how it works
- How to handle disagreements professionally

## You're on the Same Team

Developers are not your opponents. A bug you find is *not* a personal attack on the
developer — it's a problem with the *product* that you both want fixed. Internalize
this and everything else gets easier.

Two attitudes to avoid:

- ❌ **"Gotcha!" testing** — gleefully proving developers wrong. Breeds resentment.
- ❌ **Soft-pedaling** — hiding or downplaying bugs to avoid conflict. Lets defects
  ship.

The healthy middle: **report problems clearly, factually, and kindly.** Hard on the
*bug*, easy on the *person*.

## Report the Product, Not the Person

A small wording shift changes the whole tone:

- ❌ "You broke the login page." (blames the person)
- ✅ "The login page returns a 500 error with valid credentials." (describes the
  product)

Keep bug reports **neutral and factual** (you already learned this in the
[bug report lesson](02-writing-bug-reports.md)). No "this is terrible," no sarcasm.
Just: here's what I did, here's what I expected, here's what happened. Facts don't
feel like accusations.

## What Is Triage?

**Bug triage** is a regular meeting (borrowing the term from emergency medicine,
where patients are sorted by urgency) where the team reviews new bugs and decides:

1. **Is it a real bug?** (or "works as designed," or a duplicate)
2. **How severe / urgent is it?** (set/confirm
   [severity and priority](03-severity-vs-priority.md))
3. **Who fixes it, and when?** (assign and schedule)

Who's usually there: a QA lead, a developer lead, and a product owner/manager. They
look at each new bug and route it.

```text
   New bugs ──▶  TRIAGE MEETING  ──┬──▶ Accept → assign + prioritize → developer
                                   ├──▶ Reject → "works as designed"
                                   ├──▶ Duplicate → link to existing
                                   └──▶ Defer → fix in a later release
```

Your job in triage: **explain the bug clearly and advocate for its real-world
impact** — "this happens to *every* user on checkout, not an edge case." You bring
the user's perspective into the room.

## Handling Disagreement Professionally

Sometimes a developer will say "that's not a bug" or "that's by design." That's
normal. How to handle it:

- **Go back to the requirement.** "The spec says X; the app does Y." Facts settle
  most disputes. (This is where a
  [traceability matrix](../03-test-design-and-documentation/05-traceability-matrix.md)
  earns its keep.)
- **If the requirement is unclear,** that's a *finding*, not a fight — raise it with
  the product owner to clarify.
- **Focus on user impact,** not on being right. "A real user *will* hit this" is
  more persuasive than "I'm correct."
- **Stay calm and curious.** "Help me understand why it's designed this way" beats
  "you're wrong."
- **Escalate respectfully** only when needed, and never make it personal.

## Small Habits That Build Trust

- **Reproduce before reporting.** Confirm it happens reliably; note if it's
  intermittent.
- **Give enough detail** that the developer doesn't have to come ask you.
- **Say thanks** when a fix lands and verify it promptly (don't let "Fixed" bugs sit
  unretested).
- **Share praise too.** Mention what works well, not only what's broken.
- **Pair up** on tough bugs — sit together, reproduce live. It's faster and builds
  rapport.

A tester developers *want* to work with — clear, fair, helpful — is far more
effective than one they avoid.

## Exercises

1. Rewrite this message to be professional and product-focused: *"You completely
   broke the cart again, it's a mess."*
2. What three questions does a triage meeting answer about each new bug?
3. A developer says your bug is "working as designed," but you disagree. Describe,
   step by step, how you'd handle it professionally.
4. List three small habits that make developers trust and respect QA.

Next: [Module 5 — Tools and Process](../05-tools-and-process/01-test-management-tools.md)
