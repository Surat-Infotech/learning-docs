# Interactive Rebase

Sometimes your commit history gets messy. Maybe you made five tiny commits
like "fix typo", "fix typo again", "oops" while building one feature. Before
you share that work, it would be nice to tidy it up into one clean commit.

Interactive rebase is the tool for that. Think of it as an **editor for your
recent history**. You open a list of your last few commits, and Git lets you
reorder them, combine them, rename them, or even drop them entirely. When you
save and close, Git rebuilds your history exactly the way you described.

This sounds powerful and a little scary, but it is completely safe once you
understand it, and Git keeps a backup (the reflog) if anything goes wrong. We
will go slowly.

## What You'll Learn

- What interactive rebase does and when to use it
- How to start it with `git rebase -i HEAD~n`
- The action keywords: `pick`, `reword`, `edit`, `squash`, `fixup`, `drop`
- How to squash several commits into one clean commit
- How to reword a commit message
- How to reorder commits
- The golden rule: never rewrite history others already have
- How to finish with `--continue` or back out with `--abort`

## What Is Rebasing?

To **rebase** means to take a series of commits and replay them on top of a new
base. Interactive rebase adds an extra step: before replaying, it shows you the
list of commits and lets you decide what to do with each one.

The word "interactive" just means *Git pauses and asks you what you want*.

## Starting an Interactive Rebase

You tell Git how many commits back you want to edit using `HEAD~n`, where `n`
is the number of commits.

```bash
# Edit the last 3 commits
git rebase -i HEAD~3
```

`HEAD` means "where I am right now". `HEAD~3` means "three commits before
that". So this opens an editor showing your three most recent commits.

When you run it, your text editor opens with a **todo list** that looks like
this:

```text
pick a1b2c3d Add login form
pick d4e5f6g Fix typo in label
pick h7i8j9k Fix another typo

# Rebase 9z8y7x6..h7i8j9k onto 9z8y7x6 (3 commands)
#
# Commands:
# p, pick   = use commit
# r, reword = use commit, but edit the commit message
# e, edit   = use commit, but stop for amending
# s, squash = use commit, but meld into previous commit
# f, fixup  = like squash, but discard this commit's log message
# d, drop   = remove commit
#
# These lines can be re-ordered; they are executed from top to bottom.
```

Notice the commits are listed **oldest at the top**, which is the reverse of
`git log`. The lines starting with `#` are comments explaining your choices.

## The Action Keywords

Each line starts with an action word. You change history by editing these words
(not the commit hashes).

- **pick** — keep this commit exactly as it is. This is the default.
- **reword** — keep the commit, but let me edit its message.
- **edit** — stop here so I can change the commit's actual content.
- **squash** — combine this commit into the one above it, keeping both messages.
- **fixup** — combine this commit into the one above it, but throw away this
  commit's message.
- **drop** — delete this commit entirely (you can also just delete the line).

You can also **reorder** the lines, and Git will apply them in the new order.

## Example: Squashing Commits Into One

Say your three commits should really be a single commit. Change the todo list
to use `squash` (or its short form `s`) on the lower commits:

```text
pick a1b2c3d Add login form
squash d4e5f6g Fix typo in label
squash h7i8j9k Fix another typo
```

Save and close. Git combines all three into one commit and opens a new editor
so you can write the final message:

```text
# This is a combination of 3 commits.
Add login form

# (you can delete the typo lines and keep one clean message)
```

Clean it up to just `Add login form`, save, and you now have one tidy commit
instead of three.

If you do not care about the messages of the squashed commits at all, use
`fixup` instead of `squash`. It silently merges them and keeps only the first
message, so Git does not even ask you to edit anything.

```text
pick a1b2c3d Add login form
fixup d4e5f6g Fix typo in label
fixup h7i8j9k Fix another typo
```

## Example: Rewording a Message

Suppose a commit message has a mistake. Use `reword` (or `r`):

```text
pick a1b2c3d Add login form
reword d4e5f6g Fxi typo in label
pick h7i8j9k Fix another typo
```

Save and close. Git replays the first commit, then stops and opens an editor
with the second commit's message so you can fix `Fxi` to `Fix`. Save again and
Git continues.

## Example: Reordering Commits

Just move the lines around. To put the "Add login form" commit last:

```text
pick d4e5f6g Fix typo in label
pick h7i8j9k Fix another typo
pick a1b2c3d Add login form
```

Git will replay them in the new top-to-bottom order. Reordering can cause a
conflict if the commits touch the same lines — that is normal, and we handle it
below.

## The Golden Rule of Rebasing

Interactive rebase **rewrites** commits, which means it creates brand-new
commits with new hashes. The old commits are replaced.

> **Never rebase commits you have already pushed and shared with others.**

If a teammate already has the old commits and you rewrite them, your histories
will disagree, and merging becomes a painful mess. Only rebase commits that are
still **local and private** to you — work you have not pushed yet.

A simple test: if the commits exist only on your machine, rebase freely. If
they are on a shared branch like `main`, leave them alone.

## Finishing or Backing Out

After you save the todo list, Git replays each commit. Two things can happen:

If everything applies cleanly, the rebase finishes and you are done.

If Git hits a **conflict** (two changes clash on the same lines), it pauses and
asks you to fix it. Edit the conflicting files, then:

```bash
git add <fixed-file>      # mark the conflict resolved
git rebase --continue     # carry on with the rebase
```

If you decide you do not want to continue at all, you can completely undo the
rebase and return to where you started:

```bash
git rebase --abort        # cancel everything, restore original history
```

`--abort` is your safety button. As long as you have not finished, nothing is
lost.

## Common Mistakes

- **Editing the commit hashes** instead of the action words. Leave the hashes
  alone; only change `pick` to another keyword.
- **Squashing onto nothing.** The very top line cannot be `squash` or `fixup`,
  because there is no commit above it to merge into.
- **Rebasing shared history.** Rewriting pushed commits forces teammates into
  conflicts. Follow the golden rule.
- **Panicking during a conflict.** A conflict is not an error. Fix the file,
  `git add` it, and run `git rebase --continue`. Or `--abort` to start over.
- **Forgetting to save and close the editor.** Git waits for you. Nothing
  happens until you save the todo file.

## Exercises

1. Make three small commits in a test repo (for example, add a line to a file
   three times). Run `git rebase -i HEAD~3` and squash them all into one commit
   with a clean message. Verify with `git log --oneline`.
2. Use `reword` to fix a deliberate typo you put in a commit message.
3. Reorder two commits using interactive rebase, then look at `git log` to
   confirm the new order.
4. Start a rebase, then immediately run `git rebase --abort`. Confirm your
   history is unchanged.
5. Explain in your own words why you should not rebase commits you have already
   pushed to a shared branch.

## Key Takeaways

- Interactive rebase (`git rebase -i HEAD~n`) is an editor for recent history.
- The todo list lets you `pick`, `reword`, `edit`, `squash`, `fixup`, `drop`,
  and reorder commits.
- `squash` keeps messages to merge; `fixup` discards the extra message.
- Rebasing creates new commits, so follow the golden rule: never rewrite
  history others already have.
- `git rebase --continue` carries on after fixing a conflict; `git rebase
  --abort` safely cancels everything.

---

[← Previous: Resolving Conflicts in a Team](../04-collaboration/04-team-conflicts.md) | [Back to Course Index](../README.md) | [Next: Cherry-Picking →](02-cherry-pick.md)
