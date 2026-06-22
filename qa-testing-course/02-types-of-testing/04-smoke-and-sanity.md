# Smoke and Sanity Testing

Before you spend a whole day deeply testing a new build, you want a fast answer to
one question: **"Is this build even worth testing?"** If the app won't start or
login is broken, there's no point running 300 detailed tests. **Smoke** and
**sanity** testing are the quick gut-checks that save you from wasting time on a
broken build.

## What You'll Learn

- What smoke testing is and where the name comes from
- What sanity testing is
- The difference between the two
- How they fit into a normal testing day

## Smoke Testing — "Does it turn on?"

A **smoke test** is a quick, shallow check that the **most important features work
at all**, run right after a new build arrives. It's broad but not deep.

The name comes from hardware: engineers would power on a new device and see if smoke
came out. No smoke → keep testing. Smoke → stop, it's broken.

- **Goal:** decide if the build is **stable enough to test further**.
- **Scope:** broad and shallow — touch the critical paths only.
- **Example smoke test for a shop:** app loads → user can log in → a product page
  opens → an item adds to cart. If any of these fail, **reject the build** and send
  it back; don't bother with detailed testing yet.

Smoke testing is sometimes called **"build verification testing."** It's often the
*first* thing you do when a new build lands.

## Sanity Testing — "Does this specific fix make sense?"

A **sanity test** is a quick, narrow check focused on a **specific area** after a
small change or bug fix — just enough to confirm it behaves rationally before
investing in full testing.

- **Goal:** confirm a particular fix/feature works sensibly, so deeper testing
  isn't a waste.
- **Scope:** narrow and (somewhat) deeper — focused on one area.
- **Example:** a dev fixed the discount calculation. You quickly try two or three
  discount codes to confirm the math is now sane. If it's still wildly wrong, stop
  and send it back.

## Smoke vs Sanity

| | Smoke | Sanity |
| --- | --- | --- |
| **Question** | Do the core features work *at all*? | Does this *specific* change behave sensibly? |
| **Scope** | Broad, shallow (whole app's basics) | Narrow, focused (one area) |
| **When** | After every new build | After a small change or fix |
| **Documented?** | Often scripted as a checklist | Often quick and unscripted |
| **Analogy** | "Does the car start?" | "I fixed the brakes — do they grip now?" |

Memory aid: **Smoke is wide and shallow; sanity is narrow and focused.**

## Where They Fit in a Testing Day

```text
   New build arrives
        │
        ▼
   1. SMOKE test  ──fail──▶  Reject build, tell the team, stop. Don't waste a day.
        │ pass
        ▼
   2. SANITY check the changed areas  ──fail──▶  Send back the specific fix.
        │ pass
        ▼
   3. Detailed functional + regression testing  (the deep work)
```

Both are **filters**. They protect your time: never run a full day of detailed tests
on a build that can't even log in. Catch that in five minutes with a smoke test.

## A Note on Confusing Terminology

Different teams use "smoke" and "sanity" a little differently, and some use the
terms interchangeably. Don't get hung up on the dictionary definitions — understand
the *two ideas* (broad quick check vs focused quick check) and match your team's
local usage.

## Exercises

1. Define smoke and sanity testing in one sentence each.
2. Write a five-step smoke test for an email app (the critical paths only).
3. A developer says they fixed the "forgot password" email. Describe a sanity check
   you'd run before deeper testing.
4. Why is it wasteful to run a full regression suite *before* doing a smoke test?

Next: [Exploratory and Ad-hoc Testing](05-exploratory-and-adhoc.md)
