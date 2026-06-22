# Writing Good Test Cases

Writing test cases is the single most practiced skill in manual QA. A good test case
is so clear that **anyone** — a new teammate, a developer, you six months from now —
can run it and get the same result without asking questions. This lesson gives you a
reusable template and the habits that separate sloppy cases from professional ones.

## What You'll Learn

- The standard parts of a test case
- A template you can reuse forever
- The qualities of a *good* test case
- Common mistakes beginners make

## The Anatomy of a Test Case

A complete test case usually has these fields:

| Field | What it is |
| --- | --- |
| **Test Case ID** | A unique label (e.g., `TC-LOGIN-001`) for tracking |
| **Title / Summary** | A short description of what's being tested |
| **Preconditions** | What must be true *before* you start (logged in, account exists, data present) |
| **Test Steps** | Numbered, exact actions to perform |
| **Test Data** | The specific inputs to use |
| **Expected Result** | Precisely what *should* happen |
| **Actual Result** | What *did* happen (filled in during execution) |
| **Status** | Pass / Fail / Blocked (filled in during execution) |
| **Priority** | How important this test is (High/Medium/Low) |

The first six are written *before* testing; **Actual Result** and **Status** are
filled in *while* testing.

## A Reusable Template

```text
Test Case ID:    TC-______
Title:           ________________________________
Priority:        High / Medium / Low
Preconditions:   ________________________________

Steps:
  1. ____________________________________________
  2. ____________________________________________
  3. ____________________________________________

Test Data:       ________________________________
Expected Result: ________________________________
Actual Result:   ____________________ (run time)
Status:          Pass / Fail / Blocked (run time)
```

## A Worked Example

| Field | Value |
| --- | --- |
| **ID** | TC-LOGIN-002 |
| **Title** | Login fails with an incorrect password |
| **Priority** | High |
| **Preconditions** | Account `bob@example.com` exists and is active |
| **Steps** | 1. Open the login page. 2. Enter `bob@example.com`. 3. Enter `WrongPass99`. 4. Click **Login**. |
| **Test Data** | `bob@example.com` / `WrongPass99` |
| **Expected Result** | An error "Incorrect email or password" appears; user stays on the login page; user is **not** logged in. |
| **Actual Result** | *(filled at run time)* |
| **Status** | *(filled at run time)* |

## What Makes a Test Case *Good*

- **Clear and unambiguous.** No room for interpretation. "Click the blue **Login**
  button," not "log in somehow."
- **One thing per case.** Test a single behavior. Don't cram login *and* logout
  *and* password reset into one case — if it fails, you won't know which part broke.
- **Specific expected result.** "It works" is useless. State exactly what should
  appear, change, or be saved.
- **Repeatable.** Running it twice gives the same result.
- **Includes the data.** Don't say "enter a password" — say *which* password.
- **Independent where possible.** A case shouldn't depend on another case having run
  first (or if it must, say so in preconditions).
- **Traceable.** Linked back to a requirement, so you know *why* it exists (next
  lessons cover this).

## The "Status" Values

When you run a case, you mark it:

- **Pass** — actual result matched expected result.
- **Fail** — it didn't. Log a bug (Module 4).
- **Blocked** — you *couldn't* run it (e.g., the page won't even load, or a
  precondition isn't met). Not a pass, not a fail — blocked.

## Common Beginner Mistakes

- ❌ Vague steps ("test the form") → ✅ exact, numbered actions.
- ❌ Vague expected result ("should work") → ✅ "an order confirmation with a number
  is shown."
- ❌ Testing five things in one case → ✅ one behavior per case.
- ❌ No test data → ✅ exact inputs listed.
- ❌ Assuming context ("the usual user") → ✅ spell it out in preconditions.

## Exercises

1. Using the template, write a complete test case for "successfully adding an item
   to a shopping cart."
2. Take this bad step — *"Check that search works"* — and rewrite it as clear,
   numbered steps with a specific expected result.
3. Explain why "one behavior per test case" matters when a case fails.
4. When would you mark a test **Blocked** instead of **Fail**? Give an example.

Next: [Test Case Design Techniques](03-test-design-techniques.md)
