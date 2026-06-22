# Pull Requests

Imagine you wrote a new chapter for a shared book and, instead of pasting it
straight in, you slip it under your editor's door with a note: "Here's my
chapter, please review and add it if it looks good." That note is a **Pull
Request**. It is a polite, reviewable way to propose adding your changes to the
main project, rather than shoving them in directly.

Pull Requests are how almost every team works on GitHub. Learning them well makes
you a pleasant teammate.

## What You'll Learn

- What a Pull Request (PR) is and why teams use them
- The standard flow: branch → push → open PR → review → merge
- How to open a PR on GitHub, step by step
- How to write a clear PR title and description
- The three merge options: merge commit, squash, and rebase
- What to do after merging: closing and deleting the branch

## What Is a Pull Request?

A **Pull Request** (PR) is a request to merge the commits on one branch into
another, usually merging your feature branch into `main`. On GitLab the same
thing is called a **Merge Request (MR)**; it is the identical idea.

A PR is more than a merge button. It is a place where teammates can:

- **See exactly what changed** (a side-by-side "diff" of your edits).
- **Discuss** the changes and leave comments on specific lines.
- **Request changes** before the code goes in.
- **Run automated checks** (tests, linters) before merging.

### Why teams use PRs

- **Quality.** A second pair of eyes catches bugs and typos.
- **Knowledge sharing.** Reviewers learn what is changing in the codebase.
- **A record.** Each PR documents *why* a change was made, not just *what*.
- **Safety.** `main` stays stable because nothing lands without review.

## The Standard Flow

The everyday cycle looks like this:

```text
1. branch    create a feature branch off main
2. push      push that branch to the remote
3. open PR   ask to merge it into main
4. review    teammates comment; you make fixes and push again
5. merge     once approved, merge it into main
6. clean up  delete the branch
```

Let's walk the first two steps in the terminal, then move to the website.

```bash
# 1. Start from an up-to-date main
git switch main
git pull

# Create a focused branch for your work
git switch -c add-contact-form

# ...edit files, then commit...
git add .
git commit -m "Add contact form to homepage"

# 2. Push the branch and set its upstream
git push -u origin add-contact-form
```

After the push, GitHub often prints a handy link to open the PR directly. If not,
the website will offer a button.

## Opening a PR on GitHub

1. Go to your repository on [github.com](https://github.com).
2. GitHub usually shows a yellow banner: **"add-contact-form had recent pushes —
   Compare & pull request."** Click it. (If not, go to the **Pull requests** tab
   and click **New pull request**.)
3. Check the two branches at the top:
   - **base** = where you want the code to go (usually `main`).
   - **compare** = your branch (`add-contact-form`).
4. Fill in the **title** and **description** (see below).
5. Click **Create pull request**.

That's it. Your teammates can now review it.

## Writing a Good Title and Description

A reviewer should understand your PR without reading every line of code. Help
them.

### The title

Keep it short, specific, and in plain language. Imagine it as a headline.

```text
Good:  Add contact form to homepage
Good:  Fix crash when cart is empty
Bad:   updates
Bad:   fixed stuff
```

### The description

Explain the **what** and the **why**. A simple template works well:

```text
## What
Adds a contact form to the homepage with name, email, and message fields.

## Why
Users asked for an easy way to reach support without leaving the site.

## How to test
1. Open the homepage.
2. Fill in the form and click Send.
3. Confirm a success message appears.
```

If your PR fixes a tracked issue, mention it like `Closes #42`. GitHub will
automatically close issue 42 when the PR merges.

## Review: Responding to Feedback

Reviewers may leave comments or "request changes." You do **not** open a new PR.
Instead, fix things on the same branch and push again. The PR updates
automatically.

```bash
# Make the requested fixes, then:
git add .
git commit -m "Use a larger button per review feedback"
git push        # the PR refreshes with your new commit
```

Reply to comments, mark them resolved, and re-request review when ready.

## Merge Options, in Simple Terms

When a PR is approved, GitHub offers three ways to merge. They differ in how the
history ends up looking.

### Merge commit

Keeps every commit from your branch and adds one extra **merge commit** that ties
them together. History shows the full branching story.

```text
main:  A───B────────M   ← M is the merge commit
            \      /
feature:     C──D
```

- **Pro:** nothing is lost; you see exactly how work happened.
- **Con:** history can get busy with many small commits.

### Squash and merge

Combines all your branch's commits into **one tidy commit** on `main`. Your messy
"wip", "fix typo", "oops" commits become a single clean entry.

```text
main:  A───B───S        ← S contains all of C and D's changes, squashed
```

- **Pro:** clean, readable `main` history (one commit per feature).
- **Con:** you lose the individual commit breakdown. Often the preferred default.

### Rebase and merge

Replays your commits one by one directly onto `main`, with no merge commit.

```text
main:  A───B───C'───D'   ← your commits, reapplied in a straight line
```

- **Pro:** linear history, no extra merge commits.
- **Con:** commit hashes change, and it can be confusing for beginners.

If you are unsure, **squash and merge** is a safe, popular choice.

## After the Merge: Clean Up

Once merged, the branch has done its job. Delete it to keep things tidy.

```bash
# Delete the branch on the remote (GitHub also offers a button for this)
git push origin --delete add-contact-form

# Update your local main and delete the local branch
git switch main
git pull
git branch -d add-contact-form
```

GitHub shows a **Delete branch** button right after merging; clicking it does the
remote half for you.

## Common Mistakes

- **Putting work straight on `main`.** The whole point of a PR is to review before
  merging. Always branch first.
- **Giant PRs.** A 50-file PR is painful to review. Keep changes small and
  focused on one thing.
- **Vague titles.** "fixes" tells the reviewer nothing. Describe the change.
- **Opening a new PR to fix feedback.** Just push more commits to the same branch;
  the PR updates itself.
- **Forgetting to pull after merge.** Your local `main` is now behind. Run
  `git switch main && git pull` before starting new work.
- **Leaving dead branches around.** Delete merged branches so the branch list
  stays clean.

## Exercises

1. Create a branch, make a small change, commit, and push it. Open a Pull Request
   on GitHub targeting `main`.
2. Write a PR description using the What / Why / How to test template.
3. Leave a comment on your own PR, then push a new commit and watch the PR update.
4. Merge the PR using **Squash and merge**, then look at `main`'s history to see
   the single combined commit.
5. After merging, delete the branch (both remote and local) and run
   `git pull` on `main` to sync up.

## Key Takeaways

- A **Pull Request** proposes merging your branch into another, with room for
  review and discussion (called a Merge Request on GitLab).
- The flow is: **branch → push → open PR → review → merge → delete branch**.
- Write a clear **title** and a **description** covering what, why, and how to test.
- Respond to review feedback by pushing more commits to the same branch.
- Merge options: **merge commit** (full history), **squash** (one clean commit),
  **rebase** (linear history). Squash is a safe default.
- Delete the branch and pull `main` after merging.

---

[← Previous: Clone, Push, Pull, and Fetch](02-clone-push-pull-fetch.md) | [Back to Course Index](../README.md) | [Next: Forking and Contributing →](04-forking-and-contributing.md)
