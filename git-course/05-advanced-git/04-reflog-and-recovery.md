# Reflog and Recovering Lost Work

At some point everyone has that moment of panic: "I ran a bad reset and my
commits are gone!" or "I deleted the wrong branch!" Your heart sinks. But here
is the good news that almost no beginner knows: **Git almost never truly loses
committed work.**

The hero of this story is the **reflog**. Think of it as a security camera that
records every place `HEAD` has been — every commit, switch, reset, and rebase.
Even when commits seem to vanish from your branches, the reflog still remembers
where they were, and you can walk right back to them.

Once you know the reflog exists, a huge amount of Git anxiety melts away. Let's
learn to use your safety net.

## What You'll Learn

- What `git reflog` is and why it is your safety net
- How to read reflog entries
- How to recover a commit "lost" after a bad reset
- How to recover work after a bad rebase
- How to bring back a deleted branch
- Why committed work is rarely ever truly gone

## What Is the Reflog?

`HEAD` is Git's pointer to "where you are right now". The **reflog** (reference
log) is a private, local history of everywhere `HEAD` has pointed over time.

Every time you commit, switch branches, reset, merge, or rebase, Git writes a
line into the reflog. It is like a breadcrumb trail of your recent moves.

The crucial part: even if a commit is no longer reachable from any branch (so it
does not show up in `git log`), the reflog still has a reference to it. As long
as a reference exists, Git keeps the commit around.

```bash
git reflog            # show the log of where HEAD has been
```

A typical reflog looks like this:

```text
a1b2c3d HEAD@{0}: reset: moving to HEAD~2
e4f5g6h HEAD@{1}: commit: Add payment validation
i7j8k9l HEAD@{2}: commit: Add cart total
m0n1o2p HEAD@{3}: checkout: moving from main to feature
```

Each line shows a commit hash, a reference like `HEAD@{1}`, and a description of
what action moved HEAD there. `HEAD@{0}` is the most recent position;
`HEAD@{1}` is one step back, and so on.

## Recovering After a Bad Reset

Here is the classic disaster. You meant to undo one commit but accidentally
reset too far back:

```bash
git reset --hard HEAD~2     # OOPS — this threw away two commits
```

Your two commits seem to be gone. `git log` does not show them anymore. Do not
panic. Look at the reflog:

```bash
git reflog
```

```text
a1b2c3d HEAD@{0}: reset: moving to HEAD~2
e4f5g6h HEAD@{1}: commit: Add payment validation   <- your lost work!
i7j8k9l HEAD@{2}: commit: Add cart total
```

`HEAD@{1}` (hash `e4f5g6h`) is right before the bad reset — the state you want
back. Restore it:

```bash
git reset --hard e4f5g6h     # go back to that commit by hash
# or, equivalently:
git reset --hard HEAD@{1}     # go back to that reflog position
```

Your commits are back. The reflog let you rewind your rewind.

## Recovering After a Bad Rebase

Interactive rebase rewrites commits, and sometimes you realize afterward that
you squashed or dropped something you needed. The fix is the same idea.

Before the rebase finishes, `git rebase --abort` is the easiest escape. But if
the rebase already completed and you only later noticed the problem, the reflog
still has the pre-rebase state:

```bash
git reflog
```

```text
9z8y7x6 HEAD@{0}: rebase (finish): returning to refs/heads/feature
... (rebase steps) ...
e4f5g6h HEAD@{5}: rebase (start): checkout main   <- just before rebase
```

Find the entry from just before the rebase started and reset to it:

```bash
git reset --hard HEAD@{5}     # restore the branch to its pre-rebase state
```

Your branch is exactly as it was before you ran the rebase.

## Recovering a Deleted Branch

Deleting a branch only removes the *label* (the branch name pointer). The
commits it pointed to are still there — they just need a reference again.

Suppose you delete a branch by mistake:

```bash
git branch -D feature        # deleted the branch... oops
```

Find the commit the branch was pointing at. The reflog records it (look for the
last commit you made on `feature`):

```bash
git reflog
```

```text
e4f5g6h HEAD@{3}: commit: Finish feature work    <- tip of deleted branch
```

Now recreate the branch at that exact commit:

```bash
git branch feature e4f5g6h   # recreate the branch at that commit
```

The branch is back, pointing at all the same commits. There is also a dedicated
command that searches for dangling commits if the reflog is not enough:

```bash
git fsck --lost-found        # find unreachable commits as a last resort
```

## Why Committed Work Is Rarely Lost

Git is built to protect your work. When you **commit**, that snapshot is stored
in Git's database and stays there even if no branch points to it. Git only
cleans up such "orphaned" commits during garbage collection, which by default
does not touch anything younger than about two weeks.

So the rule to remember:

> If you committed it, you can almost certainly get it back.

The big exception is changes you **never committed** — uncommitted edits wiped
by `git reset --hard` or `git checkout` are genuinely hard to recover, because
Git never recorded them. This is exactly why committing often (or stashing) is
such a good habit.

## Common Mistakes

- **Forgetting the reflog exists** and assuming a reset destroyed your work. Run
  `git reflog` first — your commits are likely still listed.
- **Confusing committed and uncommitted work.** The reflog recovers commits, not
  uncommitted edits you never saved.
- **Expecting the reflog to be shared.** The reflog is local to your machine
  only; it does not travel with `push` or `clone`.
- **Waiting too long.** Reflog entries expire eventually (commits unreachable
  for ~90 days, or ~30 for some). Recover sooner rather than later.
- **Using `git reset --hard` without thinking.** It is the command most likely
  to scare you. Knowing the reflog makes it survivable, but use it carefully.

## Exercises

1. Make two commits, then run `git reset --hard HEAD~2`. Use `git reflog` to
   find your lost commits and restore them with `git reset --hard`.
2. Run `git reflog` in any active repo and read through the entries. Identify
   which actions (commit, checkout, reset) created each line.
3. Create a branch with a commit, delete it with `git branch -D`, then recover
   it using the reflog and `git branch <name> <hash>`.
4. Do a small interactive rebase that squashes commits, then use the reflog to
   restore the branch to its pre-rebase state.
5. Explain in your own words why committed work survives a bad reset but
   uncommitted changes may not.

## Key Takeaways

- `git reflog` is a local log of everywhere `HEAD` has been — your safety net.
- Recover a bad reset or rebase by resetting to the right `HEAD@{n}` or hash.
- Recover a deleted branch by recreating it at its last commit: `git branch
  <name> <hash>`.
- Committed work is almost never truly lost; uncommitted work can be, so commit
  or stash often.
- The reflog is private to your machine and entries do eventually expire, so
  recover promptly.

---

[← Previous: Stashing Changes](03-stashing.md) | [Back to Course Index](../README.md) | [Next: Tags and Releases →](05-tags-and-releases.md)
