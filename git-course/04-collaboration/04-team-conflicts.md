# Resolving Conflicts in a Team

Imagine two people editing the same shared document at the same time. You both rewrite the opening sentence differently, then try to save. The document can't be in two states at once, so it stops and asks: "Which version do you want?" In Git, that moment is a **merge conflict** — and on a team, where many people touch the same files, conflicts happen far more often than when you work alone.

You learned the mechanics of resolving conflicts earlier. This lesson is about conflicts *in a team setting*: how to avoid them, who's responsible for fixing them, and how to handle the scary big ones calmly.

## What You'll Learn

- Why merge conflicts are more common when working in a team
- How communication prevents people from "stepping on each other"
- Resolving conflicts during **pull, rebase, and merge** in a shared context
- Who is usually responsible for resolving a conflict
- Strategies for tackling large, intimidating conflicts
- A brief look at `git rerere` for repeated conflicts
- Habits that prevent conflicts in the first place

## Why Conflicts Are More Common in Teams

A **merge conflict** happens when two branches change the *same lines* of the *same file*, and Git can't decide which version wins. So it stops and asks a human.

Working solo, you rarely touch the same lines twice at once. On a team:

- Several people edit popular shared files (like a config or a main page).
- Branches live for days, so by the time you merge, `main` has moved a lot.
- Two teammates may unknowingly solve the same problem in different ways.

```text
main:    A───B───C   ← teammate edited line 10 here
              \
yours:         X     ← you also edited line 10
                     → Git can't pick one → CONFLICT
```

The more people and the longer the branches, the more often this happens. That's normal — conflicts aren't a sign anyone did something wrong.

## Communicate to Avoid Stepping on Each Other

The cheapest conflict is the one that never happens. A little coordination goes a long way:

- **Say what you're working on.** A quick "I'm refactoring the login page today" warns others to stay clear.
- **Split work along clean lines.** Two people editing the same file at once is asking for trouble; divide by file or module when you can.
- **Merge often.** Don't hoard a branch for two weeks. Small, frequent merges keep everyone close to `main`.
- **Announce big changes early.** If you're about to rename a widely-used file, tell the team first.

A standup, a chat message, or a comment on a shared task board is usually enough.

## Resolving Conflicts in a Shared Context

When a conflict appears, Git marks the clashing section inside the file with **conflict markers**:

```text
<<<<<<< HEAD
const timeout = 30;      // your version
=======
const timeout = 60;      // the incoming version from main
>>>>>>> origin/main
```

- `<<<<<<< HEAD` to `=======` is *your* current version.
- `=======` to `>>>>>>>` is the *incoming* version you're merging in.

You decide what the final code should be (it might be one side, the other, or a blend), then delete the markers. In a team, this matters: **the right answer often isn't "mine" or "theirs" but the combination both people actually need.** When in doubt, ask the teammate whose change conflicts with yours.

```bash
# After editing the file to resolve the conflict:
git add config.js          # mark this file as resolved
git commit                 # finish a merge
# ...or during a rebase:
git rebase --continue      # resume after resolving
```

The flow is the same whether the conflict shows up during a `merge`, a `rebase`, or a `pull` (which is a fetch plus one of those). The only difference is the command you run to finish:

```text
merge   → resolve → git add → git commit
rebase  → resolve → git add → git rebase --continue
pull    → behaves like merge or rebase, depending on your setting
```

## Who Resolves the Conflict?

The general rule: **the person bringing their work in resolves the conflict.** If you're merging `main` into your feature branch (or rebasing onto it), the conflict is yours to fix, because you're the one trying to combine the two.

```text
You update YOUR branch from main  → YOU resolve.
You merge YOUR PR into main        → YOU resolve before merging.
```

This is fair: you understand your own changes best, and you're the one who chose to integrate now. If the conflicting code belongs to a teammate and you're unsure of their intent, **ask them** rather than guessing. Resolving a conflict by deleting code you don't understand is a classic way to introduce bugs.

## Strategies for Big Conflicts

Sometimes a long-lived branch hits a wall of conflicts. Don't panic — break it down:

- **Update more often next time.** Big conflicts usually mean the branch sat too long.
- **Resolve one file at a time.** Use `git status` to see the list, and work through them methodically.
- **Talk to the other author.** For tricky sections, a five-minute chat beats an hour of guessing.
- **Use a visual merge tool.** A side-by-side view makes large conflicts far easier to read:

```bash
git mergetool          # opens a configured visual three-way merge tool
```

- **If it's truly overwhelming, abort and regroup:**

```bash
git merge --abort      # cancel the merge, back to a clean state
git rebase --abort     # cancel the rebase, back to a clean state
```

Then update your branch incrementally, or split your work into smaller pieces, and try again.

## A Brief Word on `git rerere`

If your team rebases long branches, you may resolve the *same* conflict more than once. `git rerere` — short for **reuse recorded resolution** — remembers how you fixed a conflict and replays that fix automatically the next time the identical conflict appears.

```bash
git config --global rerere.enabled true   # turn it on once
```

After that, Git quietly records your resolutions and reapplies them when it sees the same clash again. It's a nice time-saver for repeated rebases, though beginners don't need it day one — just know it exists.

## Preventing Conflicts

The best conflict strategy is prevention:

- **Keep PRs small.** Less code means fewer chances to overlap with someone else.
- **Merge often.** Pull from `main` daily so your branch never drifts far.
- **Don't sit on a branch.** Finish and merge promptly instead of letting work pile up.
- **Coordinate on shared files.** Agree who edits the high-traffic files and when.
- **Format and structure code consistently.** Whole-file reformatting causes huge, avoidable conflicts.

```text
Small PRs + frequent merges + good communication = few, tiny conflicts
```

## Common Mistakes

- **Treating a conflict as a disaster.** It's a routine question from Git, not a failure.
- **Resolving by guessing.** Deleting code you don't understand introduces bugs. Ask the author instead.
- **Letting a branch live for weeks.** The longer it sits, the worse the eventual conflict. Merge often.
- **Forgetting to remove conflict markers.** Leftover `<<<<<<<` lines will break your code. Search for them before committing.
- **Expecting someone else to fix your merge.** The person integrating the work owns the resolution.

## Exercises

1. Explain in your own words why conflicts are more common on a team than when working solo.
2. List three things a team can do to *prevent* conflicts before they ever happen.
3. You're merging `main` into your feature branch and hit a conflict in a file your teammate wrote. Describe, step by step, how you'd handle it — including who you'd talk to.
4. Write the commands to resolve a conflict during a rebase and continue, and the command to abort a rebase if it gets too messy.
5. In one or two sentences, explain what `git rerere` does and when it's helpful.

## Key Takeaways

- **Conflicts** are more common in teams because many people edit shared files and branches drift from `main`.
- **Communication** — saying what you're working on and merging often — prevents most conflicts.
- Resolving works the same during merge, rebase, or pull; only the finishing command differs.
- The person **integrating** the work (updating their branch or merging their PR) resolves the conflict — and asks the author when unsure.
- Tackle big conflicts one file at a time, use a visual merge tool, and `--abort` to regroup if needed.
- `git rerere` can replay past conflict resolutions automatically for repeated rebases.
- Prevention beats cure: small PRs, frequent merges, and coordination keep conflicts few and tiny.

---

[← Previous: Keeping Your Branch Up to Date](03-keeping-your-branch-updated.md) | [Back to Course Index](../README.md) | [Next: Interactive Rebase →](../05-advanced-git/01-interactive-rebase.md)
