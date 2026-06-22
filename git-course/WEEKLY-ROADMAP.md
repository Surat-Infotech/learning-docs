# Git Course — 6-Week Weekly Roadmap (10 hours/week)

This roadmap turns the [Git Course](README.md) into a clear weekly plan.

- **Time budget:** 10 hours per week
- **Total length:** 6 weeks (~60 hours)
- **Split:** About **30% reading** and **70% running commands** in a real terminal
  (so ~3 hours reading and ~7 hours practicing each week).

> **How to read this plan.** Each week lists the lessons to study and a small
> hands-on project. Create a throwaway practice repo (`git init practice`) and try
> everything there — you cannot break anything important. Do the lesson exercises,
> then complete the weekly project.

**Weekly rhythm (suggested):** 4–5 short sessions of ~2 hours. Git rewards
frequent, small practice more than long marathons.

---

## Week-at-a-Glance

| Week | Focus                     | Lessons             | Weekly Project                          |
| ---- | ------------------------- | ------------------- | --------------------------------------- |
| 1    | Setup + Fundamentals      | Module 0 + Module 1 | Track a Notes Project from Scratch      |
| 2    | Branching & Merging       | Module 2            | Feature Branches + Conflict Practice    |
| 3    | Remotes & GitHub          | Module 3            | Publish a Project + Open a Pull Request |
| 4    | Collaboration Workflows   | Module 4            | Simulated Team Workflow                 |
| 5    | Advanced Git              | Module 5            | Clean Up a Messy History                |
| 6    | Mastery + Final           | Module 6            | Set Up Your Pro Git Environment         |

---

## Week 1 — Setup and Fundamentals

**Goal:** Get Git installed and master the everyday cycle.

**Lessons (Module 0 + Module 1):**

- [What Is Version Control (and Why Git)?](00-getting-started/01-what-is-version-control.md)
- [Installing and Configuring Git](00-getting-started/02-installing-and-configuring-git.md)
- [The Three Areas](01-fundamentals/01-the-three-areas.md)
- [Your First Repository](01-fundamentals/02-your-first-repository.md)
- [Viewing History](01-fundamentals/03-viewing-history.md)
- [Undoing Changes Safely](01-fundamentals/04-undoing-changes.md)
- [Ignoring Files with `.gitignore`](01-fundamentals/05-gitignore.md)

**Weekly project — Track a Notes Project from Scratch:**

1. `git init` a new `notes` folder.
2. Add several `.md` files, staging and committing them in small, logical commits.
3. Make changes and practice `git diff`, then commit.
4. Add a `.gitignore` that ignores a `drafts/` folder and `.DS_Store`.
5. Practice undoing: discard a change with `restore`, unstage a file, and use
   `revert` to undo a commit.

✅ **End-of-week check:** You can init a repo, stage/commit cleanly, read history,
and undo mistakes without panic.

---

## Week 2 — Branching and Merging

**Goal:** Work on multiple ideas at once and combine them.

**Lessons (Module 2):**

- [Understanding Branches](02-branching-and-merging/01-understanding-branches.md)
- [Creating and Switching Branches](02-branching-and-merging/02-creating-and-switching-branches.md)
- [Merging Branches](02-branching-and-merging/03-merging.md)
- [Resolving Merge Conflicts](02-branching-and-merging/04-merge-conflicts.md)
- [Rebasing](02-branching-and-merging/05-rebasing.md)

**Weekly project — Feature Branches + Conflict Practice:**

1. From your Week 1 repo, create a `feature/about` branch and add content.
2. Merge it back into `main` (a fast-forward).
3. Create two branches that edit the **same line**, merge one, then merge the
   other to deliberately cause a conflict — and resolve it.
4. Repeat the conflict, but this time use `rebase` instead of `merge`.

✅ **End-of-week check:** You can branch, merge, rebase, and resolve a conflict
calmly.

---

## Week 3 — Remotes and GitHub

**Goal:** Get your code online and collaborate.

**Lessons (Module 3):**

- [Remotes and GitHub](03-remotes-and-github/01-remotes-and-github.md)
- [Clone, Push, Pull, and Fetch](03-remotes-and-github/02-clone-push-pull-fetch.md)
- [Pull Requests](03-remotes-and-github/03-pull-requests.md)
- [Forking and Contributing](03-remotes-and-github/04-forking-and-contributing.md)

