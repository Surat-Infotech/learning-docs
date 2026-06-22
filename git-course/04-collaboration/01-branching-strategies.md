# Branching Strategies

Imagine a busy kitchen where five cooks share one cutting board. Without a plan, they bump elbows, grab the same knife, and ruin each other's dishes. A **branching strategy** is the kitchen's set of rules: who works where, when dishes get plated, and how everything comes together for the customer. It keeps a team's code calm, predictable, and safe to ship.

A **branch** is a separate line of work in Git — a parallel copy of your project where you can make changes without disturbing the main version. In this lesson we'll look at the most popular ways teams organize their branches, and how to pick one that fits.

## What You'll Learn

- Why a team needs an agreed branching strategy at all
- The **feature-branch workflow** (the foundation most strategies build on)
- **Trunk-based development** (lots of small changes straight to main)
- **Git Flow** at a high level (main, develop, feature, release, hotfix)
- **GitHub Flow** (the simple, modern default)
- Common **branch naming conventions** like `feature/`, `fix/`, and `chore/`
- How to choose a strategy for a small team vs a large team

## Why Teams Need a Strategy

When you work alone, you can do whatever you like. When several people share one repository, you need shared rules so that:

- Nobody accidentally breaks the version that customers use.
- It's clear where new work should go and how it gets reviewed.
- Releases are predictable instead of chaotic.
- New teammates can read the rules once and know exactly what to do.

The **main branch** (often called `main`, sometimes `master`) is the official, trusted version of the project. Almost every strategy shares one golden rule: **keep `main` working at all times.** Strategies mostly differ in *how* work gets into `main` safely.

## The Feature-Branch Workflow

This is the building block for nearly every strategy. The idea: never commit directly to `main`. Instead, for each new task, create a short-lived branch off `main`, do your work there, then merge it back.

```bash
git switch main             # Start from the trusted version
git pull                    # Make sure it's up to date
git switch -c feature/login # Create and switch to a new branch
# ...do your work, commit a few times...
git push -u origin feature/login  # Share it
# ...open a Pull Request, get review, then merge...
```

```text
main:    A───B───────────────M   (M = merge of your feature)
              \             /
feature:       C───D───E───/
```

Each feature gets its own branch (`C`, `D`, `E` are your commits), and only joins `main` once it's reviewed and working. This keeps `main` clean while you experiment freely.

## Trunk-Based Development

In **trunk-based development**, the "trunk" is just another name for `main`. The philosophy: keep branches *tiny* and merge into `main` many times a day. Branches live for hours, not weeks.

```text
main: ──●──●──●──●──●──●──●──  (frequent, small merges all day)
         \  \  \
          tiny short-lived branches
```

This works well because small changes are easy to review and rarely conflict. Teams that practice it usually rely on strong automated tests and **feature flags** — switches in the code that hide unfinished features from users until they're ready.

- **Pros:** very few merge headaches, fast feedback, always-shippable `main`.
- **Cons:** needs discipline, good tests, and often feature flags to hide half-done work.

## Git Flow (High Level)

**Git Flow** is an older, more formal strategy designed for projects with scheduled releases (like software you download in versions). It uses several long-lived branches:

- **`main`** — only ever holds released, production-ready code.
- **`develop`** — the integration branch where finished features collect before release.
- **`feature/*`** — branches off `develop` for individual features.
- **`release/*`** — branches off `develop` to prepare a release (final testing, version bumps).
- **`hotfix/*`** — branches off `main` to fix urgent production bugs fast.

```text
main:     ●─────────────────●────────●   (releases + hotfixes)
           \               / \      /
release:    \         ●───●   \    /
             \       /         \  /
develop:  ●───●───●─●───────●───●─●
               \         /
feature:        ●───●───●
```

Git Flow is powerful but heavy. The many branches add overhead, so it has fallen out of fashion for web apps that deploy continuously. It still makes sense for software with distinct, versioned releases.

