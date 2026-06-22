# Keeping Your Branch Up to Date

Imagine you start drawing a map of a town, then take a week off. When you come back, the town has built three new roads and torn down a bridge. Your map is now out of date — and the longer you wait, the more it diverges from reality. A Git **feature branch** is the same: while you work on it, the `main` branch keeps moving as teammates merge their changes. If you don't refresh your branch, the two drift apart and eventually clash.

In this lesson we'll learn how to pull in everyone else's recent changes so your branch stays in sync with `main`, and the safe way to share the result.

## What You'll Learn

- Why long-lived branches **drift** away from `main` over time
- What `git pull` does, and the `--rebase` option
- Updating your feature branch from `main` with **merge** vs **rebase**
- The trade-offs between merge and rebase (and when to pick each)
- Using `git fetch` then rebase for more control
- Force-pushing safely after a rebase with `--force-with-lease`
- Why `--force-with-lease` is safer than plain `--force`

## Why Branches Drift

When you create a feature branch, it starts as an exact copy of `main` at that moment. But `main` doesn't stand still — teammates keep merging their PRs into it.

```text
At branch creation:        A week later:
main:    A───B             main:    A───B───C───D───E   ← teammates added C,D,E
              \                           \
feature:       X───Y       feature:        X───Y         ← yours is now behind
```

Your branch knows nothing about `C`, `D`, and `E`. The further `main` moves, the higher the chance your changes overlap with theirs, causing **merge conflicts** (when two branches change the same lines and Git can't decide which to keep). The fix is simple: update your branch regularly so the gap stays small.

## `git pull` and `--rebase`

`git pull` updates your *current* branch with new commits from its remote counterpart. It's actually two steps in one:

```bash
git pull
# = git fetch  (download new commits)
# + git merge  (combine them into your branch)
```

By default, `pull` **merges**, which can create an extra "merge commit." Many teams prefer `pull --rebase`, which instead replays your commits on top of the new ones for a cleaner, straight-line history:

```bash
git pull --rebase   # download new commits, then replay yours on top
```

You can make rebase the default for pulls:

```bash
git config --global pull.rebase true   # always rebase when pulling
```

## Updating a Feature Branch from `main`

Pulling keeps a branch in sync with *its own* remote copy. But you also need your feature branch to catch up with the latest **`main`**. There are two ways to do this: **merge** and **rebase**.

### Option A: Merge `main` Into Your Branch

```bash
git switch feature/login
git fetch origin            # get the latest commits from the server
git merge origin/main       # bring main's new commits into your branch
```

This creates a **merge commit** that ties the two histories together:

```text
main:    A───B───C───D───E
              \           \
feature:       X───Y───────M   (M = merge commit)
```

- **Pro:** Simple, safe, never rewrites history. No force-push needed.
- **Con:** History gets a bit tangled with extra merge commits.

### Option B: Rebase Your Branch Onto `main`

**Rebase** lifts your commits off and replays them on top of the latest `main`, as if you'd just started your work today.

```bash
git switch feature/login
git fetch origin
git rebase origin/main      # replay X, Y on top of main's latest commit
```

```text
Before:                          After rebase:
main:    A───B───C───D───E       main:    A───B───C───D───E
              \                                           \
feature:       X───Y             feature:                  X'──Y'  (replayed)
```

Notice `X` and `Y` became `X'` and `Y'` — they are **new commits** with new IDs. The history is now a clean straight line.

- **Pro:** Tidy, linear history that's easy to read.
- **Con:** Rewrites your branch's commits, so you must force-push (covered below). Never rebase commits other people have already built on.

### Which Should You Choose?

```text
Merge:   safe, beginner-friendly, slightly messy history
Rebase:  clean linear history, but rewrites commits → needs force-push
```

A common team rule: **rebase your private feature branch** to keep it tidy, but **merge** (don't rebase) anything shared by multiple people. When unsure, merging is the safe choice.

## `git fetch` Then Rebase

`git fetch` downloads the latest commits from the server **without** changing your branch. It's the cautious first step that lets you look before you leap.

```bash
git fetch origin            # download, but don't touch my branch yet
git log origin/main         # peek at what's new (optional)
git rebase origin/main      # now replay my work on top
```

Splitting fetch from rebase gives you a chance to inspect what changed before integrating it. If a rebase hits conflicts, Git pauses so you can fix them:

```bash
# Fix the conflicting files in your editor, then:
git add <fixed-files>
git rebase --continue       # resume replaying
# Or, to bail out entirely:
git rebase --abort          # put everything back as it was
```

## Force-Pushing Safely After a Rebase

After a rebase, your local branch and its remote copy have *different* commit histories (remember `X` became `X'`). A normal `git push` will be rejected because the histories don't match. You need to overwrite the remote branch — but how you do it matters.

### Why Not Plain `--force`

```bash
git push --force            # DANGEROUS: overwrites the remote no matter what
```

Plain `--force` blindly replaces the remote branch with yours. If a teammate pushed a commit to that branch while you were rebasing, `--force` **erases their work** without warning.

### Use `--force-with-lease`

```bash
git push --force-with-lease  # SAFE: only overwrite if no one else pushed
```

`--force-with-lease` first checks: "Is the remote branch still exactly where I last saw it?" If yes, it pushes. If someone else pushed in the meantime, it **refuses** and warns you — so you never silently destroy a teammate's commits.

```text
--force            = "Overwrite, no questions asked."   (risky)
--force-with-lease = "Overwrite only if nothing changed." (safe)
```

Make `--force-with-lease` your default habit. It's the difference between a careful overwrite and a reckless one.

## Common Mistakes

- **Letting a branch sit for weeks without updating.** Update often so conflicts stay small and easy.
- **Rebasing a shared branch.** Rewriting commits others have pulled creates a mess for everyone. Rebase only your own private branches.
- **Using plain `--force`.** It can wipe out a teammate's pushed work. Use `--force-with-lease` instead.
- **Panicking during a rebase conflict.** It's normal. Fix the files, `git add`, then `git rebase --continue` — or `--abort` to start over.
- **Confusing `fetch` with `pull`.** `fetch` only downloads; `pull` downloads *and* integrates. `fetch` first when you want to look before merging.

## Exercises

1. In your own words, explain why a feature branch "drifts" from `main` and why that causes conflicts.
2. Describe the difference between merging `main` into your branch and rebasing your branch onto `main`. Draw a small diagram of each.
3. A teammate has already pulled your branch and built on top of it. Should you rebase it? Explain your reasoning.
4. Explain why a force-push is needed after a rebase, and why `--force-with-lease` is safer than `--force`.
5. Write the full sequence of commands to fetch the latest `main`, rebase your `feature/login` branch onto it, and safely push the result.

## Key Takeaways

- Feature branches **drift** from `main` as teammates merge changes; update yours regularly to keep conflicts small.
- `git pull` = `fetch` + `merge`; `git pull --rebase` replays your commits for a cleaner history.
- Update a branch from `main` by **merging** (safe, no force-push) or **rebasing** (clean, linear, but rewrites commits).
- A good rule: rebase your private branch, merge anything shared by others.
- `git fetch` downloads without changing your branch — a safe way to look before integrating.
- After a rebase you must force-push; always use `--force-with-lease`, which refuses to overwrite a teammate's new work.

---

[← Previous: Code Review and Pull Request Etiquette](02-code-review-and-prs.md) | [Back to Course Index](../README.md) | [Next: Resolving Conflicts in a Team →](04-team-conflicts.md)