**Weekly project — Publish a Project + Open a Pull Request:**

1. Create a GitHub repo and `git push -u origin main` your notes project.
2. Add a `README.md` on a branch, push it, and open a **Pull Request** to `main`.
3. Review your own PR, then merge it (try the squash option).
4. **Stretch:** Fork a public "first-good-issue" repo, clone it, and practice the
   fork → branch → push → PR flow (no need to actually submit).

✅ **End-of-week check:** You can push, pull, and collaborate through pull requests.

---

## Week 4 — Collaboration Workflows

**Goal:** Work the way real teams do.

**Lessons (Module 4):**

- [Branching Strategies](04-collaboration/01-branching-strategies.md)
- [Code Review and Pull Request Etiquette](04-collaboration/02-code-review-and-prs.md)
- [Keeping Your Branch Up to Date](04-collaboration/03-keeping-your-branch-updated.md)
- [Resolving Conflicts in a Team](04-collaboration/04-team-conflicts.md)

**Weekly project — Simulated Team Workflow:**

Use **two clones** of the same GitHub repo (in two folders) to pretend you are two
teammates.

1. In clone A, push a change to `main`.
2. In clone B (started before that change), make your own change on a branch.
3. Update clone B's branch from `main` (try `git pull --rebase`).
4. Force-push the rebased branch safely with `--force-with-lease` and open a PR.

✅ **End-of-week check:** You can keep a branch up to date and follow a clean
team workflow.

---

## Week 5 — Advanced Git

**Goal:** Shape history and recover from anything.

**Lessons (Module 5):**

- [Interactive Rebase](05-advanced-git/01-interactive-rebase.md)
- [Cherry-Picking Commits](05-advanced-git/02-cherry-pick.md)
- [Stashing Changes](05-advanced-git/03-stashing.md)
- [Reflog and Recovering Lost Work](05-advanced-git/04-reflog-and-recovery.md)
- [Tags and Releases](05-advanced-git/05-tags-and-releases.md)
- [Finding Bugs with `git bisect`](05-advanced-git/06-bisect.md)

**Weekly project — Clean Up a Messy History:**

1. Make 5–6 sloppy commits ("wip", "fix", "typo") on a branch.
2. Use **interactive rebase** to squash and reword them into 2 clean commits.
3. Use **cherry-pick** to copy one commit onto another branch.
4. Practice **stash**: shelve work, switch branches, and pop it back.
5. Do a "bad" `reset --hard`, then **recover** the lost commit with `git reflog`.
6. Tag a commit as `v1.0.0` and push the tag.

✅ **End-of-week check:** You can rewrite local history cleanly and recover lost
work without fear.

---

## Week 6 — Mastery and Final Setup

**Goal:** Build pro habits and a comfortable environment.

**Lessons (Module 6):**

- [Rewriting History Safely](06-mastery/01-rewriting-history-safely.md)
- [Git Hooks](06-mastery/02-git-hooks.md)
- [Aliases and Configuration Tips](06-mastery/03-aliases-and-config.md)
- [Troubleshooting Common Git Problems](06-mastery/04-troubleshooting.md)
- [Git Best Practices](06-mastery/05-best-practices.md)

**Final project — Set Up Your Pro Git Environment:**

1. Add useful **aliases** (`st`, `co`, `lg`) to your global config.
2. Set sensible defaults (`init.defaultBranch=main`, `pull.rebase`, an editor).
3. Add a simple `pre-commit` **hook** that blocks committing if a test fails.
4. Write yourself a one-page "Git rescue" cheat sheet from the troubleshooting
   lesson.
5. Review the best-practices checklist and apply it to a real project's README and
   commit style.

✅ **End-of-week check:** You have a tuned Git setup and can confidently handle
real-world Git situations on any team.

---

## Readiness Checkpoints

- [ ] **End of Week 1:** Stage, commit, view history, and undo changes confidently.
- [ ] **End of Week 2:** Branch, merge, rebase, and resolve conflicts.
- [ ] **End of Week 3:** Push to GitHub and collaborate via pull requests.
- [ ] **End of Week 5:** Rewrite local history and recover lost work.
- [ ] **End of Week 6:** Apply professional workflows and best practices on any team.

---

[← Back to Course Index](README.md)
