# Rewriting History Safely

Git lets you go back and change your past commits — fix a typo in a message,
combine messy commits, or remove a file you never meant to add. This is powerful,
like having an eraser for your project's history. But an eraser can also rub out
work other people are standing on. Think of it like editing a shared document:
editing your *own* draft that nobody has read yet is harmless; editing a page
someone else already printed and built on top of causes confusion. This lesson
teaches you when rewriting is safe, when it is dangerous, and how to do it
without hurting your team.

## What You'll Learn

- What "rewriting history" actually means in Git
- When rewriting is **safe** (local, unshared commits) versus **dangerous**
  (shared branches)
- A quick recap of the three rewrite tools: `amend`, interactive `rebase`, and
  `reset`
- How to remove a secret or a huge file from your entire history
- A high-level look at `git filter-repo` and BFG (with a clear warning)
- The **golden rule** of rewriting history, one more time
- How to communicate a force-push to your teammates so nobody gets hurt

## What "Rewriting History" Means

Every commit has a unique ID (a long hash like `a1b2c3d`). That ID is calculated
from the commit's contents *and* its parent. So when you change anything about a
commit — its message, its files, or its position — Git cannot reuse the old ID.
It creates a **brand-new commit** with a new ID. The old commit is abandoned.

```text
Before:   A ── B ── C        (C is the commit you want to fix)
After:    A ── B ── C'       (C' is a NEW commit; C is gone)
```

This matters because anyone who already has the old commit `C` will be confused
when your branch suddenly has `C'` instead.

## When It Is Safe vs Dangerous

### Safe: commits only you have

If the commits live **only on your computer** — you have not pushed them, or you
pushed them only to a private branch nobody else uses — rewriting is completely
safe. You are editing your own private draft.

```bash
# You committed locally but have NOT run git push yet.
git log --oneline    # these commits exist only here → safe to rewrite
```

### Dangerous: commits other people share

If you have pushed commits to a **shared branch** (like `main`, or a branch a
teammate has pulled), rewriting them forces everyone else to untangle a mismatch.
Their Git still has the old commits; yours now has new ones. Merges get messy and
people can lose work.

```text
You rewrite shared commits → teammates' branches no longer match yours →
they get confusing errors and may accidentally re-add your old commits.
```

A simple test: **"Could someone else already have this commit?"** If yes, do not
rewrite it without talking to them first.

## A Quick Recap of the Rewrite Tools

You met these earlier in the course. Here they are side by side.

### `git commit --amend` — fix the most recent commit

```bash
# Fix the message of the last commit:
git commit --amend -m "Add login validation"

# Add a forgotten file to the last commit:
git add forgotten-file.js
git commit --amend --no-edit    # keep the existing message
```

### `git rebase -i` — edit several recent commits

```bash
# Open an editor to reorder, squash, reword, or drop the last 4 commits:
git rebase -i HEAD~4
```

In the editor you change `pick` to `reword`, `squash`, `edit`, or `drop`. This is
the main tool for cleaning up a messy branch *before* you share it.

### `git reset` — move the branch pointer

```bash
# Undo the last commit but KEEP the changes in your working files:
git reset --soft HEAD~1

# Undo the last commit and THROW AWAY the changes (be careful):
git reset --hard HEAD~1
```

`--soft` keeps your work; `--hard` deletes it. When in doubt, use `--soft`.

## Removing a Secret or Large File From History

Deleting a file in a new commit is **not enough** — the old commits still contain
it. If you committed a password, an API key, or a 500 MB video, it lives in every
snapshot from that point on. You must rewrite history to truly erase it.

> **Warning:** This rewrites every affected commit, changing their IDs. Treat it
> as a major operation. Coordinate with your whole team before doing it on a
> shared repository.

### The modern tool: `git filter-repo`

`git filter-repo` is the recommended tool for surgically removing files or text
from all of history. It is a separate program you install (it is not built into
Git).

