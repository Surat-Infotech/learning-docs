# Non-Functional Testing

Functional testing asks *"does it work?"* Non-functional testing asks *"does it work
**well**?"* A login page can log you in correctly (functional ✅) but take 30
seconds, look broken on a phone, and leak your password (non-functional ❌). These
"how well" qualities often decide whether people actually *like* your software.

## What You'll Learn

- What non-functional testing covers
- The main non-functional types: performance, usability, security, compatibility,
  reliability
- What a manual tester can realistically check vs what needs specialists/tools
- Practical things to look for

## The "-ilities"

Non-functional testing checks qualities that often end in "-ility": usabil**ity**,
reliabil**ity**, compatibil**ity**, security, performance, and so on. Here are the
big ones.

### Performance Testing — "Is it fast enough?"

How the system behaves under load and speed conditions.

- **Sub-types you'll hear:**
  - **Load testing** — behavior under expected traffic (e.g., 1,000 users).
  - **Stress testing** — push *past* the limit to see where it breaks.
  - **Spike testing** — sudden traffic surge (a flash sale).
- **Manual tester's reality:** heavy performance testing uses tools (JMeter,
  k6). But you can *notice* and report slowness: "the search takes 8 seconds — feels
  broken."

### Usability Testing — "Is it easy and pleasant to use?"

Can a real person accomplish their goal without confusion or frustration?

- **What to look for:** confusing labels, hidden buttons, too many steps, unclear
  error messages, inconsistent design.
- **Manual tester's reality:** this is *very* doable by hand. Watch real people use
  it; note every hesitation. Your fresh eyes are valuable.

### Security Testing — "Is user data safe?"

Checks for vulnerabilities that could expose data or let attackers in.

- **Basic things even a manual tester can check:**
  - Can you reach an admin page while logged out, or as a normal user?
  - Are passwords hidden on screen and not shown in the URL?
  - Does the session expire after logout (press Back — are you still in)?
- **Manual tester's reality:** deep security testing (penetration testing) is a
  specialist field. But spotting obvious leaks is part of your job.

### Compatibility Testing — "Does it work everywhere?"

Does the software behave correctly across different environments?

- **Dimensions:** browsers (Chrome, Safari, Firefox), devices (phone, tablet,
  desktop), operating systems, screen sizes.
- **Manual tester's reality:** very doable. Open the same page in several browsers
  and on a phone; compare. We dig into this in
  [Module 5](../05-tools-and-process/04-cross-browser-and-mobile.md).

### Reliability / Stability — "Does it keep working over time?"

Does the app stay healthy during long or repeated use, or does it slow down, leak
memory, or crash?

- **What to look for:** crashes after extended use, errors that pile up, features
  that degrade the longer you stay on a page.

## Functional vs Non-Functional, Side by Side

| Feature | Functional check | Non-functional check |
| --- | --- | --- |
| Login | Correct password logs you in | Login completes in under 2 seconds |
| Search | Returns matching products | Results render quickly, look right on mobile |
| Checkout | Order is created | Page is intuitive; payment data is secure |

Same feature — two different lenses. A complete tester uses **both**.

## What's Realistic for a Manual Tester?

| Type | Manual-friendly? | Notes |
| --- | --- | --- |
| Usability | ✅ Yes | Your strongest non-functional area |
| Compatibility | ✅ Yes | Just open it in different browsers/devices |
| Basic security | ⚠️ Partly | Spot obvious leaks; specialists do deep work |
| Performance | ⚠️ Partly | Notice slowness; tools do real load testing |

Don't feel you must master load-testing tools to start. **Notice, describe, and
report** non-functional problems clearly — that alone is hugely valuable.

## Exercises

1. Give a functional and a non-functional test for the same feature: "upload a
   profile photo."
2. Classify each: (a) login takes 10 seconds, (b) the Submit button does nothing,
   (c) text is unreadable on a phone, (d) the page works in Chrome but not Safari.
3. List three basic security checks a manual tester can do *without* special tools.
4. Pick any website. Note two usability problems and one compatibility problem you
   can find in 10 minutes.

Next: [Regression and Retesting](03-regression-and-retesting.md)
