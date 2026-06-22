# Writing an Effective Bug Report

Finding a bug is only half the job. If you can't *describe* it clearly, the
developer can't fix it — and you'll get the dreaded "cannot reproduce" reply. A
great bug report is the most visible product of your work, and writing one well is a
genuine skill. This lesson gives you the template and habits to do it right every
time.

## What You'll Learn

- The parts of a good bug report
- The golden rule: reproducibility
- How to write clear steps to reproduce
- Common mistakes that get bugs rejected

## The Golden Rule: Make It Reproducible

The #1 job of a bug report is to let *someone else* recreate the problem on *their*
machine. If a developer can't reproduce it, they usually can't fix it. Everything
below serves this one goal.

## The Anatomy of a Bug Report

| Field | What goes here |
| --- | --- |
| **Title / Summary** | A short, specific one-liner |
| **Description** | A brief overview of the problem |
| **Steps to Reproduce** | Numbered, exact steps to make it happen |
| **Expected Result** | What *should* have happened |
| **Actual Result** | What *actually* happened |
| **Environment** | Browser, device, OS, app version, URL |
| **Severity / Priority** | How bad / how urgent (next lesson) |
| **Attachments** | Screenshot, screen recording, logs |

## Writing a Great Title

The title is what everyone scans first. Make it specific.

- ❌ "Login broken" — vague, useless.
- ❌ "It doesn't work" — meaningless.
- ✅ "Login fails with valid credentials on Safari — shows 'Server Error 500'"

A good title says **what**, **where**, and ideally a hint of the symptom.

## Steps to Reproduce (the heart of the report)

Write numbered steps so exact that a stranger could follow them blindly:

```text
1. Go to https://app.example.com/login
2. Enter email: test@example.com
3. Enter password: Correct123!
4. Click the "Login" button

Expected: Dashboard loads and shows "Welcome".
Actual:   Page shows "Server Error 500" and stays on the login screen.
```

Tips:

- Start from a **known point** (a URL or screen).
- Include the **exact data** you used.
- One action per step.
- Note anything special about the **state** (logged in as admin? cart already had
  items?).

## Expected vs Actual — Always Both

State them **side by side**. This single contrast is what turns "something's weird"
into a clear, actionable defect:

- **Expected:** "An order confirmation with an order number appears."
- **Actual:** "The page goes blank and no order is created."

Without *expected*, the developer doesn't know what "correct" even looks like.

## Don't Forget the Environment

The same app can work in Chrome and fail in Safari, or work on desktop and fail on
mobile. Always capture:

- Browser + version (Chrome 120, Safari 17)
- Device / OS (iPhone 14, Windows 11)
- App version or build number
- The exact URL
- The user/account type (if relevant)

This is often *the* clue that cracks the bug.

## Attach Evidence

A picture is worth a thousand words — a screen recording, ten thousand.

- **Screenshot** of the error (annotate it if you can).
- **Screen recording** for anything involving steps or timing.
- **Console/network logs** if you know how to grab them (a nice bonus).

## A Complete Example

> **Title:** Checkout — "Place Order" button does nothing on Firefox
>
> **Description:** On Firefox, clicking "Place Order" on the checkout page has no
> effect. No order is created and no error is shown.
>
> **Steps to Reproduce:**
> 1. Log in as a standard user.
> 2. Add any product to the cart.
> 3. Go to checkout and fill in valid shipping + payment details.
> 4. Click **Place Order**.
>
> **Expected:** Order is placed; a confirmation page with an order number appears.
> **Actual:** Nothing happens. Button appears to click but no order is created.
> **Environment:** Firefox 121, Windows 11, app build 4.2.0, /checkout
> **Severity:** High **Priority:** High
> **Attachments:** screen-recording.mp4, console-log.txt

## Mistakes That Get Bugs Rejected

- ❌ Vague title ("page broken").
- ❌ Missing or unclear steps → "cannot reproduce."
- ❌ No expected vs actual.
- ❌ No environment info.
- ❌ Emotional or blaming language ("this is terrible") — stay factual and neutral.
- ❌ Reporting multiple unrelated bugs in one ticket → file them separately.

## Exercises

1. List the eight standard fields of a bug report.
2. Rewrite this bad report into a good one: *"Search is broken, fix it."* Invent
   reasonable details.
3. Why must every bug report include *both* expected and actual results?
4. Pick any app, find a real bug or oddity, and write a complete bug report for it
   using the template.

Next: [Severity vs Priority](03-severity-vs-priority.md)