```bash
# Remove a file from the ENTIRE history (all commits):
git filter-repo --path secrets.env --invert-paths
```

`--invert-paths` means "keep everything except this path."

### The older tool: BFG Repo-Cleaner

**BFG** is an alternative that is simple and fast for the common cases of deleting
big files or scrubbing secrets.

```bash
# Delete every file named id_rsa from all of history:
bfg --delete-files id_rsa

# Replace any text matching patterns in a file with ***REMOVED***:
bfg --replace-text passwords.txt
```

### The critical follow-up step

Removing a secret from history does **not** make it safe again — it was already
exposed. After scrubbing, you must also:

1. **Rotate the secret** (generate a new key/password; assume the old one is
   compromised).
2. **Force-push** the rewritten history.
3. Tell collaborators to re-clone or carefully reset their copies.

## The Golden Rule (Once More)

> **Never rewrite history that you have shared with others — unless everyone
> involved knows and agrees.**

For your own private branch: rewrite freely. For `main` or any branch teammates
use: leave it alone, or coordinate first.

## Communicating a Force-Push to Your Team

When you rewrite shared history, your next push must be a **force-push** because
the remote branch no longer fast-forwards cleanly.

```bash
# Safer force-push: refuses if someone else pushed in the meantime.
git push --force-with-lease

# Blunt force-push (avoid on shared branches): overwrites no matter what.
git push --force
```

Prefer `--force-with-lease` — it protects you from accidentally erasing a
teammate's new commits you had not seen yet.

Before you force-push a shared branch, tell your team something like:

```text
"Heads up: I rewrote history on the feature/login branch to remove a leaked
key. Please stop pushing to it, then run:
    git fetch origin
    git reset --hard origin/feature/login
to sync. Sorry for the disruption!"
```

A 30-second message saves hours of confusion.

## Common Mistakes

- **Rewriting `main` because the history "looks messy."** Tidiness is not worth
  breaking everyone's clones. Only rewrite shared branches for real reasons, with
  agreement.
- **Thinking a delete-commit removes a secret.** The secret still lives in older
  commits. You must rewrite history *and* rotate the secret.
- **Using `git push --force` on a shared branch.** Use `--force-with-lease` so you
  do not silently wipe a teammate's work.
- **Forgetting to tell the team.** A surprise force-push leaves people with broken
  branches and no idea why.
- **Running `git reset --hard` without thinking.** It throws away uncommitted
  work permanently. Check `git status` first.

## Exercises

1. In a throwaway practice repo, make three commits. Use `git rebase -i HEAD~3`
   to squash them into a single commit, then run `git log --oneline` to confirm
   the IDs changed.
2. Commit a file called `notes.txt`, then use `git commit --amend` to add a second
   file to that same commit without creating a new one. Verify with `git show`.
3. Explain in one sentence, in your own words, why rewriting a pushed `main`
   branch is dangerous but rewriting an unpushed local branch is fine.
4. Write the exact `git push` command you would use to safely force-push after a
   rewrite, and explain why you chose it over the alternative.
5. Draft a short message you would send your team before force-pushing a shared
   branch, including the commands they should run to sync.

## Key Takeaways

- Rewriting a commit always creates a **new commit with a new ID**; the old one is
  abandoned.
- Rewriting is **safe** for local, unshared commits and **dangerous** for commits
  others already have.
- `amend`, `rebase -i`, and `reset` are your rewrite tools; `--soft` keeps work,
  `--hard` deletes it.
- To truly remove a secret or big file, rewrite **all** history with
  `git filter-repo` or **BFG**, then **rotate the secret**.
- The golden rule: **never rewrite shared history without everyone's agreement.**
- When you must force-push a shared branch, use `--force-with-lease` and **tell
  your team** how to sync.

---

[← Previous: Finding Bugs with git bisect](../05-advanced-git/06-bisect.md) | [Back to Course Index](../README.md) | [Next: Git Hooks →](02-git-hooks.md)
