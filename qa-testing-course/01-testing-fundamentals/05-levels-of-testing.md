# The Levels of Testing

Software is tested at different "zoom levels" — from a single tiny piece all the
way up to the whole system as a user sees it. These are the **levels of testing**.
As a manual QA tester you'll live mostly at the top levels, but you should
understand the whole ladder so you know who tests what.

## What You'll Learn

- The four levels: unit, integration, system, acceptance
- Who usually performs each level
- Where manual QA spends most of its time
- The classic "testing pyramid" idea

## The Four Levels

```text
        ▲  zoomed OUT (whole product, user's view)
        │
   4. Acceptance Testing    ← "Does it meet the user's needs?"  (business / users)
   3. System Testing        ← "Does the whole app work together?" (QA testers)
   2. Integration Testing   ← "Do the pieces work together?"      (devs + QA)
   1. Unit Testing          ← "Does this one small piece work?"   (developers)
        │
        ▼  zoomed IN (one tiny piece)
```

### 1. Unit Testing (zoomed all the way in)

Tests a single, tiny piece of code in isolation — one function, one button's logic.

- **Who:** developers (it requires reading/writing code).
- **Example:** a test that checks the function `add(2, 3)` returns `5`.
- **Manual QA role:** usually none directly, but good unit tests mean fewer dumb
  bugs reach you.

### 2. Integration Testing (do the pieces fit?)

Checks that separate pieces work **together**. Individually they're fine — but does
data flow correctly *between* them?

- **Who:** developers and QA.
- **Example:** the login screen correctly passes the username to the user-profile
  service, which returns the right profile.
- **Common bugs found:** mismatched data formats, broken handoffs between parts.

### 3. System Testing (the whole app)

Tests the **complete, integrated application** as a whole, in a realistic
environment — the way a user would experience it end to end.

- **Who:** **QA testers** — this is your home turf.
- **Example:** complete the full journey: register → log in → add item to cart →
  check out → receive confirmation email.
- This is where most manual test cases run.

### 4. Acceptance Testing (is it acceptable to ship?)

Confirms the software meets **business requirements and user needs** — the final
"yes, this is what we wanted" check.

- **Who:** the business, product owners, or actual end users.
- **Types you'll hear about:**
  - **UAT (User Acceptance Testing)** — real users try it before launch.
  - **Alpha testing** — internal, before release.
  - **Beta testing** — a limited set of outside users try it in the real world.

## Where Does Manual QA Live?

Mostly at **System** and **Acceptance** levels — testing the whole product the way
a human uses it. You'll *benefit* from solid unit and integration testing done by
developers (fewer bugs reach you), but your hands-on focus is the complete
experience.

## The Testing Pyramid (a quick mention)

You'll see this diagram everywhere. It suggests teams should have **many** fast,
cheap unit tests at the bottom, **fewer** integration tests in the middle, and
**few** slow, expensive end-to-end tests at the top.

```text
        /\        few    ← end-to-end / UI tests (slow, expensive)
       /  \
      /----\     some    ← integration tests
     /      \
    /--------\   many     ← unit tests (fast, cheap)
```

Why it matters to you: end-to-end tests (often the manual ones you run) are
valuable but slow, so teams try not to rely on them for *everything*. Knowing this
helps you talk sensibly with developers about test strategy.

## Exercises

1. List the four levels from most zoomed-in to most zoomed-out, with who performs
   each.
2. A user can log in, and the profile page loads correctly, but the *email
   notification* never sends. Which level of testing would most likely catch this,
   and why?
3. What's the difference between alpha and beta testing?
4. In the testing pyramid, why are there *fewer* end-to-end tests than unit tests?

Next: [Module 2 — Types of Testing](../02-types-of-testing/01-functional-testing.md)
