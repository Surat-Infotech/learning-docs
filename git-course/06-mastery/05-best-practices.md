# Git Best Practices

You now know the commands. This final lesson is about the **habits** that turn
those commands into a smooth, professional workflow. Think of it like learning to
cook: knowing how to use a knife is one thing; keeping your station clean, tasting
as you go, and plating nicely is what makes you a *good* cook. These practices are
the small, repeatable habits that keep your project history clean, your teammates
happy, and your future self grateful.

## What You'll Learn

- Why you should commit early, often, and in small focused chunks
- How to write good commit messages (imperative mood, the 50/72 rule,
  conventional commits)
- How to choose meaningful branch names
- Why you must never commit secrets, and how to avoid it
- How to review your changes before committing
- How to keep history readable, pull before you push, and protect `main`

## Commit Early, Often, and Small

A good commit is a **single logical change** — one bug fix, one feature, one
refactor. Resist the urge to bundle ten unrelated changes into one giant commit.

```text
Bad:   "Fixed login, updated styles, added tests, renamed files"  (huge, mixed)
Good:  "Fix login redirect bug"                                    (one thing)
       "Add tests for login redirect"                              (next commit)
```

Small commits are easier to review, easier to understand later, and easy to undo
if one of them turns out to be wrong. Committing often also means less work is at
risk if something goes wrong.

## Write Good Commit Messages

A commit message is a note to your future teammates (and future you). Make it
useful.

### Use the imperative mood

Write the subject line as a command — as if completing the sentence *"If applied,
this commit will ___."*

```text
Good:  Add password reset endpoint
Good:  Fix crash when cart is empty
Bad:   Added password reset endpoint     (past tense)
Bad:   Fixes stuff                        (vague)
```

### Follow the 50/72 rule

A widely used convention for message formatting:

- Keep the **subject line under ~50 characters** — short and scannable.
- Leave a **blank line**, then write the body, **wrapping lines at ~72
  characters**.

```text
Add retry logic to payment client

The payment provider occasionally returns a 503. We now retry up to
three times with exponential backoff before surfacing an error to the
user. This reduces failed checkouts during provider hiccups.
```

The subject answers *what*; the body answers *why*.

### Conventional Commits (a popular format)

Many teams prefix the subject with a **type** so messages are consistent and
tooling can read them. This is called **Conventional Commits**.

```text
feat: add dark mode toggle
fix: prevent double-submit on checkout
docs: update README install steps
refactor: extract date helper
test: add cases for empty cart
chore: bump dependencies
```

The format is `type: short description`. You do not have to adopt this, but it is
common on teams and makes history easy to scan.

## Use Meaningful Branch Names

A branch name should tell people what the branch is for at a glance.

```text
Good:  feature/user-profile-page
Good:  fix/login-redirect
Good:  chore/upgrade-node-18
Bad:   my-branch
Bad:   stuff
Bad:   test2
```

A common pattern is `type/short-description`, sometimes with a ticket number like
`fix/JIRA-123-login-redirect`.

## Never Commit Secrets

Passwords, API keys, tokens, and private certificates must **never** go into Git.
Once committed and pushed, assume they are public forever (and recall from the
*Rewriting History* lesson how hard they are to remove).

```bash
# Keep secret files out of Git by listing them in .gitignore:
echo ".env" >> .gitignore
echo "*.key" >> .gitignore
```

Store secrets in environment variables or a secrets manager, and commit only an
example file with placeholders:

```text
# .env.example  (safe to commit — no real values)
DATABASE_URL=
API_KEY=
```

If you ever do leak a secret: rotate it immediately, then scrub history.

## Review Before You Commit

Before staging and committing, look at exactly what you are about to record. This
catches stray debug prints, leftover secrets, and accidental changes.

```bash
git diff             # changes not yet staged
git diff --staged    # changes that ARE staged, about to be committed
git status           # a summary of what is going on
```

Make checking `git diff --staged` a reflex right before every `git commit`. It
takes seconds and prevents embarrassing commits.

## Keep History Readable

A clean history is a gift to everyone who reads it later.

- Squash messy "fix typo", "oops", "wip" commits before sharing (interactive
  rebase).
- Keep each commit a coherent, working step where possible.
- Write descriptive messages so `git log` reads like a story of the project.

```bash
# Tidy up local commits before opening a pull request:
git rebase -i origin/main
```

Remember the golden rule: tidy *local, unshared* commits — never rewrite shared
history without agreement.

## Pull Before You Push

Before you push, make sure you have your teammates' latest work to avoid
conflicts and rejected pushes.

```bash
git pull --rebase    # get the latest, replay your commits on top
# resolve any conflicts, then:
git push
```

This habit keeps history linear and prevents the "your branch and origin have
diverged" surprise.

## Protect `main` With Reviews

On a team, `main` (or `master`/`production`) is sacred — it is what gets deployed.
Protect it:

- Do your work on **branches**, never directly on `main`.
- Open a **pull request** and have a teammate **review** before merging.
- Turn on **branch protection** on the host (GitHub/GitLab) so `main` requires a
  passing build and at least one approval.

```text
feature branch → pull request → review + tests pass → merge into main
```

Code review is not about distrust — it catches bugs early, spreads knowledge, and
makes everyone better.

## Common Mistakes

- **Giant, mixed commits.** One commit that touches everything is hard to review
  and impossible to cleanly undo. Keep them small and focused.
- **Vague messages like "update" or "fix stuff."** Future you will have no idea
  what changed or why. Be specific and use the imperative mood.
- **Committing `.env` or keys.** Add them to `.gitignore` *before* your first
  commit, not after they leak.
- **Pushing without pulling.** You will hit rejected pushes and divergence. Pull
  (preferably `--rebase`) first.
- **Committing straight to `main` on a team.** Use branches and pull requests so
  changes are reviewed before they ship.

## Exercises

1. Take a chunk of work and split it into at least three small, focused commits,
   each with a clear imperative-mood message.
2. Rewrite these bad messages into good ones: "stuff", "fixed it", "changes",
   "asdf". For each, invent a plausible change and write a proper subject line.
3. Practice the review reflex: stage some changes, run `git diff --staged`, and
   describe out loud exactly what you are about to commit before committing.
4. Create a `.gitignore` that excludes `.env` and `*.key`, add a safe
   `.env.example`, and verify with `git status` that the real `.env` is ignored.
5. Pick a small feature idea and write out the full team workflow you would
   follow, from branch name to pull request to merge into `main`.

## Key Takeaways

- Commit **early, often, and small** — one logical change per commit.
- Write messages in the **imperative mood**, follow the **50/72 rule**, and
  consider **Conventional Commits** for consistency.
- Use **meaningful branch names** and **never commit secrets** (use `.gitignore`
  and a secrets manager).
- **Review with `git diff --staged` before every commit**, keep history readable,
  and **pull before you push**.
- Protect `main` with branches, pull requests, and reviews.

## You Did It

Take a moment — you have come a long way. You started not knowing what version
control was, and now you can initialize repositories, branch and merge with
confidence, collaborate through remotes and pull requests, untangle conflicts,
dig through history with `log` and `bisect`, rewrite history safely, automate
checks with hooks, and follow the habits that professional teams rely on every
day.

Git will still surprise you occasionally — everyone, even experts, reaches for
`git status` and the reflog now and then. That is normal. The difference is that
you now have the mental models and the first-aid kit to stay calm and fix it.

You are ready to use Git confidently on any project and any team. Go build
something, commit it well, and share it with the world. Happy committing.

---

[← Previous: Troubleshooting Common Git Problems](04-troubleshooting.md) | [Back to Course Index](../README.md)
