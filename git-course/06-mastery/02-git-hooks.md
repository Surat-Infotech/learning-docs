# Git Hooks

Imagine your home had a doorbell that automatically wiped your shoes before you
stepped inside — every single time, without you remembering to do it. **Git
hooks** are exactly that for your repository: little scripts that Git runs
automatically at certain moments, like just before a commit or just before a
push. They let you catch mistakes early, enforce rules, and automate boring
checks so a human never has to remember them.

## What You'll Learn

- What a Git hook is: a script Git runs automatically on an event
- Where hooks live: the `.git/hooks` folder
- The most useful hooks: `pre-commit`, `commit-msg`, and `pre-push`
- How to write a simple `pre-commit` hook that runs a linter or tests
- Why hooks are **local by default** and do not travel with the repo
- How tools like **Husky** let a team share hooks in a JavaScript project

## What Is a Hook?

A **hook** is just a script (a small program) sitting in a special folder. Git
looks for these scripts at specific points in its workflow and, if it finds one,
runs it. The hook can either let the action continue or **stop it** by exiting
with an error.

Think of a hook as a checkpoint guard. When you try to commit, the guard runs
your checks. If everything passes, you walk through. If something fails, the guard
blocks you until you fix it.

```text
You run: git commit
   │
   ▼
Git runs the pre-commit hook  ──► passes ──► commit is created
                              └─► fails  ──► commit is cancelled
```

## Where Hooks Live: `.git/hooks`

Every repository has a hidden folder for hooks. Look inside it:

```bash
# From the root of any Git repo:
ls .git/hooks
```

```text
applypatch-msg.sample     pre-commit.sample      pre-push.sample
commit-msg.sample         pre-rebase.sample      prepare-commit-msg.sample
post-update.sample        pre-receive.sample     update.sample
```

Notice every file ends in `.sample`. Git ships these as **examples that are
turned off**. A hook only runs if its file has the *exact* event name (no
`.sample`) and is **executable**.

```bash
# Activate the example pre-commit hook by removing .sample:
mv .git/hooks/pre-commit.sample .git/hooks/pre-commit

# Make sure it is executable:
chmod +x .git/hooks/pre-commit
```

## The Most Useful Hooks

There are many hooks, but three cover almost everything a beginner needs.

### `pre-commit` — runs before a commit is recorded

This is the most popular hook. Use it to check your code *before* it becomes part
of history: run a linter, run tests, block forbidden files. If the script exits
with a non-zero code, the commit is cancelled.

### `commit-msg` — checks the commit message

This hook receives the path to your message and can reject it if it does not
follow your rules — for example, requiring messages to start with a ticket number
or follow a certain format.

### `pre-push` — runs before code is sent to the remote

Use this for slower checks you do not want on every commit but do want before
sharing — like running the full test suite before `git push`.

```text
pre-commit  → cheap, fast checks (lint, format) on every commit
commit-msg  → validate the wording of the message
pre-push    → heavier checks (full tests) before sharing
```

## A Simple `pre-commit` Example

Let's build a `pre-commit` hook that runs a linter and the tests. Create the file
`.git/hooks/pre-commit`:

```bash
#!/bin/sh
# pre-commit hook: stop the commit if checks fail.

echo "Running linter..."
npm run lint
if [ $? -ne 0 ]; then
  echo "Lint failed. Commit aborted."
  exit 1            # non-zero exit cancels the commit
fi

echo "Running tests..."
npm test
if [ $? -ne 0 ]; then
  echo "Tests failed. Commit aborted."
  exit 1
fi

echo "All checks passed."
exit 0              # zero exit allows the commit
```

The two important ideas:

- The first line `#!/bin/sh` (the **shebang**) tells the system this is a shell
  script.
- `exit 1` **blocks** the commit; `exit 0` **allows** it. `$?` holds the exit code
  of the previous command (0 means success).

Make it executable and try a commit:

```bash
chmod +x .git/hooks/pre-commit
git commit -m "Add feature"     # the hook runs first
```

If your linter complains, the commit will not happen until you fix the issues.

### Skipping a hook (occasionally)

Sometimes you genuinely need to bypass a hook — for example, committing a
work-in-progress on a private branch. Use `--no-verify`:

```bash
# Skip pre-commit and commit-msg hooks for this one commit:
git commit -m "WIP" --no-verify
```

Use this sparingly. Hooks exist to protect you.

## Hooks Are Local by Default

Here is a surprise that trips people up: **hooks are not committed and do not get
cloned.** The `.git/hooks` folder lives inside `.git`, which Git never tracks. So
if you write a great `pre-commit` hook, your teammate who clones the repo gets
**none of it**.

```text
Your machine:        repo + your custom hooks
Teammate clones:     repo only — no hooks!
```

This is intentional (you should not be able to make someone's machine run a
script just by them cloning a repo), but it means sharing hooks across a team
needs a tool.

## Sharing Hooks With Husky (JavaScript Projects)

In a JavaScript/Node project, the popular solution is **Husky**. Husky stores hook
scripts in a normal folder that *is* committed (so everyone gets them), and points
Git at that folder automatically when people install dependencies.

```bash
# Install Husky in a Node project:
npm install --save-dev husky
npx husky init
```

That creates a `.husky/` folder (which you commit). You add commands to its hook
files:

```text
# .husky/pre-commit  (this file is committed to the repo)
npm run lint
npm test
```

Now every teammate who runs `npm install` automatically gets the same hooks. Tools
like **lint-staged** are often paired with Husky to lint only the files you
actually changed, keeping the `pre-commit` step fast.

## Common Mistakes

- **Leaving the `.sample` suffix.** A file named `pre-commit.sample` never runs.
  It must be named exactly `pre-commit`.
- **Forgetting `chmod +x`.** A hook that is not executable is silently ignored.
- **Expecting hooks to be shared.** They live in `.git/hooks` and are not cloned.
  Use Husky (or `core.hooksPath`) to share them.
- **Making `pre-commit` too slow.** If every commit takes 30 seconds, people will
  start using `--no-verify`. Keep it fast; push heavy checks to `pre-push`.
- **Forgetting the shebang.** Without `#!/bin/sh` (or similar), the system may not
  know how to run your script.

## Exercises

1. In a practice repo, list the contents of `.git/hooks` and read one of the
   `.sample` files to see how a hook is structured.
2. Create a `pre-commit` hook that prints `"Hook is running"` and allows the
   commit (`exit 0`). Make a commit and confirm you see the message.
3. Modify that hook to `exit 1` and try to commit. Confirm the commit is blocked.
   Then commit again with `--no-verify` to bypass it.
4. Write a `pre-commit` hook that blocks the commit if any staged file contains the
   word `TODO` (hint: `git diff --cached` shows staged changes).
5. In a Node project, set up Husky with `npx husky init` and add `npm test` to the
   `pre-commit` hook. Explain why this version *is* shared with teammates.

## Key Takeaways

- A **hook** is a script Git runs automatically at a specific moment, and it can
  **block** the action by exiting non-zero.
- Hooks live in `.git/hooks`; remove the `.sample` suffix and run `chmod +x` to
  activate one.
- `pre-commit` runs before a commit, `commit-msg` validates the message, and
  `pre-push` runs before sharing.
- `exit 0` allows the action; `exit 1` cancels it. `--no-verify` skips hooks for
  one command.
- Hooks are **local and not cloned**, so use **Husky** (in JS projects) to share
  them across a team.

---

[← Previous: Rewriting History Safely](01-rewriting-history-safely.md) | [Back to Course Index](../README.md) | [Next: Aliases and Configuration Tips →](03-aliases-and-config.md)
