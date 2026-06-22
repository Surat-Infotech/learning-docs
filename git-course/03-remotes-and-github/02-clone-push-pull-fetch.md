# Clone, Push, Pull, and Fetch

Now that your project knows where its remote lives, you need to actually move
code back and forth. Think of the remote as a shared mailbox: **push** drops your
letters in, **pull** brings everyone's letters to your desk, and **clone** copies
the whole mailbox to a brand-new desk. These four commands are how local and
remote stay in sync.

## What You'll Learn

- How to copy a remote project to your machine with `git clone`
- How to upload your commits with `git push`
- What "upstream" means and how `git push -u origin main` sets it
- The difference between `git fetch` (download only) and `git pull` (download + merge)
- What **remote-tracking branches** like `origin/main` are
- How to push a brand-new branch to the remote
- A clear mental model of local-to-remote sync

## `git clone`: Copy a Remote to Your Machine

When a project already exists on GitHub, you do not run `git init`. Instead you
**clone** it. Cloning downloads the entire repository, its full history, and sets
up the `origin` remote for you automatically.

```bash
# Copy a remote repo into a new folder named after the project
git clone https://github.com/your-username/my-project.git
```

```text
Cloning into 'my-project'...
remote: Enumerating objects: 42, done.
Receiving objects: 100% (42/42), done.
```

This creates a folder called `my-project` containing the code. You can also pick
a different folder name:

```bash
# Clone into a folder called "work" instead
git clone https://github.com/your-username/my-project.git work
```

After cloning, check that `origin` was set up for you:

```bash
cd my-project
git remote -v
# origin  https://github.com/your-username/my-project.git (fetch)
# origin  https://github.com/your-username/my-project.git (push)
```

No need to run `git remote add`; clone did it.

## `git push`: Send Your Commits Up

You have committed work locally. Those commits are not on the remote yet. To
upload them, you **push**:

```bash
# Push the commits on your current branch to origin
git push origin main
```

Reading this: "Send my `main` branch's commits to the remote named `origin`."

If others have pushed changes you do not have yet, Git will **reject** your push
and ask you to pull first. That is Git protecting you from overwriting someone
else's work.

### Setting upstream with `git push -u origin main`

Typing `origin main` every time gets tedious. The `-u` flag (short for
`--set-upstream`) links your local branch to a remote branch so Git remembers the
connection:

```bash
# First push: set the upstream link
git push -u origin main
```

After that, you can just type:

```bash
git push   # Git already knows: origin, main
git pull   # same shortcut works for pulling
```

### What "upstream" means

**Upstream** is the remote branch your local branch is connected to. Once set,
your local `main` "tracks" `origin/main`. Git can then tell you things like "your
branch is ahead of origin/main by 2 commits," meaning you have 2 commits to push.

```text
local main  ──tracks──▶  origin/main   (the upstream)
```

## `git fetch` vs `git pull`

This is the distinction that trips up most beginners, so let's be precise.

### `git fetch`: download only

`git fetch` **downloads** new commits from the remote but does **not** change
your working files. It updates your record of what the remote looks like, so you
can inspect changes before merging them in.

```bash
# Download remote changes without touching your files
git fetch origin
```

After fetching, your files are untouched, but Git now knows about new commits on
`origin/main`. You can compare:

```bash
git log main..origin/main   # see what's new on the remote
```

### `git pull`: fetch + merge

`git pull` is two steps in one: it **fetches** and then **merges** the remote
changes into your current branch.

```text
git pull  =  git fetch  +  git merge
```

```bash
# Download remote changes AND merge them into your branch
git pull origin main
```

Use `pull` when you simply want your local branch updated. Use `fetch` first when
you want to look before you leap, for example to avoid a surprise merge conflict.

## Remote-Tracking Branches: `origin/main`

When you fetch or clone, Git creates **remote-tracking branches**. These are
read-only bookmarks that record where the remote's branches were the last time
you talked to it. They are named `origin/<branch>`, like `origin/main`.

```text
main          your local branch (you commit here)
origin/main   a snapshot of the remote's main as of your last fetch
```

You do not edit `origin/main` directly. Think of it as Git's memory: "last I
checked, the remote's `main` looked like this." A `fetch` is what refreshes that
memory.

```bash
git branch -a   # list all branches, including remote-tracking ones
# * main
#   remotes/origin/main
```

## Pushing a New Branch

When you create a new branch locally, the remote does not have it yet. Push it
with `-u` so the link is set:

```bash
# Create and switch to a new branch
git switch -c feature-login

# ...make commits...

# Push the new branch and set its upstream
git push -u origin feature-login
```

Now `feature-login` exists on the remote, and future `git push` / `git pull` on
that branch need no extra arguments.

## The Big Picture: Local ↔ Remote Sync

```text
        ┌────────────────────── YOUR MACHINE ──────────────────────┐
        │                                                           │
        │   Working files  ──add──▶  Staging  ──commit──▶  local    │
        │                                                  main     │
        │                                                   │       │
        └───────────────────────────────────────────────── │ ──────┘
                                                            │
                          push (upload) ───────────────────┤
                          ◀── pull (download + merge) ──────┤
                          ◀── fetch (download only) ────────┤
                                                            │
        ┌──────────────────────── REMOTE (origin) ──────────▼──────┐
        │                         origin/main                       │
        └───────────────────────────────────────────────────────── ┘
```

- **commit** moves work into your local history.
- **push** sends local commits up to the remote.
- **fetch** brings remote commits down (files unchanged).
- **pull** brings them down and merges them into your files.

## Common Mistakes

- **Forgetting to push.** Committing only saves work locally. Until you `push`,
  the remote and your teammates see nothing.
- **Push rejected (non-fast-forward).** This means the remote has commits you do
  not have. Run `git pull` to merge them, then push again.
- **Expecting `fetch` to change your files.** It will not. Fetch only updates
  remote-tracking branches. You still need a `merge` (or use `pull`).
- **Cloning inside an existing repo.** Run `git clone` in a plain folder, not
  inside another repository, or you will nest repos by accident.
- **Pushing a new branch without `-u`.** It works, but you will have to keep
  typing `origin <branch>` until you set the upstream.

## Exercises

1. Clone any public GitHub repo (for example your own `git-practice`) onto your
   machine, then run `git remote -v` to confirm `origin` was set automatically.
2. Make a commit locally and run `git push -u origin main`. Then make a second
   commit and confirm a plain `git push` now works.
3. On the GitHub website, edit a file and commit it there. Back in your terminal,
   run `git fetch` and then `git log main..origin/main` to see the new commit
   without merging it.
4. Now run `git pull` and confirm the change appears in your local files.
5. Create a branch called `experiment`, make a commit, and push it with
   `git push -u origin experiment`. Verify it appears on GitHub.

## Key Takeaways

- `git clone <url>` copies a remote repo and sets up `origin` automatically.
- `git push` uploads your commits; `git push -u origin main` sets the upstream so
  future pushes are just `git push`.
- **Upstream** is the remote branch your local branch tracks.
- `git fetch` **downloads** changes only; `git pull` **fetches and merges**.
- **Remote-tracking branches** like `origin/main` are Git's memory of the remote.
- Push a new branch with `-u` to create it on the remote and link it.

---

[← Previous: Remotes and GitHub](01-remotes-and-github.md) | [Back to Course Index](../README.md) | [Next: Pull Requests →](03-pull-requests.md)
