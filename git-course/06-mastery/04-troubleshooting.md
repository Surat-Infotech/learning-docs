# Troubleshooting Common Git Problems

Sooner or later, Git will show you a scary-looking message and you will think you
broke everything. Good news: you almost certainly did not. Git is built to *keep*
your work, not lose it — most "disasters" are a one-line fix away. Think of this
lesson as a first-aid kit: a set of common "I'm stuck" situations, each with a
calm, step-by-step cure. When something goes wrong, find the matching problem
below and follow the recipe.

## What You'll Learn

- How to fix a commit made on the wrong branch
- Several safe ways to undo your last commit
- What **detached HEAD** means and how to get out of it
- How to handle "your branch and origin have diverged"
- How to deal with an accidentally committed large file
- How to abort a merge that went wrong
- How to read Git error messages calmly instead of panicking

## First: Stay Calm and Read the Message

Git's error messages usually **tell you what to do next**. Before doing anything,
read the message slowly. It often contains the exact command you need.

```bash
# When unsure of the current situation, these two are always safe to run:
git status        # shows where you are and what is going on
git log --oneline # shows recent commits
```

Almost nothing in Git is truly lost. Commits stick around in the **reflog** (a
log of where `HEAD` has been) even after you think they are gone:

```bash
git reflog        # your safety net — every move HEAD made, recoverable
```

Now, the cookbook.

## Problem: I committed to the wrong branch

You meant to commit on `feature` but you were on `main`.

**Solution** — move the commit to the right branch:

```bash
# You are on main with a commit that belongs on feature.

git branch feature        # create feature here (it now points at your commit)
git reset --hard HEAD~1   # rewind main back one commit (removes it from main)
git checkout feature      # switch to feature, where the commit safely lives
```

If `feature` already exists, use `git cherry-pick` instead: switch to `feature`
and run `git cherry-pick <commit-hash>` to copy the commit over, then remove it
from `main` with `git reset --hard HEAD~1`.

## Problem: I need to undo my last commit

There are three flavors depending on what you want.

```bash
# Keep the changes STAGED (ready to recommit):
git reset --soft HEAD~1

# Keep the changes but UNSTAGE them (back in working directory):
git reset --mixed HEAD~1     # --mixed is the default

# Throw the changes away entirely (careful!):
git reset --hard HEAD~1
```

If the commit was already **pushed and shared**, do not rewrite it. Instead make a
new commit that reverses it:

```bash
# Safe for shared history: creates a NEW commit undoing the old one.
git revert HEAD
```

## Problem: I'm in "detached HEAD" state

You ran something like `git checkout a1b2c3d` and Git warns about a **detached
HEAD**. This simply means `HEAD` is pointing at a *specific commit* instead of a
*branch*. You are looking at history directly. New commits here are not attached
to any branch and can be lost.

```text
Normal:    HEAD → main → (commit)        you are "on a branch"
Detached:  HEAD → (commit)               you are floating, no branch
```

**Solution** — decide whether you made changes:

```bash
# If you made NO changes and just want to leave:
git switch -            # go back to your previous branch

# If you DID make commits here and want to keep them:
git switch -c rescue-branch    # create a branch right where you are
```

Creating a branch "anchors" your floating commits so they cannot be lost.

## Problem: "Your branch and origin/main have diverged"

This means your local branch and the remote both have new commits the other does
not. Picture two people editing from the same starting point.

```text
            ┌─ your local commits
   common ──┤
            └─ remote commits (someone else pushed)
```

**Solution** — bring them together. The cleanest option for a personal branch is
to rebase your work on top of theirs:

```bash
git pull --rebase origin main
```

If conflicts appear, fix the files, then:

```bash
git add <fixed-files>
git rebase --continue
```

If you would rather merge than rebase, just run `git pull` (no `--rebase`), which
creates a merge commit joining both histories. Either way, then `git push`.

## Problem: I accidentally committed a large file

You committed a huge file (a video, a database dump) and now `git push` is slow or
rejected.

