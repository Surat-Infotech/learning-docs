# Forking and Contributing to Open Source

You found an open source project you love, spotted a typo in its docs, and want
to fix it. But you do not have permission to push to that project, it isn't
yours. A **fork** solves this. A fork is your own personal copy of someone
else's project, living on your GitHub account. You change your copy freely, then
politely offer your fix back to the original. Think of it like photocopying a
shared recipe so you can scribble improvements before suggesting them to the
chef.

This is how the open source world works, and it is surprisingly approachable.

## What You'll Learn

- What a fork is and how it differs from a clone and a branch
- The full fork → clone → branch → push → PR workflow
- How to add an `upstream` remote pointing at the original project
- How to keep your fork in sync with the original
- Etiquette for contributing (read the rules, keep PRs small)
- How to find a beginner-friendly first issue

## What Is a Fork?

A **fork** is a server-side copy of a repository, made under your own GitHub
account. You make it by clicking the **Fork** button on a project's GitHub page.
Because the copy is yours, you have full push access to it, even though you have
none on the original.

```text
   ORIGINAL (upstream)                YOUR FORK
   github.com/cool-team/app    ──▶    github.com/you/app
   (you cannot push here)             (you can push here)
```

## Fork vs Clone vs Branch

These three words sound similar but happen at different levels. Here is the clear
distinction:

- **Fork** — a copy of a repo on the **server** (GitHub), under your account. Use
  it when you want your own version of a project you do not own.
- **Clone** — a copy of a repo on your **local machine**. Use it to download any
  repo (including your fork) so you can work on it.
- **Branch** — a separate line of work **inside** a repo. Use it to isolate a
  feature or fix.

```text
Fork:    cool-team/app  ──▶  you/app          (server to server)
Clone:   you/app        ──▶  laptop:~/app     (server to your machine)
Branch:  main           ──▶  fix-typo         (inside the repo)
```

A simple way to remember: you **fork** once, **clone** once, and make many
**branches**.

## The Fork → Clone → Branch → Push → PR Workflow

This is the standard recipe for contributing to a project you don't own.

### Step 1: Fork on GitHub

Go to the project's page and click **Fork** (top-right). GitHub creates
`github.com/your-username/app`.

### Step 2: Clone your fork

Copy **your fork** (not the original) to your machine:

```bash
git clone https://github.com/your-username/app.git
cd app
```

### Step 3: Create a branch

Never work on `main` directly. Make a focused branch:

```bash
git switch -c fix-readme-typo
```

### Step 4: Make changes, commit, and push

```bash
# Edit files, then:
git add .
git commit -m "Fix typo in README install steps"

# Push the branch to YOUR fork (origin = your fork)
git push -u origin fix-readme-typo
```

### Step 5: Open a Pull Request

On GitHub, open a PR from your fork's branch into the **original** project's
`main`. GitHub is smart about this: it shows the base as the original repo and
the compare as your fork. The maintainers review it just like any other PR.

```text
base:    cool-team/app : main      (the original)
compare: you/app : fix-readme-typo (your fork's branch)
```

## Adding an `upstream` Remote

Right now your clone knows about one remote: `origin`, which is your fork. But the
original project keeps moving forward. To track it, add a second remote,
conventionally named **`upstream`**, pointing at the original.

```bash
# origin already points to your fork; add upstream for the original
git remote add upstream https://github.com/cool-team/app.git

# Confirm you now have two remotes
git remote -v
# origin    https://github.com/your-username/app.git (fetch/push)  ← your fork
# upstream  https://github.com/cool-team/app.git     (fetch/push)  ← the original
```

```text
   upstream  ──▶  cool-team/app   (the original; you pull updates FROM here)
   origin    ──▶  you/app         (your fork; you push your work TO here)
```

## Keeping Your Fork in Sync

While you work, the original project may gain new commits. To avoid your fork
falling behind, pull from `upstream` and push to `origin`:

```bash
# Get the latest from the original project
git switch main
git fetch upstream

# Merge the original's main into your local main
git merge upstream/main

# Push the updated main back up to your fork
git push origin main
```

GitHub also has a **Sync fork** button on your fork's page that does this for the
`main` branch in one click. Either way, do this **before** starting new work so
you branch off fresh code.

## Etiquette for Contributing

Maintainers are usually volunteers. A little courtesy goes a long way.

- **Read `CONTRIBUTING.md` first.** Many projects include this file with rules on
  style, tests, and how they want PRs submitted. Follow it.
- **Read the `README` and the issue tracker.** Make sure your change is wanted
  before spending hours on it.
- **Keep PRs small and focused.** One fix per PR. A tiny, clear PR gets merged
  quickly; a sprawling one stalls.
- **Match the project's style.** Use the same formatting and naming conventions
  the code already uses.
- **Write a clear description.** Explain what you changed and why. Link the issue
  with `Closes #123` if one exists.
- **Be patient and polite.** Maintainers are busy. Respond kindly to feedback and
  make requested changes promptly.

## Finding Your First Contribution

A great place to start is a **good first issue**. Many projects label easy,
beginner-friendly tasks with exactly that tag.

```text
On a GitHub repo:  Issues tab  →  Labels  →  "good first issue"
```

These are picked specifically so newcomers can succeed. Documentation fixes,
small bug fixes, and adding tests are friendly entry points. Comment on the issue
to say you'd like to work on it, then follow the fork workflow above.

## Common Mistakes

- **Cloning the original instead of your fork.** You will not be able to push.
  Clone `you/app`, not `cool-team/app`, and add the original as `upstream`.
- **Working on `main`.** Always branch. It keeps your fork's `main` clean and easy
  to sync.
- **Letting your fork drift.** If you never sync with `upstream`, your PR may
  conflict with newer code. Sync before you start.
- **Skipping `CONTRIBUTING.md`.** Ignoring the project's rules is the fastest way
  to get a PR closed.
- **Huge, unfocused PRs.** Bundling ten unrelated changes makes review hard. Split
  them up.

## Exercises

1. Find a public project on GitHub and click **Fork**. Confirm the copy now lives
   under your username.
2. Clone your fork, then add the original repo as a remote named `upstream`. Run
   `git remote -v` and confirm you see both `origin` and `upstream`.
3. Create a branch, make a tiny documentation improvement, commit it, and push it
   to your fork.
4. Sync your fork with `upstream` using `git fetch upstream` and
   `git merge upstream/main` (or the **Sync fork** button on GitHub).
5. Browse a project's Issues tab and filter by the **good first issue** label.
   Pick one that looks approachable and read its discussion.

## Key Takeaways

- A **fork** is your own server-side copy of someone else's repo, with full push
  access for you.
- **Fork** is server-to-server, **clone** is server-to-your-machine, **branch** is
  inside a repo.
- The workflow is **fork → clone → branch → push → open PR** against the original.
- Add an **`upstream`** remote pointing at the original to track its updates.
- Keep your fork in sync by fetching from `upstream` and pushing to `origin`.
- Follow project etiquette: read `CONTRIBUTING.md`, keep PRs small, be polite.
- Look for **good first issue** labels to find a friendly first contribution.

---

[← Previous: Pull Requests](03-pull-requests.md) | [Back to Course Index](../README.md) | [Next: Branching Strategies →](../04-collaboration/01-branching-strategies.md)
