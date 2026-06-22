# Creating and Switching Branches

Now that you know a branch is just a movable pointer to a commit, it is time to make some. Think of branches like tabs in a notebook: you can open a new tab to work on something, flip between tabs freely, rename a tab, or tear one out when you are done with it.

In this lesson you will learn every everyday command for working with branches — creating, listing, switching, renaming, and deleting them — and you will build a small feature branch from start to finish.

## What You'll Learn

- How to list your branches with `git branch`
- How to create a branch with `git branch <name>`
- How to switch branches with the modern `git switch`
- The shortcut `git switch -c` to create and switch in one step
- The older `git checkout -b` command (you will see it everywhere)
- How to rename a branch with `git branch -m`
- How to delete a branch with `git branch -d` versus `-D`
- How to tell which branch you are currently on

## Listing Branches

`git branch` with no arguments lists all the branches in your repository. The current branch is marked with an asterisk (`*`).

```bash
git branch
# Output:
# * main
#   feature-login
#   bugfix-typo
```

The `*` next to `main` means `main` is the branch you are on right now.

To also see the last commit on each branch:

```bash
git branch -v
# Output:
# * main          a1b2c3d  Add homepage
#   feature-login f4e5d6c  Start login form
```

## Creating a Branch

To create a new branch, give `git branch` a name:

```bash
# Create a branch called feature-login
git branch feature-login
```

Important: this **creates** the branch but does **not** switch to it. You are still on whatever branch you were on before. The new branch points at the same commit you are currently sitting on.

```text
A ──> B ──> C   <── main   <── HEAD
                  ^
                  feature-login
```

Both `main` and `feature-login` now point at commit `C`. They are identical until you commit on one of them.

## Switching Branches with git switch

To move onto a branch, use `git switch`:

```bash
# Move onto the feature-login branch
git switch feature-login
# Output: Switched to branch 'feature-login'
```

Now `HEAD` points at `feature-login`, and your files match that branch.

```text
A ──> B ──> C   <── main
                  ^
                  feature-login   <── HEAD
```

To switch back to `main`:

```bash
git switch main
# Output: Switched to branch 'main'
```

> `git switch` was added to make branch-switching clear and beginner-friendly. It only switches branches, so it is hard to misuse.

## Create and Switch in One Step

Most of the time you want to create a branch *and* immediately start working on it. The `-c` flag (for "create") does both:

```bash
# Create a new branch and switch to it at once
git switch -c feature-login
# Output: Switched to a new branch 'feature-login'
```

This is the command you will reach for most often.

## The Older Way: git checkout

Before `git switch` existed, branching was done with `git checkout`. You will see this in countless tutorials and on your team's keyboard, so it is worth knowing:

```bash
# Switch to an existing branch (old way)
git checkout feature-login

# Create and switch in one step (old way)
git checkout -b feature-login
```

So `git checkout -b <name>` is the old equivalent of `git switch -c <name>`.

> Why two ways? `git checkout` does many different jobs (switching branches, restoring files, and more), which confused beginners. Git split off the clearer `git switch` and `git restore` commands. Both still work — use `git switch` when you can.

## Renaming a Branch

Made a typo in a branch name, or want a clearer one? Use `-m` (for "move/rename"):

```bash
# Rename the branch you are currently on
git branch -m better-name
```

```bash
# Rename a specific branch by giving both names
git branch -m old-name new-name
```

## Deleting a Branch

When you are finished with a branch — for example, after its work has been merged — you can delete it. There are two flavors.

### Safe delete: -d

```bash
# Delete a branch (only if its work is already merged)
git branch -d feature-login
# Output: Deleted branch feature-login (was f4e5d6c).
```

The lowercase `-d` is the **safe** option. If the branch contains commits that have *not* been merged anywhere, Git refuses to delete it so you do not lose work:

```text
error: The branch 'feature-login' is not fully merged.
If you are sure you want to delete it, run 'git branch -D feature-login'.
```

### Force delete: -D

```bash
# Force-delete a branch even if its work is unmerged
git branch -D feature-login
```

The uppercase `-D` **forces** the delete, throwing away any unmerged commits. Use it only when you are certain you do not want that work. The capital letter is a small built-in warning: it is the more dangerous option.

> You cannot delete the branch you are currently standing on. Switch to another branch first (usually `git switch main`), then delete.

## A Worked Example

Let's create a feature branch, do some work on it, and confirm the branches diverged. Start on `main`:

```bash
# Confirm where we are
git branch --show-current
# Output: main

# Create and switch to a feature branch
git switch -c add-greeting
# Output: Switched to a new branch 'add-greeting'
```

Now make a change and commit it on this branch:

```bash
# Create a file (using your editor or a command)
echo "Hello, world!" > greeting.txt

# Stage and commit it
git add greeting.txt
git commit -m "Add a greeting file"
# Output: [add-greeting 9a8b7c6] Add a greeting file
```

The `add-greeting` branch has moved forward, but `main` has not:

```text
A ──> B ──> C   <── main
              \
               D   <── add-greeting   <── HEAD
```

Switch back to `main` and look at the folder:

```bash
git switch main
# Output: Switched to branch 'main'

ls
# greeting.txt is GONE here — it only exists on add-greeting
```

This proves the work is isolated. The `greeting.txt` file lives only on the `add-greeting` branch. Switch back and it reappears:

```bash
git switch add-greeting
ls
# greeting.txt is back
```

You have just experienced the whole point of branches: parallel, isolated work in one folder.

## Common Mistakes

- **Expecting `git branch <name>` to switch you.** It only creates the branch. Use `git switch -c <name>` to create and switch together.
- **Committing on the wrong branch.** Always check `git branch --show-current` (or `git status`) before committing.
- **Trying to delete the branch you are on.** Switch away first, then delete.
- **Using `-D` when you meant `-d`.** Uppercase force-deletes and can lose unmerged work. Prefer lowercase `-d` so Git can protect you.
- **Panicking when files "disappear" after switching.** They are safe in the other branch's commits — switch back and they return.

## Exercises

1. List all branches in a repository with `git branch`. Which one has the `*` next to it, and what does that mean?
2. Create a branch named `experiment` two different ways: once with `git branch` followed by `git switch`, and once with the single command `git switch -c`. Confirm with `git branch --show-current`.
3. On a feature branch, create a file and commit it. Switch to `main` and run `ls`. Is the file there? Explain why or why not.
4. Rename a branch from `experiment` to `tryout` using `git branch -m`. Verify with `git branch`.
5. Delete a fully-merged branch with `git branch -d`. Then explain in your own words when you would need `-D` instead, and why it is riskier.

## Key Takeaways

- `git branch` lists branches; `git branch <name>` creates one without switching.
- `git switch <name>` moves you onto an existing branch.
- `git switch -c <name>` creates a branch and switches to it in one step — your everyday command.
- `git checkout <name>` and `git checkout -b <name>` are the older equivalents; you will still see them.
- `git branch -m <new-name>` renames a branch.
- `git branch -d` safely deletes a merged branch; `git branch -D` force-deletes and can lose work.
- You cannot delete the branch you are currently on — switch away first.

---

[← Previous: Understanding Branches](01-understanding-branches.md) | [Back to Course Index](../README.md) | [Next: Merging →](03-merging.md)
