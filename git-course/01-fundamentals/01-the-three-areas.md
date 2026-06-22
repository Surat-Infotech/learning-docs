# The Three Areas: Working Directory, Staging, and Repository

Git keeps a history of your project as a series of **snapshots** — like taking a
photo of all your files at a moment in time. To understand Git, you only need one
mental model: your files move through **three areas**. Think of it like packing a
parcel: you work on things on your desk, place the ones you want to send into a
box, then seal and ship the box.

## What You'll Learn

- The three areas Git uses: working directory, staging area, and repository
- How a file flows from modified, to staged, to committed
- What a **commit** actually is and what it contains
- Why Git uses a separate staging area at all
- What lives inside the hidden `.git` folder
- The vocabulary you will see again and again: tracked, untracked, staged

## The Big Picture

Git organizes your project into three areas. A file moves from one area to the
next as you work.

```text
   Working Directory          Staging Area              Repository
   (your files /            (the box you are          (sealed snapshots /
    your desk)               packing)                  commit history)

   edit a file  ──── git add ────▶  file is staged  ── git commit ──▶  snapshot saved
```

Let's meet each area.

### 1. Working Directory (your desk)

The **working directory** is the folder on your computer where your real files
live. It is your desk: the place where you actually edit, create, and delete
files. When you open a file in your editor and change it, you are changing the
working directory.

Git watches this folder. The moment you change a tracked file, Git notices and
calls it **modified**.

### 2. Staging Area (the box you are packing)

The **staging area** (also called the **index**) is a holding zone. It is the box
you pack *before* you ship. You choose exactly which changes go into the box, one
at a time if you like.

This is the part beginners often find surprising: just because you changed a file
does **not** mean Git will save it. You first place it in the box with `git add`.
A change sitting in the box is called **staged**.

Why a box at all? Because it lets you group related changes together. You might
edit five files but only want to save two of them as one logical change. The
staging area gives you that control.

### 3. Repository (sealed, shipped snapshots)

The **repository** is Git's permanent history — a stack of sealed snapshots. When
you run `git commit`, Git seals the box and stores it forever as a **commit**.
Each commit is a complete snapshot of your project at that moment, plus a label
explaining what changed.

Once a change is committed, it is safe. It is part of the project's history.

## How a File Flows

Here is the journey of a single change, step by step.

```text
1. You edit report.txt          → it becomes MODIFIED (in working directory)
2. git add report.txt           → it becomes STAGED   (in the box)
3. git commit -m "Update report" → it becomes COMMITTED (sealed in history)
```

In commands:

```bash
# You just edited report.txt in your editor.
git status              # Git shows report.txt as modified
git add report.txt      # put the change in the box (staging area)
git commit -m "Update report"  # seal the box: save a snapshot
```

That cycle — **modify, stage, commit** — is the heartbeat of Git. You will repeat
it thousands of times.

## What Is a Commit, Exactly?

A **commit** is one sealed snapshot. It is not just "the files that changed" — it
is a complete picture of your whole project at that point. Each commit stores:

- **A snapshot** of all your files at that moment.
- **A message** you write describing what changed (for example, "Add login page").
- **An author** — who made it, and when.
- **A parent** — a pointer to the commit that came before it. This is how Git
  links commits into a chain, forming your history.

```text
   A ◀── B ◀── C        (each commit points back to its parent)
 first       latest
```

Every commit also has a unique ID called a **hash** — a long string like
`9f2c1a7...`. You can use that ID to look at, or return to, any past snapshot.

## Tracked vs Untracked Files

Two more words you'll see constantly:

- **Tracked**: a file Git already knows about (you have added or committed it
  before). Git watches tracked files for changes.
- **Untracked**: a brand-new file Git has never seen. Git ignores it until you
  `git add` it for the first time.

```bash
git status
# Untracked files:
#   notes.txt          <- new file, Git is not watching it yet
```

## The `.git` Folder

When you turn a folder into a Git project (you'll do this next lesson with
`git init`), Git creates a hidden folder called `.git` inside it.

```text
my-project/
├── .git/          <- Git's brain: history, staging area, settings
├── report.txt
└── notes.txt
```

The `.git` folder is where Git stores **everything**: every commit, the staging
area, branches, and configuration. Your working directory is just the files you
see; the `.git` folder is the actual repository.

A simple rule: **never edit or delete the `.git` folder by hand.** If you delete
it, you delete your entire history. Let Git manage it through commands.

## Common Mistakes

- **Thinking editing a file saves it in Git.** Editing only changes the working
  directory. You must `git add` and then `git commit` to save a snapshot.
- **Forgetting the staging step.** Many beginners expect `git commit` to grab all
  changes automatically. By default, it only saves what is staged.
- **Confusing "staged" with "committed."** Staged means "in the box." Committed
  means "the box is sealed and in history."
- **Touching the `.git` folder.** Treat it as off-limits; let Git manage it.

## Exercises

1. Draw the three areas on paper from memory, and write the command that moves a
   file from each area to the next.
2. In your own words, explain the difference between *modified*, *staged*, and
   *committed*. Pretend you are explaining it to a friend.
3. List the four things stored inside a single commit.
4. Explain why Git uses a separate staging area instead of committing every change
   automatically. Give one real example where this control is useful.

## Key Takeaways

- Git has three areas: the **working directory** (your files), the **staging
  area** (the box you pack), and the **repository** (sealed snapshots).
- A change flows **modified → staged → committed**, using `git add` then
  `git commit`.
- A **commit** is a full snapshot plus a message, an author, and a parent pointer
  to the previous commit.
- Files are **tracked** (Git watches them) or **untracked** (Git ignores them).
- The hidden `.git` folder is the real repository — never edit it by hand.

---

[← Previous: Installing and Configuring Git](../00-getting-started/02-installing-and-configuring-git.md) | [Back to Course Index](../README.md) | [Next: Your First Repository →](02-your-first-repository.md)
