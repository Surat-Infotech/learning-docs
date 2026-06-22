# Understanding Branches

Imagine you are writing a story, and you reach a point where you want to try a different ending — but you are not sure it will work. You do not want to scribble over your original. So you grab a fresh sheet of paper, copy the last line, and continue the new ending there. If you like it, you keep it. If not, you throw that sheet away and your original is untouched.

That fresh sheet is exactly what a **branch** is in Git: a separate, parallel timeline of your work where you can experiment safely without disturbing the main version.

In this lesson we will look closely at what a branch *really* is under the hood. Spoiler: it is much simpler and lighter than most people expect.

## What You'll Learn

- What a branch actually is (a tiny movable pointer to a commit)
- Why branches are so cheap and fast in Git
- Why branches matter — isolating work and experimenting safely
- What `HEAD` means and how it tracks "where you are"
- What the default `main` branch is
- How to picture commits and branch pointers in a diagram
- The difference between a branch and the work it points to

## A Quick Reminder About Commits

Before we talk about branches, remember what a **commit** is: a saved snapshot of your project at a moment in time. Every commit has a unique ID (a long string like `a1b2c3d`) and a link back to the commit that came before it (its **parent**).

So a series of commits forms a chain, like beads on a string:

```text
A ──> B ──> C
```

Here commit `C` points back to `B`, and `B` points back to `A`. This chain is your project's history.

## So What Is a Branch?

Here is the surprising part. A **branch is not a copy of your files**. A branch is simply a **lightweight, movable pointer to one commit** — usually the most recent one.

Think of it as a sticky note with a name on it, stuck onto a particular commit:

```text
A ──> B ──> C   <── main
```

The label `main` is a branch. Right now it points at commit `C`. That is the *entire* branch — just a name pointing at a commit.

Because a branch is only a pointer (literally a tiny file containing one commit ID), creating a new branch is instant and costs almost nothing. This is why people say Git branches are "cheap."

### Branches Move

When you are on a branch and you make a new commit, the branch pointer **moves forward** to that new commit automatically.

Before the new commit:

```text
A ──> B ──> C   <── main
```

You make a new commit `D`. Now:

```text
A ──> B ──> C ──> D   <── main
```

The `main` label slid forward from `C` to `D`. You did not have to move it yourself — Git did it for you. That is what "movable pointer" means.

## What Is HEAD?

If a branch is a pointer to a commit, then **`HEAD`** is a pointer to *the branch you are currently on*. It answers the question: "Where am I right now?"

```text
A ──> B ──> C   <── main   <── HEAD
```

This reads as: `HEAD` points to `main`, and `main` points to commit `C`. So you are currently on the `main` branch, sitting at commit `C`.

When you switch to a different branch, `HEAD` simply moves to point at that other branch. Your files on disk change to match wherever `HEAD` now points.

You can see what `HEAD` points to:

```bash
# Show the current branch name
git branch --show-current
# Output: main
```

```bash
# A more detailed view of where HEAD is
git status
# Output: On branch main
```

## The Default Branch: main

When you create a new repository, Git automatically makes one branch for you and checks it out. By modern convention this branch is named **`main`**.

```bash
# After git init, you are already on main
git status
# Output: On branch main
#         No commits yet
```

> Note: Older projects and tutorials may call this branch `master` instead of `main`. They behave identically — it is just a name. Newer versions of Git use `main` by default.

There is nothing technically special about `main`. It is an ordinary branch. By tradition, teams treat it as the "official," trusted version of the project — the one that should always work. Experimental work happens on *other* branches and is only merged into `main` once it is ready.

## Why Branches Matter

Here are the everyday reasons branches are so useful:

### 1. Isolate Your Work

Suppose you are adding a login feature. You create a branch called `login-feature` and do all your work there. Meanwhile, `main` stays exactly as it was — clean and working. If a teammate needs the stable version, it is right there on `main`, untouched by your half-finished code.

### 2. Experiment Safely

Want to try a risky idea? Make a branch. If the experiment fails, you delete the branch and it is as if it never happened. Your real work is safe.

```text
                  E ──> F   <── crazy-experiment
                 /
A ──> B ──> C ──> D   <── main
```

The experiment lives on its own line. If `crazy-experiment` turns out badly, deleting it leaves `main` perfectly intact.

### 3. Work on Many Things at Once

Different branches let you keep separate tasks separate. A bug fix on one branch, a new feature on another — neither interferes with the other.

## A Fuller Diagram

Let's put it all together. Imagine a project where `main` has three commits, and someone started a `feature` branch from commit `B` and added two more commits:

```text
                  D ──> E   <── feature
                 /
A ──> B ──> C   <── main   <── HEAD
```

Reading this diagram:

- The shared history is `A ──> B`.
- `main` continued to `C`.
- `feature` branched off at `B` and continued to `D ──> E`.
- `HEAD` is on `main`, so right now your files match commit `C`.

Notice that both branches **share** commits `A` and `B`. Git does not duplicate them. Branches diverge only where the work actually differs.

## Branches Are Not Folders

A common beginner mental model is that switching branches copies files into a different folder. That is not what happens. There is one working folder. When you switch branches, Git *rewrites the files in that same folder* to match whatever commit the new branch points to. Switch back, and Git rewrites them again.

So you always work in one place — Git just swaps the contents to match wherever `HEAD` is pointing.

## Common Mistakes

- **Thinking a branch is a heavy copy of all your files.** It is just a pointer to one commit. Creating one is instant.
- **Confusing `HEAD` with a commit.** `HEAD` normally points to a *branch*, and that branch points to a commit. It is a pointer to a pointer.
- **Believing `main` is magical.** It is an ordinary branch that teams *agree* to treat as the official one. Git does not enforce that.
- **Expecting branches to copy files into different folders.** There is one working folder; Git swaps its contents when you switch.
- **Forgetting which branch you are on.** Always check with `git status` or `git branch --show-current` before committing.

## Exercises

1. In your own words, explain why creating a branch in Git is so fast. What is actually being created?
2. Draw (on paper or in a text file) a diagram showing a `main` branch with commits `A ──> B ──> C`, then a `feature` branch starting at `B` with commits `D ──> E`. Mark where `HEAD` would be if you are currently on `feature`.
3. Run `git status` and `git branch --show-current` in an existing repository. What branch are you on?
4. Explain the relationship between `HEAD`, a branch, and a commit in a single sentence.
5. Describe a real situation from your own projects (coding or not) where having a separate "parallel timeline" of work would have helped.

## Key Takeaways

- A **branch** is a lightweight, movable pointer to a single commit — not a copy of your files.
- Because it is just a pointer, creating a branch is instant and cheap.
- When you commit on a branch, the branch pointer moves forward to the new commit automatically.
- **`HEAD`** points to the branch you are currently on; that branch points to a commit.
- **`main`** is the default branch and is conventionally treated as the official, stable version.
- Branches let you **isolate work** and **experiment safely** without touching your main work.
- Branches share commits up to the point where they diverge — history is not duplicated.

---

[← Previous: Ignoring Files with .gitignore](../01-fundamentals/05-gitignore.md) | [Back to Course Index](../README.md) | [Next: Creating and Switching Branches →](02-creating-and-switching-branches.md)
