# Your First Repository

Time to create your very first Git project and save your first snapshot. Think of
this lesson as learning the daily rhythm of Git: make a change, pack it in the
box, seal it. By the end you will have done the full cycle yourself, start to
finish.

## What You'll Learn

- How to turn any folder into a Git repository with `git init`
- How to check the state of your project with `git status` (your best friend)
- How to add files to the staging area with `git add`
- How to save a snapshot with `git commit -m`
- How to write a clear, useful commit message
- The full staging → commit cycle, with `git status` at every step

## Starting a Repository with `git init`

To start tracking a project, go into its folder and run `git init`. This is the
command that creates the hidden `.git` folder you learned about last lesson.

```bash
mkdir my-first-repo      # make a new folder
cd my-first-repo         # go inside it
git init                 # turn it into a Git repository
```

Expected output:

```text
Initialized empty Git repository in /Users/you/my-first-repo/.git/
```

That's it — this folder is now a Git repository. The repository is **empty** for
now because you have not saved any snapshots yet.

## `git status` — Your Best Friend

`git status` tells you the current state of your project: what's changed, what's
staged, what's untracked. Run it constantly. When in doubt, run `git status`.

```bash
git status
```

On a brand-new repo you'll see:

```text
On branch main

No commits yet

nothing to commit (working tree clean)
```

"Nothing to commit" means there's nothing new to save. Let's change that.

## Creating a File

Make a simple file so we have something to track.

```bash
echo "# My First Repo" > README.md   # create README.md with one line
git status
```

Now `git status` shows:

```text
On branch main

No commits yet

Untracked files:
  (use "git add <file>..." to include in what will be committed)
	README.md

nothing added to commit but untracked files present
```

`README.md` is **untracked** — Git sees it but isn't watching it yet. Notice Git
even tells you the next command to run. Git is friendly like that; read its
hints.

## Staging with `git add`

`git add` puts changes into the staging area (the box). You can add one file,
several files, or everything at once.

```bash
# Add a single file
git add README.md

# Add several specific files
git add README.md notes.txt

# Add everything that changed in the current folder
git add .
```

After staging the README:

```bash
git add README.md
git status
```

```text
On branch main

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)
	new file:   README.md
```

"Changes to be committed" means the file is **staged** — it's in the box, ready
to be sealed. A quick tip on `git add .`: it stages everything, which is handy but
can accidentally include files you didn't mean to. When you're learning, adding
files by name keeps you in control.

## Committing with `git commit -m`

`git commit` seals the box into a permanent snapshot. The `-m` flag lets you write
the message right on the command line.

```bash
git commit -m "Add README with project title"
```

Expected output:

```text
[main (root-commit) a1b2c3d] Add README with project title
 1 file changed, 1 insertion(+)
 create mode 100644 README.md
```

That `a1b2c3d` is the start of the commit's unique hash. Your first commit is
saved. Check the state again:

```bash
git status
```

```text
On branch main
nothing to commit, working tree clean
```

"Working tree clean" means every change is safely committed. This is the calm,
happy state you want to return to.

## Writing Good Commit Messages

A commit message explains *what changed and why*. Future you (and your teammates)
will read these. A good habit:

- Write a **short summary line** (about 50 characters or fewer).
- Use the **imperative mood** — write it like a command: "Add", "Fix", "Update",
  not "Added" or "Adds". A trick: it should finish the sentence *"If applied, this
  commit will ___."*

```bash
# Good
git commit -m "Add login form validation"
git commit -m "Fix typo in homepage heading"

# Less helpful
git commit -m "stuff"
git commit -m "changes"
git commit -m "asdfgh"
```

Clear messages turn your history into a readable story of the project.

## The Full Cycle: A Worked Example

Let's do the whole rhythm again, watching `git status` at each step.

```bash
# Step 1: create and change files in the working directory
echo "Buy milk" > todo.txt
echo "Walk dog" >> todo.txt
git status
```

```text
Untracked files:
	todo.txt
```

```bash
# Step 2: stage the change (put it in the box)
git add todo.txt
git status
```

```text
Changes to be committed:
	new file:   todo.txt
```

```bash
# Step 3: commit (seal the box)
git commit -m "Add todo list with two tasks"
git status
```

```text
On branch main
nothing to commit, working tree clean
```

You just completed the full **modify → stage → commit** cycle. Now edit the file
again to see the cycle from the "modified" side:

```bash
echo "Read a book" >> todo.txt   # change a tracked file
git status
```

```text
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
	modified:   todo.txt
```

Because `todo.txt` is now tracked, Git calls it **modified** (not untracked). Stage
and commit it the same way:

```bash
git add todo.txt
git commit -m "Add 'read a book' to todo list"
```

That rhythm — `status`, `add`, `commit`, `status` — is Git in everyday use.

## Common Mistakes

- **Forgetting `git init`.** Git commands won't work until the folder is a
  repository. Run `git status`; if it says "not a git repository," run `git init`.
- **Committing without staging.** A plain `git commit` only saves staged changes.
  If nothing is staged, Git tells you "nothing to commit."
- **Vague commit messages** like "stuff" or "fix." Describe the actual change.
- **Overusing `git add .`** and accidentally staging files you didn't intend.
  Check `git status` before committing.
- **Forgetting the `-m`.** Without `-m`, Git opens a text editor for the message,
  which can confuse beginners. Use `-m "your message"` to stay on the command line.

## Exercises

1. Create a new folder, run `git init`, and confirm with `git status` that it's an
   empty repository with no commits yet.
2. Create a file called `about.txt`, then run `git status` and identify it as
   untracked. Stage it and run `git status` again to see it become "to be
   committed."
3. Commit `about.txt` with a clear imperative message. Then run `git status` and
   confirm the working tree is clean.
4. Edit `about.txt`, observe that Git now calls it *modified* (not untracked), and
   complete the cycle by staging and committing the change.
5. Make three small commits in a row, each with a good message. Read your messages
   back and check they each finish the sentence "If applied, this commit will ___."

## Key Takeaways

- `git init` turns a folder into a Git repository by creating the `.git` folder.
- `git status` shows the current state — run it often; it's your best friend.
- `git add` stages changes (one file, several, or `.` for everything).
- `git commit -m "message"` saves a snapshot of the staged changes.
- Good commit messages are short and written in the imperative ("Add", "Fix").
- The everyday rhythm is **status → add → commit → status**.

---

[← Previous: The Three Areas of Git](01-the-three-areas.md) | [Back to Course Index](../README.md) | [Next: Viewing History →](03-viewing-history.md)