## GitHub Flow (The Simple Default)

**GitHub Flow** strips things down to the essentials, and it's what most modern web teams use:

1. Create a branch off `main` for your work.
2. Commit your changes and push the branch.
3. Open a **Pull Request** (a PR — a request to merge your branch, with built-in review and discussion).
4. Get review and let automated checks run.
5. Merge into `main`.
6. Deploy `main` (often automatically).

```text
main:  A───B─────────────M───(deploy)
            \           /
branch:      C───D───E─/   ← PR opened here, reviewed, merged
```

There's no `develop`, no `release` branch — just `main` plus short feature branches. It's easy to learn and pairs naturally with continuous deployment.

## Branch Naming Conventions

Good branch names tell teammates what a branch is for at a glance. A common pattern is `type/short-description`:

```text
feature/user-login       # a new feature
fix/header-overlap        # a bug fix
chore/update-deps         # maintenance, not user-facing
docs/api-readme           # documentation only
hotfix/payment-crash      # urgent production fix
```

Tips for naming:

- Use lowercase and hyphens, not spaces.
- Keep it short but descriptive.
- Many teams add a ticket number: `feature/PROJ-123-user-login`.

```bash
git switch -c fix/header-overlap   # Clear, conventional branch name
```

## Choosing a Strategy

There is no single "best" strategy — only the one that fits your team. A rough guide:

- **Solo project or small team (1–5 people):** Use **GitHub Flow**. It's simple, fast, and has very little ceremony.
- **Medium team shipping often to the web:** **GitHub Flow** or **trunk-based development**, backed by good automated tests.
- **Large team with versioned releases (e.g. desktop/mobile apps):** Consider **Git Flow** for its clear release and hotfix branches.

When in doubt, start simple. You can always add structure later; it's painful to remove it once people are used to it.

## Common Mistakes

- **Committing straight to `main`.** Even on small teams, branch and use a PR so work gets a second look.
- **Letting feature branches live for weeks.** The longer a branch lives, the more it drifts from `main` and the worse the merge conflicts. Keep them short.
- **Adopting Git Flow for a simple web app.** The extra branches add overhead you probably don't need. Use GitHub Flow unless you truly have versioned releases.
- **Inconsistent branch names.** `vinayfix`, `temp`, `branch2` tell nobody anything. Agree on a convention and stick to it.
- **Mixing many unrelated changes in one branch.** One branch, one focused purpose makes review and merging far easier.

## Exercises

1. In your own words, explain the difference between GitHub Flow and Git Flow. When would you reach for each one?
2. You're on a 3-person team building a website that deploys every day. Which strategy would you recommend, and why?
3. Write five branch names using the `type/description` convention for these tasks: add a search bar, fix a broken link, upgrade a library, write setup docs, and patch a crash in production.
4. Draw a simple diagram (text or paper) of the feature-branch workflow: `main`, one feature branch, and the merge back in.
5. Look up one open-source project on GitHub and read its CONTRIBUTING file or branch list. Which strategy does it seem to use?

## Key Takeaways

- A **branching strategy** is a team's shared rule set for organizing work and keeping `main` always working.
- The **feature-branch workflow** — branch, work, merge via PR — is the foundation of most strategies.
- **Trunk-based development** favors many tiny merges into `main` per day, supported by tests and feature flags.
- **Git Flow** uses many long-lived branches (main, develop, feature, release, hotfix) and suits versioned releases.
- **GitHub Flow** is the simple modern default: branch off `main`, open a PR, merge, deploy.
- Use clear **naming conventions** like `feature/`, `fix/`, and `chore/`.
- Choose simple strategies for small teams; add structure only when the team and release process truly need it.

---

[← Previous: Forking and Contributing](../03-remotes-and-github/04-forking-and-contributing.md) | [Back to Course Index](../README.md) | [Next: Code Review and Pull Request Etiquette →](02-code-review-and-prs.md)
