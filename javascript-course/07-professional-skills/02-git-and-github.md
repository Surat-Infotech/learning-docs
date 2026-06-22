# Git and GitHub Basics

Imagine writing an essay and saving copies named "final", "final2", "final-really-final". That gets messy fast. **Git** solves this problem for code. It saves the history of your project so you can go back in time, try new ideas safely, and work with others. This lesson teaches the basics.

## What You'll Learn

- What version control is and why it matters
- The difference between Git and GitHub
- The basic workflow: `init`, `status`, `add`, `commit`
- What the staging area is
- How `.gitignore` works
- Branches explained simply
- Connecting to GitHub and pushing your code
- A typical daily flow
- How to write good commit messages

## What Is Version Control?

**Version control** is a system that records changes to your files over time. With it you can:

- See the full history of your project.
- Go back to an older version if you break something.
- Try new ideas without fear, because you can always undo.
- Work with other people without overwriting each other's work.

The most popular version control tool is **Git**.

## Git vs GitHub

People mix these up, so let's be clear:

- **Git** is the tool that runs on *your computer*. It tracks your code history locally.
- **GitHub** is a *website* where you can store your Git projects online and share them with others.

Think of Git as the engine and GitHub as a place to park your car online so others can see it. There are other similar websites too (GitLab, Bitbucket), but GitHub is the most common.

## The Basic Workflow

Here is the core loop you will use all the time.

### Step 1: Start a Repository

A **repository** (or "repo") is just a project folder that Git is watching.

```bash
# Turn the current folder into a Git repository
git init
```

You only run this once per project.

### Step 2: Check the Status

```bash
# See which files have changed and what is staged
git status
```

This is your most-used command. Run it often to see what's going on.

### Step 3: Stage Your Changes

```bash
# Stage one file
git add index.js

# Stage everything that changed
git add .
```

### Step 4: Commit Your Changes

A **commit** is a saved snapshot of your project at a moment in time.

```bash
# Save the staged changes with a message describing them
git commit -m "Add login button"
```

The `-m` lets you write a short message explaining what you did.

## The Staging Area Explained Simply

Between your files and a commit there is a middle step called the **staging area**. Think of it like a shopping cart:

1. You change files (items on the shelf).
2. `git add` puts files into the cart (the staging area).
3. `git commit` buys everything in the cart (saves the snapshot).

The staging area lets you choose *exactly* which changes go into each commit. You don't have to commit everything at once.

## .gitignore

Some files should never be saved in Git. The biggest example is the `node_modules` folder (it's huge and easy to rebuild). To ignore files, create a file named **.gitignore** in your project and list them:

```bash
# Inside .gitignore

# Ignore installed packages
node_modules

# Ignore secret settings
.env

# Ignore log files
*.log
```

Git will now pretend these files don't exist and won't track them.

## Branches Simply

A **branch** is like a separate copy of your project where you can try changes safely. The main branch (often called `main`) stays clean. You make a new branch, experiment, and only merge it back when it works.

```bash
# Create a new branch
git branch new-feature

# Switch to it (newer, recommended command)
git switch new-feature

# Older command that also switches branches
git checkout new-feature

# Create AND switch in one step
git switch -c new-feature
```

When your work is ready, you bring it back into `main`:

```bash
# First switch back to main
git switch main

# Then merge your branch into main
git merge new-feature
```

**Merging** means combining the changes from one branch into another. Branches let many people work on different features at the same time without stepping on each other.

## Connecting to GitHub

To put your project online, you link it to a GitHub repository. A **remote** is the online location of your project.

```bash
# Connect your local repo to a GitHub repo (URL comes from GitHub)
git remote add origin https://github.com/yourname/your-repo.git

# Send your commits to GitHub (the first time use -u)
git push -u origin main

# After the first time, you can just run:
git push

# Get the latest changes from GitHub onto your computer
git pull
```

`origin` is just the standard nickname for your main remote.

If a project already exists on GitHub and you want a copy, you **clone** it:

```bash
# Download a full copy of an online repo to your computer
git clone https://github.com/someone/their-repo.git
```

## A Typical Daily Flow

Here's how a normal day of coding with Git looks:

```bash
# 1. Get the latest changes from GitHub
git pull

# 2. Do your work in your code editor...

# 3. Check what changed
git status

# 4. Stage your changes
git add .

# 5. Save a snapshot with a clear message
git commit -m "Fix bug in cart total"

# 6. Send your work to GitHub
git push
```

Repeat this loop every day. It quickly becomes second nature.

## Writing Good Commit Messages

A good commit message tells the reader what changed and why, in a short clear sentence.

**Good messages:**

```bash
git commit -m "Add password validation to signup form"
git commit -m "Fix crash when cart is empty"
```

**Bad messages:**

```bash
git commit -m "stuff"
git commit -m "asdf"
git commit -m "changes"
```

Tips:

- Start with a verb: *Add*, *Fix*, *Update*, *Remove*.
- Keep it short but specific.
- Describe the change, not "did some work".

Good messages help future you (and your teammates) understand the history.

## Common Mistakes

- **Forgetting to `git add` before committing.** Commits only save staged files.
- **Committing node_modules or secrets.** Always set up `.gitignore` first.
- **Writing vague messages like "fix".** Be specific.
- **Forgetting to `git pull` before starting.** You may work on old code and cause conflicts.
- **Pushing to the wrong branch.** Run `git status` to confirm which branch you're on.

## Exercises

1. Create a new folder, run `git init`, add a file, and make your first commit with a clear message.
2. Create a `.gitignore` file that ignores `node_modules` and a `.env` file. Confirm with `git status` that they are ignored.
3. Create a branch called `experiment`, switch to it, make a change and commit it, then switch back to `main` and merge it.
4. Create a repository on GitHub, connect it with `git remote add origin`, and push your project.

## Key Takeaways

- **Version control** records your project's history so you can undo, experiment, and collaborate.
- **Git** runs on your computer; **GitHub** stores projects online.
- The core loop is `add` to stage, then `commit` to save a snapshot.
- The **staging area** is a cart where you choose what goes into a commit.
- Use **.gitignore** to keep junk and secrets out of Git.
- **Branches** let you work safely without touching `main`.
- Use `push` to upload and `pull` to download changes.
- Write clear, specific commit messages.

---

[← Back to npm and Package Management](./01-npm-and-packages.md) | [Back to Course Index](../README.md) | [Next: Browser DevTools and Debugging →](./03-devtools-and-debugging.md)
