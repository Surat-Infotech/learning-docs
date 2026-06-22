# Viewing History: log, show, and diff

Every commit you make is a save point in your project's story. Now you'll learn to
read that story: list past snapshots, open any one of them, and compare versions to
see exactly what changed. Think of it like flipping through a photo album and
spotting the differences between two photos.

## What You'll Learn

- How to list your commit history with `git log`
- Useful `git log` flags: `--oneline`, `--graph`, `-n`, `--stat`
- What a commit hash is and how to read it
- How to inspect a single commit with `git show`
- How to compare versions with `git diff` (working vs staged vs committed)
- How to escape Git's pager when output fills the screen

## Listing History with `git log`

`git log` shows your commits, newest first. Run it in a repository with at least
one commit:

```bash
git log
```

```text
commit a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9b0 (HEAD -> main)
Author: Maya <maya@example.com>
Date:   Mon Jun 22 10:30:00 2026 +0000

    Add todo list with two tasks

commit 9f8e7d6c5b4a3210fedcba9876543210abcdef12
Author: Maya <maya@example.com>
Date:   Mon Jun 22 10:15:00 2026 +0000

    Add README with project title
```

Each entry shows the **hash**, the **author**, the **date**, and the **message**.

### Escaping the Pager: Press `q`

If the history is long, Git shows it in a **pager** — a scrolling viewer that
takes over your terminal. You'll see a `:` at the bottom. To get out:

- Press **`q`** to quit and return to your prompt.
- Use the **arrow keys** or **Space** to scroll while inside it.

If you ever feel "stuck" after a Git command, press `q`.

## Reading a Commit Hash

A **hash** is a commit's unique ID — a 40-character string like
`a1b2c3d4e5f6...`. You rarely type the whole thing. Git accepts the **first 7 or
so characters** as a shortcut, as long as they're unique:

```bash
git show a1b2c3d        # same as typing the full hash
```

`HEAD` is a special name meaning "the commit I'm currently on" (usually the latest
one). You'll see it in the log as `HEAD -> main`.

## Cleaner Logs with Flags

The full log is verbose. These flags make it far more readable.

### `--oneline` — one commit per line

```bash
git log --oneline
```

```text
a1b2c3d Add todo list with two tasks
9f8e7d6 Add README with project title
```

Short hash plus message — perfect for a quick overview.

### `-n` — limit how many commits show

```bash
git log -n 1        # show only the most recent commit
git log -3          # show the last 3 (the n is optional)
```

### `--stat` — show which files changed

```bash
git log --stat
```

```text
a1b2c3d Add todo list with two tasks
 todo.txt | 2 ++
 1 file changed, 2 insertions(+)
```

The `++` and numbers tell you how many lines were added or removed.

### `--graph` — draw the history as a diagram

```bash
git log --oneline --graph
```

```text
* a1b2c3d Add todo list with two tasks
* 9f8e7d6 Add README with project title
```

The `--graph` flag really shines once you have branches (you'll learn those soon);
it draws the lines that connect commits. A popular combination to remember:

```bash
git log --oneline --graph --all
```

## Inspecting One Commit with `git show`

`git log` lists commits. `git show` opens **one** commit so you can see exactly
what it changed.

```bash
git show a1b2c3d
```

```text
commit a1b2c3d... (HEAD -> main)
Author: Maya <maya@example.com>
Date:   Mon Jun 22 10:30:00 2026 +0000

    Add todo list with two tasks

diff --git a/todo.txt b/todo.txt
new file mode 100644
index 0000000..e69de29
+++ b/todo.txt
@@ -0,0 +1,2 @@
+Buy milk
+Walk dog
```

The lines starting with `+` were added. Lines starting with `-` (in other commits)
were removed. With no hash, `git show` shows the most recent commit:

```bash
git show          # shows HEAD, the latest commit
```

## Comparing Versions with `git diff`

`git diff` shows **differences** — the changes that have not yet been committed.
Which differences depends on the flag.

### `git diff` — working directory vs staging

Plain `git diff` shows changes you've made but **have not staged** yet.

```bash
echo "Read a book" >> todo.txt   # change a file but don't add it
git diff
```

```text
diff --git a/todo.txt b/todo.txt
index e69de29..f1a2b3c 100644
--- a/todo.txt
+++ b/todo.txt
@@ -1,2 +1,3 @@
 Buy milk
 Walk dog
+Read a book
```

The `+Read a book` line is your new, unstaged change.

### `git diff --staged` — staging vs last commit

Once you stage a change, plain `git diff` shows nothing (there are no *unstaged*
changes). To see what's **in the box**, use `--staged`:

```bash
git add todo.txt
git diff             # shows nothing now — everything is staged
git diff --staged    # shows the staged change vs the last commit
```

(`--cached` is an older alias for `--staged`; they do the same thing.)

A handy way to remember it:

```text
git diff            → working directory  vs  staging   (unstaged changes)
git diff --staged   → staging            vs  last commit (staged changes)
```

### Comparing Two Commits

You can compare any two commits by giving their hashes, oldest first:

```bash
git diff 9f8e7d6 a1b2c3d      # what changed between these two commits
```

You can also compare a commit to your current files:

```bash
git diff 9f8e7d6             # changes since that commit, up to now
```

## Common Mistakes

- **Feeling stuck in the pager.** That `:` at the bottom is the pager. Press `q`
  to leave.
- **Expecting `git diff` to show staged changes.** Plain `git diff` hides staged
  changes; use `git diff --staged` to see what's in the box.
- **Confusing `git log` and `git show`.** `log` lists many commits; `show` opens
  one commit's details.
- **Typing full 40-character hashes.** The first 7 characters are usually enough.
- **Getting the commit order backwards in `git diff A B`.** Git shows what changed
  going *from* A *to* B, so put the older commit first.

## Exercises

1. Run `git log`, then `git log --oneline`. Describe in one sentence how the output
   differs and when each is more useful.
2. Use `git log --stat` and identify how many lines changed in your most recent
   commit.
3. Copy the short hash of any commit and run `git show <hash>`. Find the lines
   marked with `+` and explain what they mean.
4. Make a change to a file but do not stage it, and run `git diff`. Then stage it
   and run both `git diff` and `git diff --staged`. Explain why the output moved
   from one command to the other.
5. Compare two of your commits with `git diff <older> <newer>` and summarize what
   changed between them.

## Key Takeaways

- `git log` lists commits newest-first; `--oneline`, `--graph`, `-n`, and `--stat`
  make it more readable.
- A commit **hash** is its unique ID; the first ~7 characters usually suffice, and
  `HEAD` means the current commit.
- `git show <hash>` opens one commit and shows exactly what it changed.
- `git diff` shows **unstaged** changes; `git diff --staged` shows **staged**
  changes vs the last commit.
- You can compare any two commits with `git diff <older> <newer>`.
- Press `q` to exit Git's pager.

---

[← Previous: Your First Repository](02-your-first-repository.md) | [Back to Course Index](../README.md) | [Next: Undoing Changes →](04-undoing-changes.md)
