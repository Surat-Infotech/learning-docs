# Stashing Changes

You are halfway through writing some code when a teammate messages: "Can you
quickly fix this bug on the other branch?" But you are not ready to commit your
half-finished work, and Git will not let you switch branches with messy changes
hanging around.

This is what stashing is for. Think of `git stash` as a **drawer** where you
tuck away your work-in-progress to deal with later. You sweep your current
changes into the drawer, your working area becomes clean, and you can switch
branches freely. When you come back, you open the drawer and your changes are
right there waiting.

It is one of the most convenient everyday tools in Git, and it is completely
safe — nothing is deleted, just shelved.

## What You'll Learn

- What `git stash` does and why it is handy
- How to save, list, and inspect stashes
- The difference between `apply`, `pop`, `drop`, and `clear`
- How to stash untracked files with `-u`
- How to give a stash a useful name with `git stash push -m`
- How to apply a specific stash from the list
- When stashing is the right choice

## What Is a Stash?

A **stash** is a saved snapshot of your uncommitted changes — both the staged
and unstaged work in your working directory. Saving a stash rolls your working
area back to a clean state (matching the last commit) and stores your changes
safely on a stack you can return to.

You can have many stashes at once, kept in a list like a stack of notes.

## Saving Your Work

The simplest command shelves everything you have changed:

```bash
git stash          # save all tracked changes and clean the working area
```

After this, `git status` shows a clean working tree, as if you had not touched
anything. Your changes are not gone — they are in the drawer.

A clearer, modern way to write the same thing is `git stash push`:

```bash
git stash push     # same as "git stash"
```

## Naming a Stash

By default a stash gets a generic auto-generated label, which is confusing when
you have several. Give it a description with `-m` (for "message"):

```bash
git stash push -m "half-done login form"
```

Now your stash has a name you will actually recognize later.

## Listing and Inspecting Stashes

To see everything in the drawer:

```bash
git stash list
```

You will see something like:

```text
stash@{0}: On main: half-done login form
stash@{1}: WIP on feature: 9a8b7c6 Add settings page
```

Each stash has a reference like `stash@{0}`. The most recent stash is always
`stash@{0}`. The number is its position on the stack, not a permanent ID — it
shifts as you add or remove stashes.

To peek at what a stash contains without applying it:

```bash
git stash show stash@{0}        # summary of changed files
git stash show -p stash@{0}     # full diff (the -p means "patch")
```

## Getting Your Work Back: apply vs pop

There are two ways to restore a stash, and the difference matters.

**`git stash apply`** restores the changes but **leaves the stash in the list**:

```bash
git stash apply        # restore the most recent stash, keep it saved
```

**`git stash pop`** restores the changes and **removes the stash** from the
list:

```bash
git stash pop          # restore the most recent stash, then delete it
```

Use `pop` for the common case ("I'm done with this stash"). Use `apply` when
you might want to apply the same stash to more than one branch, or you want to
keep it as a backup until you are sure it worked.

## Applying a Specific Stash

If you have several stashes, name the one you want by its reference:

```bash
git stash apply stash@{2}      # apply the third stash, keep it
git stash pop stash@{1}        # apply the second stash, then remove it
```

## Removing Stashes

To delete a single stash you no longer need:

```bash
git stash drop stash@{0}       # delete one specific stash
```

To empty the entire drawer at once:

```bash
git stash clear                # delete ALL stashes
```

Be careful with `clear` — it removes everything and there is no easy undo. Only
run it when you are sure you do not need any of the stashes.

## Stashing Untracked Files

By default, `git stash` only saves files Git is already **tracking**. Brand-new
files that Git has never seen are left behind. To include them, add `-u` (for
"untracked"):

```bash
git stash push -u -m "wip including new files"
```

If you also want to stash ignored files (the ones in `.gitignore`), use `-a`
for "all", though this is rarely needed.

## When to Use Stashing

Stashing is ideal when:

- You need to **switch branches quickly** and your current work is not ready to
  commit.
- You want to **pull the latest changes** but have local edits in the way.
- You started work on the **wrong branch** and want to move it: stash, switch to
  the right branch, then `git stash pop`.

If your work *is* ready to be saved properly, prefer a real commit (even a
temporary "WIP" commit) over a stash, because commits are easier to track and
share. Stashing is best for short-lived, in-the-moment shelving.

## Common Mistakes

- **Forgetting untracked files.** Plain `git stash` ignores brand-new files.
  Use `-u` to include them.
- **Confusing `apply` and `pop`.** `pop` deletes the stash after restoring;
  `apply` keeps it. If you are unsure, use `apply` first.
- **Running `git stash clear` too soon.** It wipes every stash with no easy
  undo. Drop individual stashes instead unless you truly want all gone.
- **Letting stashes pile up.** A long `git stash list` of unnamed entries gets
  confusing fast. Name them with `-m` and clean up old ones.
- **Treating a stash as permanent storage.** Stashes are for short-term use.
  For anything important or long-lived, make a commit.

## Exercises

1. Edit a tracked file, then run `git stash`. Confirm with `git status` that
   your working tree is clean, then restore it with `git stash pop`.
2. Create two stashes with descriptive messages using `git stash push -m`. List
   them with `git stash list` and confirm both names appear.
3. Create a brand-new file, then stash it using `git stash push -u`. Confirm
   the new file disappears, then bring it back.
4. Make two stashes, then use `git stash apply stash@{1}` to restore the older
   one. Confirm it is still in the list afterward.
5. Explain in your own words when you would stash instead of making a commit.

## Key Takeaways

- `git stash` shelves your uncommitted work and gives you a clean working tree.
- Use `git stash list` to see saved stashes; the newest is `stash@{0}`.
- `apply` restores and keeps the stash; `pop` restores and removes it.
- `git stash push -u -m "name"` stashes untracked files with a useful name.
- Stashing is perfect for quickly switching branches; for important work, make
  a real commit instead.

---

[← Previous: Cherry-Picking](02-cherry-pick.md) | [Back to Course Index](../README.md) | [Next: Reflog and Recovery →](04-reflog-and-recovery.md)
