# Cherry-Picking Commits

Imagine your teammate fixed an important bug on their branch, but you need that
exact fix on your branch *right now* — without merging their entire branch and
all its other unfinished changes. You just want that one commit.

That is exactly what cherry-picking does. Think of a bowl of fruit: instead of
taking the whole bowl, you reach in and pick out the one cherry you want. Git's
`cherry-pick` lets you reach into any branch's history, grab a single commit,
and copy it onto your current branch.

It is a precise, surgical tool. Used carefully, it is completely safe, and we
will walk through exactly when and how to use it.

## What You'll Learn

- What `git cherry-pick` actually does
- When cherry-picking is the right tool
- How to cherry-pick a single commit by its hash
- How to cherry-pick a range of commits
- How to handle conflicts during a cherry-pick
- Using `--continue` and `--abort`
- The danger of duplicate commits and how to avoid it

## What Cherry-Pick Does

`git cherry-pick <hash>` takes the **changes** introduced by one commit and
applies them as a **new commit** on your current branch.

The key word is *new*. Cherry-pick does not move the original commit. It copies
the change and creates a fresh commit with a brand-new hash on your branch. The
original stays exactly where it was.

```text
Before:
main:    A---B---C
                  \
feature:           D---E   (E is a great bug fix)

After cherry-picking E onto main:
main:    A---B---C---E'    (E' is a copy of E)
                  \
feature:           D---E
```

Notice `E'` (E-prime) is a copy. It has the same content as `E`, but a
different hash.

## When to Use It

Cherry-picking shines in a few common situations:

- **Grab a bug fix from another branch** without merging everything else.
- **Apply a hotfix to multiple branches** — for example, a fix that needs to go
  on both `main` and a release branch.
- **Rescue a single commit** you made on the wrong branch.

If you want *all* of another branch's commits, use `merge` or `rebase` instead.
Cherry-pick is for when you want **just one** (or a few) specific commits.

## Cherry-Picking a Single Commit

First, find the hash of the commit you want. You can see hashes with
`git log --oneline`:

```bash
git log --oneline feature
# h7i8j9k Fix crash on empty input   <- we want this one
# d4e5f6g Add experimental setting
# a1b2c3d Start feature
```

Now switch to the branch where you want the fix and cherry-pick it:

```bash
git switch main             # go to the branch that needs the fix
git cherry-pick h7i8j9k     # copy that commit onto main
```

Git creates a new commit on `main` containing the same change. Check it with
`git log --oneline`.

## Cherry-Picking Several Commits

You can pass more than one hash to copy several commits at once:

```bash
git cherry-pick a1b2c3d h7i8j9k     # pick two specific commits
```

You can also pick a **range** of commits using `..` (two dots):

```bash
# Pick every commit after START up to and including END
git cherry-pick START..END
```

Be careful: `START..END` does **not** include the `START` commit itself — it
means "everything after START, up to END". To include the starting commit too,
use `START^..END` (the `^` means "the commit before START"):

```bash
git cherry-pick START^..END         # include START as well
```

## Handling Conflicts

Just like merging, a cherry-pick can hit a **conflict** if the commit's changes
clash with what is already on your branch. Git pauses and tells you which files
conflict.

Open the files, fix the conflict markers, then:

```bash
git add <fixed-file>          # mark the conflict resolved
git cherry-pick --continue    # finish creating the new commit
```

If you change your mind and want to cancel the whole thing, restoring your
branch to before the cherry-pick:

```bash
git cherry-pick --abort       # undo and go back to where you started
```

There is also `--skip`, which skips the current commit if it no longer applies
(for example, the change is already present). Use it sparingly.

## The Duplicate Commit Caution

Because cherry-pick **copies** a commit, the same change can end up existing in
two places with two different hashes. This is usually fine, but it can cause
confusion later.

The classic trap: you cherry-pick commit `E` from `feature` onto `main`, and
*then* you also merge `feature` into `main`. Now the change from `E` appears
twice in your history — once as the cherry-picked copy, once from the merge.
Git is usually smart enough to avoid a true duplicate, but your log can look
cluttered, and in some cases you get conflicts.

A good rule of thumb:

> Cherry-pick when you want a commit *instead of* merging the whole branch. If
> you plan to merge the branch later anyway, you probably do not need to
> cherry-pick from it.

## Common Mistakes

- **Cherry-picking a commit you will also merge later.** This leads to
  duplicate-looking history. Pick *or* merge, not usually both.
- **Forgetting which branch you are on.** Cherry-pick adds the commit to your
  *current* branch. Run `git status` first to confirm where you are.
- **Misusing ranges.** Remember `A..B` excludes `A`. Use `A^..B` to include it.
- **Treating a conflict as a failure.** A conflict just needs fixing: edit,
  `git add`, then `git cherry-pick --continue`.
- **Cherry-picking huge ranges** when a merge would be simpler. For many
  commits, merge or rebase is usually cleaner.

## Exercises

1. Create two branches. Make a commit on one branch, then cherry-pick it onto
   the other. Confirm with `git log --oneline` that the change appears on both,
   with different hashes.
2. Cherry-pick a commit that conflicts with your current branch. Resolve the
   conflict and finish with `git cherry-pick --continue`.
3. Start a cherry-pick, then run `git cherry-pick --abort`. Confirm your branch
   is unchanged.
4. Cherry-pick a range of two or three commits using the `START^..END` form.
5. In your own words, explain the difference between cherry-picking a commit and
   merging the branch it came from.

## Key Takeaways

- `git cherry-pick <hash>` copies the change from one commit onto your current
  branch as a brand-new commit.
- It is perfect for grabbing a single fix from another branch.
- You can pick multiple commits, or a range with `START^..END`.
- Conflicts are handled with `git add` then `git cherry-pick --continue`, or
  cancel with `git cherry-pick --abort`.
- Beware duplicate commits — usually pick *or* merge, not both.

---

[← Previous: Interactive Rebase](01-interactive-rebase.md) | [Back to Course Index](../README.md) | [Next: Stashing Changes →](03-stashing.md)
