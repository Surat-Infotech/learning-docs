# The Defect Life Cycle

When you find a bug, it doesn't just get "fixed" and disappear. It travels through a
series of **statuses** — from the moment you report it to the moment it's verified
closed. This journey is the **defect (or bug) life cycle**, and every QA team tracks
it. Knowing the stages tells you exactly what's happening to your bug at any time.

## What You'll Learn

- What a "defect" is (and the related vocabulary)
- The standard statuses a bug moves through
- What each status means and who acts on it
- The tricky statuses: rejected, duplicate, deferred, reopened

## First, the Vocabulary

People use several words that overlap. The common distinction:

- **Error / Mistake** — a human slip (a developer typed `>` instead of `<`).
- **Defect / Bug / Fault** — the flaw in the software that resulted (the discount
  applies at the wrong amount).
- **Failure** — what the *user* experiences when the defect runs (they get charged
  the wrong price).

In everyday work, **"bug"** and **"defect"** are used interchangeably. Don't
overthink it.

## The Life Cycle, Step by Step

```text
   NEW ──▶ ASSIGNED ──▶ OPEN (in progress) ──▶ FIXED ──▶ RETEST ──┬──▶ CLOSED
    │                                                              │
    │                                                              └──▶ REOPENED ──┐
    │                                                                              │
    └──▶ REJECTED / DUPLICATE / DEFERRED                          (back to ASSIGNED)
```

### 1. New

You found a bug and logged it. Status = **New**. Nobody has reviewed it yet.

### 2. Assigned

A lead or developer reviews it, accepts it as valid, and assigns it to a developer
to fix.

### 3. Open / In Progress

The developer is actively working on the fix.

### 4. Fixed (a.k.a. Resolved)

The developer believes they've fixed it and marks it so. **Important:** "fixed" is
the *developer's* claim — it is **not** confirmed yet. That's your job next.

### 5. Retest

QA (you) re-runs the original steps on the new build to verify the fix. (This is the
**retesting** from
[Module 2](../02-types-of-testing/03-regression-and-retesting.md).)

### 6. Closed

Your retest passed — the bug is genuinely gone. Status = **Closed**. 🎉 This is the
happy ending.

### 7. Reopened

Your retest **failed** — the bug is still there (or came back). You **reopen** it,
and it goes back to the developer. This happens more than beginners expect, which is
exactly why QA must verify fixes rather than trust them.

## The "Not a Straightforward Fix" Statuses

Not every bug marches straight to Closed. A reviewer might set:

| Status | Meaning | Example |
| --- | --- | --- |
| **Rejected** | Not considered a valid bug | "That's how it's designed to work." |
| **Duplicate** | Already reported elsewhere | Linked to the existing bug. |
| **Deferred** | Real, but fix postponed to later | Minor cosmetic issue; fix next release. |
| **Not Reproducible** | Dev can't make it happen | Needs more detail/steps from you. |

These aren't failures on your part — they're normal triage outcomes. But two of them
are signals to *you*:

- **Not Reproducible** usually means your bug report lacked detail. Improve your
  steps (next lesson!).
- **Rejected as "works as designed"** sometimes means a requirement was unclear —
  worth a conversation.

## Who Owns the Bug at Each Stage?

| Status | Owner |
| --- | --- |
| New | QA (you just logged it) |
| Assigned / Open / Fixed | Developer |
| Retest | QA (you) |
| Closed / Reopened | QA (you decide) |

Notice QA **opens** and **closes** the bug. The developer fixes it in the middle.
You're the bookends — you decide a bug exists, and you decide it's truly gone.

## Exercises

1. Draw the defect life cycle from memory, including the Reopened path.
2. Explain why a "Fixed" status is **not** the same as "Closed."
3. A developer marks your bug **Not Reproducible**. What probably went wrong, and
   what will you do about it?
4. Give an example of a bug that might reasonably be marked **Deferred**.

Next: [Writing an Effective Bug Report](02-writing-bug-reports.md)
