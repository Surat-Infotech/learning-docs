# Open Pull Request

You are opening a GitHub Pull Request for the current branch. Follow these steps exactly.

## Step 1 — Gather Context

Run these commands in parallel:

- `git branch --show-current` — current branch name
- `git log --oneline main..HEAD` — commits on this branch vs main
- `git diff main...HEAD --name-only` — all files changed vs main

## Step 2 — Determine PR Metadata

From the branch name and commit list, extract:

- **Type** — `Content`, `Fix`, or `Chore` based on the branch prefix (`content/` → `Content`, `fix/` → `Fix`, `chore/` → `Chore`). If the branch has no recognisable prefix, infer the type from the changes.
- **Short description** — derive from the branch name slug and the changed modules (e.g. `content/async-await-module` → `Async JavaScript: Add Promises and Async/Await Lessons`)

## Step 3 — Compose PR Title

Format: `<Type>: <Short Description>`

Examples:

- `Content: Add Promises and Async/Await Lessons to Module 05`
- `Fix: Correct Typos in Fundamentals Module`

Use sentence case for the description. When the change targets a specific course module, name it.

## Step 4 — Compose PR Body

Write a PR body with these sections:

```
## Summary
<2-4 bullet points describing WHAT changed and WHY>

## Modules Affected
<List of course modules touched, e.g. "- 05-asynchronous-javascript">
(Omit this section if the change is repo-wide or root-level only)
```

Keep it concise and factual. Focus on the "why" in the summary.

## Step 5 — Create the PR

Run:

```
gh pr create --base main --title "<title>" --body "<body>"
```

Use a HEREDOC for the body to preserve formatting:

```bash
gh pr create --base main --title "..." --body "$(cat <<'EOF'
## Summary
- ...

## Modules Affected
- ...
EOF
)"
```

## Step 6 — Output the PR URL

Print the PR URL returned by `gh pr create` so the user can open it immediately.

## Rules

- PR title follows the format: `<Type>: <Description>`
- Always target `main` as the base branch
- Never include Claude attribution anywhere in the PR
- If `gh` is not authenticated or the remote is not GitHub, report the error clearly
