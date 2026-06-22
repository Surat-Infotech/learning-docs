# Code Review and Pull Request Etiquette

Imagine a chef sending out a new dish. Before it reaches the customer, a second cook tastes it: "A little more salt, and plate it neater." That quick taste-test catches problems while they're cheap to fix. **Code review** is that taste-test for software — a teammate reads your changes before they reach the shared project.

The place this happens is the **Pull Request** (PR) — a request to merge your branch into another branch, bundled with a discussion thread and automated checks. In this lesson we'll learn how to open a PR people *enjoy* reviewing, and how to be a kind, useful reviewer yourself.

## What You'll Learn

- Why code review matters beyond just "catching bugs"
- How to open a small, focused, easy-to-review PR
- Writing a good PR description and linking the related issue
- Responding gracefully to review comments
- Giving feedback that's kind *and* useful
- What **draft PRs** are and when to use them
- What **CI checks** are and why you should wait for them
- **Squashing** commits before merge for a clean history

## Why Code Review Matters

Code review isn't about distrust. A second pair of eyes:

- **Catches bugs and edge cases** the author missed.
- **Spreads knowledge** so more than one person understands each part of the code.
- **Keeps quality consistent** across the team (style, patterns, naming).
- **Mentors people** — both author and reviewer learn from each exchange.
- **Creates a record** of *why* a change was made, useful months later.

The goal is a better codebase and a stronger team, not finding fault.

## Opening a Reviewable PR

The single biggest favor you can do a reviewer is to **keep the PR small and focused**. A 50-line PR gets a thoughtful review in minutes; a 2,000-line PR gets a tired "looks good" that catches nothing.

### Keep It Small and Focused

- One PR = one logical change. Don't mix a bug fix with a big refactor.
- If a change is large, split it into a series of smaller PRs.

```text
GOOD:  "Add email validation to signup form"   (focused, ~40 lines)
BAD:   "Signup form + redesign navbar + update 12 deps"  (please no)
```

### Write a Good Description

The description tells the reviewer *what* changed and *why*. A simple template:

```text
## What
Adds email validation to the signup form.

## Why
Users were submitting invalid emails (issue #214), causing failed sends.

## How to test
1. Open /signup
2. Enter "notanemail" and submit
3. You should see an inline error message
```

### Link the Issue

If your work relates to a tracked task or bug, link it. On GitHub, writing `Closes #214` in the description will automatically close that issue when the PR merges.

```text
Closes #214      # auto-closes the linked issue on merge
Related to #198  # references without closing
```

## Draft PRs

A **draft PR** is a Pull Request marked as "not ready yet." It signals: *please don't merge this, but feel free to glance.* Use a draft when you want early feedback on direction, or to let CI run while you keep working.

```text
[Draft] Add email validation   ← work in progress, do not merge
```

When it's ready, you click "Ready for review" to flip it into a normal PR.

## CI Checks

**CI** stands for **Continuous Integration** — an automated system that runs your tests, linters, and builds every time you push. On a PR, these appear as a list of checks with green ticks or red crosses.

```text
✓ build         passed
✓ unit tests    passed
✗ lint          failed   ← fix this before asking for review
```

Etiquette: **wait for CI to pass before requesting review.** Don't make a human read code that the machine already knows is broken. If a check fails, read its log, fix the issue, and push again.

## Responding to Review Comments

Receiving feedback can sting, but reviewers are helping you. Some habits that keep things smooth:

- **Assume good intent.** A comment like "Why this approach?" is curiosity, not an attack.
- **Reply to every comment.** Either make the change, or explain why you didn't. Don't leave threads silent.
- **Make changes in new commits**, push them, and let the reviewer see what you addressed. Mark threads resolved once handled.
- **Ask questions** if a comment is unclear. "Can you suggest what you'd prefer here?" is perfectly fine.

```bash
# After addressing feedback:
git add .
git commit -m "Address review: extract validation helper"
git push   # the PR updates automatically
```

## Giving Kind, Useful Feedback

When *you* are the reviewer, your words shape how the team feels about review. Aim for feedback that improves the code without bruising the person.

- **Comment on the code, not the coder.** "This loop could be a `map`," not "you always overcomplicate things."
- **Ask, don't command.** "What do you think about pulling this into a function?" invites discussion.
- **Praise the good parts.** "Nice clean naming here" makes review a positive experience.
- **Separate must-fix from nice-to-have.** Prefix optional ideas with `nit:` (a small, non-blocking nitpick).

```text
nit: could rename `x` to `userCount` for clarity   ← optional
blocking: this crashes when the list is empty       ← must fix
```

- **Review promptly.** A PR left for days blocks your teammate. Aim to respond within a day.

## Squashing Before Merge

While building a feature you might make messy commits: "wip", "fix typo", "actually fix it". **Squashing** combines them into one clean commit when merging, so `main`'s history stays tidy.

```text
Before squash (on your branch):
  ● add validation
  ● wip
  ● fix typo
  ● address review

After squash (on main):
  ● Add email validation to signup form (#214)
```

On GitHub, choose **"Squash and merge"** to do this with one click. The result: one clear commit per feature in `main`, which makes history easy to read and easy to undo.

## Common Mistakes

- **Giant PRs.** Huge changes get rubber-stamped. Keep them small and reviewable.
- **No description.** A title alone forces the reviewer to reverse-engineer your intent. Always explain what and why.
- **Requesting review on red CI.** Fix the automated checks first.
- **Taking feedback personally.** It's about the code. Breathe, then reply.
- **Harsh reviewer comments.** "This is wrong" helps no one. Be specific and kind.
- **Ignoring comment threads.** Reply to each one so the reviewer knows it's handled.

## Exercises

1. Write a PR description (using the What / Why / How to test template) for an imaginary change that adds a "dark mode" toggle to a website.
2. Rewrite this unkind review comment to be kind and useful: "This code is a mess, did you even test it?"
3. Explain in your own words the difference between a draft PR and a normal PR, and give one situation where you'd use a draft.
4. A teammate's PR has a failing lint check. Write the short, friendly message you'd leave on their PR.
5. Describe what "Squash and merge" does and why a team might prefer it for keeping `main`'s history clean.

## Key Takeaways

- **Code review** improves quality, spreads knowledge, and mentors the team — it's not about distrust.
- The best PRs are **small, focused, well-described, and linked** to their issue.
- Use **draft PRs** for work in progress and early feedback.
- **CI checks** run tests and linters automatically — wait for them to pass before requesting review.
- Respond to every review comment; assume good intent and reply with questions when unsure.
- As a reviewer, critique the code (not the person), ask rather than command, praise the good, and review promptly.
- **Squashing** before merge keeps `main`'s history to one clean commit per feature.

---

[← Previous: Branching Strategies](01-branching-strategies.md) | [Back to Course Index](../README.md) | [Next: Keeping Your Branch Up to Date →](03-keeping-your-branch-updated.md)
