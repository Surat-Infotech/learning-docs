# Rebasing: A Cleaner History

You have learned that merging ties two branches together, sometimes leaving a little knot in your history (the merge commit). **Rebasing** is a different way to combine work that keeps your history looking like one clean, straight line.

Here is the analogy. Imagine you started writing notes on a photocopy of a document, but meanwhile the original got a few new paragraphs added. Instead of stapling your notes onto the old copy, you take a *fresh* copy with the new paragraphs and re-write your notes on top of it. Your notes are now based on the latest version. That "re-base" — changing the base your work sits on — is exactly what `git rebase` does.

This lesson keeps things gentle. The deeper, more powerful "interactive" rebase comes in a later module. For now, focus on the core idea and one golden safety rule.

## What You'll Learn

- What rebasing actually does (replays your commits onto a new base)
- A before/after diagram of a rebase
- How to run `git rebase main`
- The difference between merge and rebase, and when to use each
- The GOLDEN RULE: never rebase commits you have already shared
- How to handle conflicts during a rebase with `--continue` and `--abort`

## What Rebase Does

Picture this familiar situation: you branched off `main`, did some work, and meanwhile `main` moved forward too.

Before (the branches have diverged):

```text
A ──> B ──> C ──> F   <── main
              \
               D ──> E   <── feature   <── HEAD
```

Your `feature` commits (`D` and `E`) were built on top of `C`. But `main` has since gained commit `F`. Your work is now based on an older point.

A **rebase** lifts your `feature` commits off and replays them, one by one, on top of the *current* tip of `main` (`F`). It is as if you had started your feature from `F` all along.

After `git rebase main` (while on `feature`):

```text
A ──> B ──> C ──> F ──> D' ──> E'   <── feature   <── HEAD
                  ^
                  main
```

Notice three things:

1. The history is now a **straight line** — no fork, no merge commit.
2. Your commits became `D'` and `E'` (with little marks). They are **new commits** with the same changes but new IDs, because their starting point changed.
3. `feature` now contains all of `main`'s latest work plus your own, neatly stacked on top.

## Running a Rebase

The everyday command is simple. Stand on your feature branch and rebase onto `main`:

```bash
# Make sure you are on the branch you want to move
git switch feature

# Replay feature's commits on top of the latest main
git rebase main
# Output:
# Successfully rebased and updated refs/heads/feature.
```

After this, your `feature` branch is up to date with `main` and your commits sit cleanly on top. Often the next step is a fast-forward merge into `main`, which now leaves no merge commit at all:

```bash
git switch main
git merge feature
# Output: Fast-forward
```

Because `feature` is now directly ahead of `main`, the merge is a simple fast-forward — a perfectly straight, clean history.

## Merge vs Rebase: When to Use Each

Both combine work; they just shape history differently.

| | Merge | Rebase |
|---|-------|--------|
| History shape | Preserves the fork; may add a merge commit | Straight line, no merge commit |
| Commits | Keeps original commits unchanged | Creates new commits (new IDs) |
| Records "these branches met here" | Yes (the merge commit) | No |
| Best for | Shared/public branches, honest history | Tidying up your own local work before sharing |

A simple way to choose:

- **Use merge** when you are combining branches that other people also use, or when you *want* a record that two lines of work joined. Merging never rewrites existing commits, so it is always safe.
- **Use rebase** to keep your *own* feature branch up to date with `main` and to keep the project history clean and linear — but only while that work is still private to you.

Neither is "better." They are tools for different goals.

## The GOLDEN RULE of Rebasing

This is the single most important thing in this lesson, so it gets its own heading and a box:

> **Never rebase commits you have already pushed or shared with others.**

Why? Because rebasing **rewrites history** — it replaces your commits (`D`, `E`) with brand-new ones (`D'`, `E'`) that have different IDs. If those original commits already exist on a shared server or in a teammate's copy, you have now created a confusing fork: their history still has the old commits, yours has the new ones. Untangling that is painful for everyone.

So follow this simple boundary:

- **Local, unshared work** → rebasing is fine and useful.
- **Already pushed/shared work** → do **not** rebase it; use merge instead.

If in doubt, merge. Merging never rewrites history, so it can never cause this problem.

## Handling Conflicts During a Rebase

Because a rebase replays your commits onto a new base, it can hit the same kind of conflicts a merge does — two changes touching the same lines. The difference is that a rebase replays commits **one at a time**, so it may pause on each one.

When a conflict appears, Git stops and tells you:

```bash
git rebase main
# Output:
# CONFLICT (content): Merge conflict in home.txt
# error: could not apply d4e5f6a... Make welcome exciting
# Resolve all conflicts manually, then run "git rebase --continue".
```

The resolution steps feel just like a merge conflict, with one new command at the end:

```bash
# 1. Open the conflicted file and fix it (remove the <<<<<<< ======= >>>>>>> markers)

# 2. Stage the fixed file to mark it resolved
git add home.txt

# 3. Continue the rebase (NOT git commit!)
git rebase --continue
```

The key difference: after staging, you run **`git rebase --continue`** instead of `git commit`. This tells Git to finish applying the current commit and move on to the next one. If more commits conflict, repeat the same three steps until Git announces success.

### Bailing Out

Just like a merge, you can cancel a rebase at any time and return to exactly where you started:

```bash
git rebase --abort
```

This undoes the entire in-progress rebase, putting your branch back to how it was before you ran `git rebase`. It is your safe escape hatch — nothing is lost.

## A Quick Mental Model

- **Merge** says: "Keep both timelines, and record where they joined."
- **Rebase** says: "Pretend I started my work from the latest point, so history stays a straight line."

Both get your work and `main`'s work together. You are just choosing what the history book should look like afterward.

## Common Mistakes

- **Rebasing shared commits.** This is the big one. Never rebase work you have already pushed or that others use. When unsure, merge.
- **Running `git commit` during a rebase.** After fixing a conflict, use `git rebase --continue`, not `git commit`.
- **Forgetting to `git add` before continuing.** Staging the fixed file is how you tell Git the conflict is resolved.
- **Expecting your commit IDs to stay the same.** Rebasing creates new commits with new IDs by design; that is normal.
- **Feeling trapped mid-rebase.** `git rebase --abort` always returns you safely to the starting point.

## Exercises

1. In your own words, explain what `git rebase main` does to your feature branch's commits, and why their IDs change.
2. Draw a before/after diagram for rebasing a `feature` branch (with commits `D ──> E`) onto a `main` that has advanced to commit `F`.
3. State the GOLDEN RULE of rebasing and explain, in one or two sentences, why breaking it causes problems for a team.
4. Create a scenario where rebasing causes a conflict, resolve it, and complete the rebase using `git add` followed by `git rebase --continue`.
5. Make a small table of two situations — one where you would choose merge and one where you would choose rebase — and justify each choice.

## Key Takeaways

- **Rebasing** replays your commits on top of another branch, producing a clean, straight-line history.
- It creates **new commits** (new IDs) with the same changes, because their base changed.
- `git rebase main` (run from your feature branch) moves your work on top of the latest `main`.
- **Merge** preserves the fork and never rewrites history; **rebase** rewrites history for a tidy line — use it only on private work.
- **GOLDEN RULE:** never rebase commits you have already pushed or shared. When in doubt, merge.
- During a rebase conflict, fix the file, `git add` it, then `git rebase --continue` (not `git commit`).
- `git rebase --abort` safely cancels an in-progress rebase.

---

[← Previous: Merge Conflicts](04-merge-conflicts.md) | [Back to Course Index](../README.md) | [Next: Remotes and GitHub →](../03-remotes-and-github/01-remotes-and-github.md)