**If you have not pushed yet** and it was the last commit:

```bash
git rm --cached huge-file.zip      # untrack the file, keep it on disk
echo "huge-file.zip" >> .gitignore # stop it coming back
git commit --amend --no-edit       # rewrite the last commit without it
```

**If it is buried deeper in history**, you must scrub it from every commit using
`git filter-repo` or BFG (covered in the *Rewriting History Safely* lesson). That
rewrites history, so coordinate with your team first.

## Problem: A merge went wrong

You ran `git merge` (or `git pull`) and got a wall of conflicts you do not want to
deal with right now.

**Solution** — abort and go back to before the merge started:

```bash
git merge --abort      # cancel the merge, restore your branch to its prior state
```

Same idea works for a rebase that went sideways:

```bash
git rebase --abort     # cancel the rebase entirely
```

These are "undo" buttons that return you to safety. Take a breath, then try again
when you are ready to resolve the conflicts properly.

## Problem: I think I lost a commit

You reset too far, or deleted a branch, and a commit seems gone.

**Solution** — find it in the reflog and recover it:

```bash
git reflog                       # find the hash of the lost commit
git checkout -b recovered <hash> # bring it back on a new branch
```

The reflog remembers your moves for weeks, so "lost" commits are usually
recoverable.

## How to Read Error Messages Calmly

Most Git errors follow a friendly pattern. For example:

```text
! [rejected]        main -> main (fetch first)
error: failed to push some refs to 'origin'
hint: Updates were rejected because the remote contains work that you do
hint: not have locally. ... integrate the remote changes (e.g. 'git pull')
```

Read the `hint:` lines — Git is literally telling you to run `git pull` first.
The pattern is almost always: a short error, then hints suggesting the fix. When
you see a Git error:

1. Read it slowly; look for `hint:` lines.
2. Run `git status` to see the current state.
3. Match it to a recipe above.
4. Remember the reflog has your back.

## Common Mistakes

- **Reaching for `--hard` first.** `git reset --hard` deletes uncommitted work.
  Prefer `--soft` or `--mixed` unless you truly want changes gone.
- **Force-pushing to fix a divergence on a shared branch.** Use `git pull
  --rebase` to integrate instead of overwriting teammates' work.
- **Panicking about detached HEAD.** It is not an error — just create a branch if
  you want to keep your work.
- **Rewriting pushed commits to undo them.** Use `git revert` on shared history,
  not `reset`.
- **Forgetting the reflog exists.** Many "I lost everything" moments are a
  `git reflog` away from recovery.

## Exercises

1. In a practice repo, make a commit on `main`, then move it to a new branch
   `feature` using the wrong-branch recipe. Verify with `git log` on each branch.
2. Make a commit, then undo it three different ways (`--soft`, `--mixed`,
   `--hard`) in separate attempts, observing where your changes end up each time.
3. Check out an old commit by its hash to enter detached HEAD, confirm the warning,
   then escape safely with `git switch -`.
4. Start a merge that you know will conflict, then cancel it with `git merge
   --abort` and confirm your branch is back to normal with `git status`.
5. Make and then "lose" a commit with `git reset --hard`, then recover it using
   `git reflog`. Write down the steps you used.

## Key Takeaways

- Stay calm: read the message (especially `hint:` lines), run `git status`, and
  remember the **reflog** can recover almost anything.
- Undo the last commit with `git reset` (`--soft`/`--mixed`/`--hard`), or
  `git revert` if it was already shared.
- **Detached HEAD** just means you are on a commit, not a branch — create a branch
  to keep any work.
- "Diverged" branches are fixed by integrating, usually `git pull --rebase`.
- `git merge --abort` and `git rebase --abort` are safe "undo" buttons when things
  go sideways.

---

[← Previous: Aliases and Configuration Tips](03-aliases-and-config.md) | [Back to Course Index](../README.md) | [Next: Best Practices →](05-best-practices.md)
