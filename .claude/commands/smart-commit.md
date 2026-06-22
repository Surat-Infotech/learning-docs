# Smart Grouped Commit

You are performing a smart grouped commit. Follow these exact steps:

## Step 1 — Ask About Branch

Ask the user: **"Do you want to commit to the current branch (`{current_branch}`) or create a new branch?"**

First, run `git branch --show-current` to get the current branch name and show it in the question.

Present two choices:

1. Commit to the current branch
2. Create a new branch (ask for a branch type)

Wait for the user's answer before proceeding.

## Step 2 — Create Branch (if chosen)

If the user wants a new branch, ask **one follow-up question**:

1. **Branch type** — present three choices: `content` (new/updated course material), `fix` (corrections), `chore` (tooling, config, housekeeping)

Then:

- Suggest a branch name based on the staged/unstaged changes using the format:
  - `<type>/<short-description>` (e.g. `content/async-await-module`, `fix/typo-fundamentals`)
- Confirm the name with the user or use what they provide
- Run: `git checkout -b <branch-name>`

## Step 3 — Analyse Changes

Run `git status --short` and `git diff --name-only HEAD` to see all modified, added, and deleted files.

Group the files logically by area. This repo holds a single `javascript-course/` with numbered modules — group by module. Typical groupings:

- **javascript-course/00-getting-started/** changes together
- **javascript-course/01-fundamentals/** changes together
- **javascript-course/02-functions-and-scope/** changes together
- ...each numbered module as its own group
- Root-level files (README, roadmap, config) as their own group

## Step 4 — Show the Grouping Plan

Present the proposed grouping to the user as a numbered list before doing anything. Ask them to confirm or adjust. Example:

```
Proposed commits:
1. 05-asynchronous-javascript — new promises and async/await lessons
2. 01-fundamentals — fix typos in variables lesson
3. root — update roadmap with week 5 schedule

Proceed with this grouping? (yes / adjust)
```

Wait for confirmation.

## Step 5 — Commit Each Group

For each group **in order**:

1. `git add <files in this group>`
2. Write a concise, imperative commit message that explains WHAT changed and WHY — matching the existing repo style (e.g. `Add 8-week weekly roadmap (30h/week)`, `Fix typos in fundamentals module`)
3. Commit using: `git commit -m "<message>"`
4. **Do NOT add any `Co-Authored-By` line** — commits must appear only under the user's git identity

## Step 6 — Confirm

Run `git log --oneline -6` and show the result so the user can verify all commits landed correctly.

## Step 7 — Push

Push the branch to the remote:

- If the branch has no upstream yet, run: `git push -u origin <branch-name>`
- If it already tracks a remote, run: `git push`

Show the output so the user can see the remote URL and confirm the push succeeded.

## Rules

- Never add `Co-Authored-By: Claude` or any Claude attribution to commits
- Always use the repo's existing git `user.name` and `user.email` — never override them
- Commit messages are short, imperative sentences in sentence case (e.g. `Add data structures module`) — matching the existing history, not Conventional Commits
- One logical change per commit — don't mix unrelated modules in the same commit
