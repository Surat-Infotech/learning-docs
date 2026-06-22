# Undoing Changes Safely

Everyone makes mistakes — that's exactly why Git exists. The good news: almost
anything can be undone. Think of Git as having a generous undo button, plus a few
"emergency exits." This lesson gives you a safe, beginner-friendly toolkit so you
can fix mistakes with confidence and without losing work.

## What You'll Learn

- How to throw away changes in your working directory with `git restore`
- How to unstage a file with `git restore --staged`
- How to fix your most recent commit with `git commit --amend`
- How to safely undo an old commit with `git revert`
- A gentle introduction to `git reset` and the danger of `--hard`
- A decision table: "I want to..." → the command to use

## First, a Safety Rule

Before undoing anything, run `git status`. It tells you what state you're in and
even suggests the right command. When unsure, prefer commands that **create** new
history over commands that **erase** it. We'll point out which is which.

## Discard Working Changes: `git restore <file>`

Say you edited a file, didn't like the result, and want it back the way it was at
the last commit. Use `git restore`:

```bash
echo "oops, bad edit" >> notes.txt   # a change you regret
git restore notes.txt                # throw it away, back to last commit
```

After this, `notes.txt` matches the last committed version. **Warning:** this
permanently discards the un-committed edit — there's no undo for the undo. Only
use it when you're sure you want those changes gone.

```bash
git status      # check what will be discarded BEFORE you restore
```

## Unstage a File: `git restore --staged <file>`

Imagine you staged a file with `git add` but realize you're not ready to commit it
yet. You want to take it **out of the box** without losing your edits. Add the
`--staged` flag:

```bash
git add report.txt              # staged by mistake (or too early)
git restore --staged report.txt # take it out of staging
```

Your edits to `report.txt` are safe — they just move from "staged" back to
"modified." This is the gentle opposite of `git add`.

```text
git add               → modified  ──▶ staged
git restore --staged  → staged    ──▶ modified  (edits kept)
git restore           → modified  ──▶ discarded (edits lost)
```

## Fix the Last Commit: `git commit --amend`

You committed, then immediately noticed a typo in the message — or you forgot to
include a file. `--amend` rewrites your **most recent** commit.

Fix just the message:

```bash
git commit --amend -m "Add login form validation"
```

Add a forgotten file to the last commit:

```bash
git add forgotten-file.txt
git commit --amend --no-edit    # keep the existing message
```

**Important:** amending replaces the old commit with a new one (a new hash). Only
amend commits you haven't shared with others yet. Amending a commit that teammates
already have causes confusing conflicts.

## Safely Undo an Old Commit: `git revert`

What if a commit from a while back was a mistake, but you've made other commits
since? You don't want to erase history — you want to add a new commit that
**cancels out** the bad one. That's `git revert`.

```bash
git revert a1b2c3d      # create a new commit that undoes a1b2c3d
```

Git makes a brand-new commit that reverses the changes from `a1b2c3d`, leaving the
original commit safely in history.

```text
Before:  A ── B(bad) ── C
After:   A ── B(bad) ── C ── D   (D undoes B's changes; nothing erased)
```

`git revert` is the **safest** way to undo because it never rewrites history. It's
the right choice for commits you've already shared with others.

## A Gentle Look at `git reset`

`git reset` moves the branch pointer to an earlier commit. It comes in three
modes, and the differences matter a lot. We're introducing it gently here — treat
it carefully.

### `--soft` — move the pointer, keep everything staged

```bash
git reset --soft HEAD~1     # undo the last commit, keep changes staged
```

`HEAD~1` means "one commit before the current one." With `--soft`, the commit is
undone but all its changes stay **staged**, ready to re-commit. Useful for redoing
a commit you split or messed up.

### `--mixed` — move the pointer, keep changes unstaged (the default)

```bash
git reset HEAD~1            # same as --mixed
```

The commit is undone and its changes drop back to **modified** (unstaged), but your
files are untouched. Nothing is lost — you just re-stage and re-commit.

### `--hard` — move the pointer AND throw away the changes

```bash
git reset --hard HEAD~1    # DANGER: undo the commit AND delete its changes
```

**`--hard` is destructive.** It discards the commit *and* wipes the matching
changes from your working directory. Work deleted this way can be very hard or
impossible to recover. A few rules to stay safe:

- Run `git status` and make sure you understand exactly what you'll lose.
- Never use `--hard` on commits you've shared with others.
- When in doubt, use `git revert` instead — it's reversible.

```text
--soft   → commit undone, changes STAGED      (safe)
--mixed  → commit undone, changes MODIFIED     (safe)
--hard   → commit undone, changes DELETED       (dangerous!)
```

## Decision Table: "I want to..."

```text
I want to...                                  Use this command
-------------------------------------------   --------------------------------
Throw away edits to a file (not committed)    git restore <file>
Take a file out of staging (keep edits)       git restore --staged <file>
Fix the message of my last commit             git commit --amend -m "..."
Add a forgotten file to my last commit        git add <file> && git commit --amend --no-edit
Undo an old commit that's already shared      git revert <hash>
Undo last commit but keep changes staged      git reset --soft HEAD~1
Undo last commit and unstage the changes      git reset HEAD~1   (--mixed)
Undo last commit AND delete the changes       git reset --hard HEAD~1  (careful!)
```

## Common Mistakes

- **Using `git restore <file>` without checking first.** It permanently discards
  un-committed edits. Run `git status` before you do.
- **Amending or resetting shared commits.** Rewriting history that others already
  have causes conflicts. Use `git revert` for anything you've pushed.
- **Reaching for `git reset --hard` to "clean up."** It can delete real work. Most
  of the time `git restore`, `git revert`, or a softer reset does the job safely.
- **Confusing `revert` with `reset`.** `revert` adds a new commit that undoes; it
  never erases. `reset` moves the pointer and can erase.
- **Misreading `HEAD~1`.** It means *one commit back*, not the current one.

## Exercises

1. Edit a tracked file, then use `git restore <file>` to discard the change.
   Confirm with `git status` that the working tree is clean again.
2. Stage a file with `git add`, then unstage it with `git restore --staged`.
   Verify your edits are still there afterward.
3. Make a commit with an intentionally bad message, then fix it with
   `git commit --amend`. Confirm the message changed with `git log --oneline`.
4. Create a commit you don't want, then undo it with `git revert`. Look at
   `git log` and explain why two commits now appear instead of zero.
5. In your own words, write down the difference between `--soft`, `--mixed`, and
   `--hard` reset, and one sentence on why `--hard` is risky.

## Key Takeaways

- `git restore <file>` discards un-committed edits (permanent — check first).
- `git restore --staged <file>` unstages a file while keeping your edits.
- `git commit --amend` fixes the most recent commit (only if not yet shared).
- `git revert` safely undoes a commit by adding a new commit; it never erases.
- `git reset` has three modes: `--soft` (keep staged), `--mixed` (keep modified),
  `--hard` (delete — dangerous).
- When unsure, prefer commands that create history (`revert`) over ones that erase
  it, and always run `git status` first.

---

[← Previous: Viewing History](03-viewing-history.md) | [Back to Course Index](../README.md) | [Next: Ignoring Files with .gitignore →](05-gitignore.md)
