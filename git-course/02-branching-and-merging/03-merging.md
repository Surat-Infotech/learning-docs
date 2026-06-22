# Merging Branches

You created a branch, did some great work on it, and now you want that work to become part of your main project. That joining-together step is called **merging**.

Think back to the story analogy: you wrote an alternate ending on a separate sheet, you love it, and now you want to fold it into the real manuscript. Merging is how Git folds one branch's work into another.

In this lesson you will learn the two kinds of merges Git performs, when each happens, and the standard everyday flow for merging a finished feature into `main`.

## What You'll Learn

- What `git merge` does and the direction it works in
- What a **fast-forward** merge is, with a diagram
- What a **three-way** merge is, with a diagram
- What a **merge commit** is and why it has two parents
- The typical flow: switch to `main`, then merge your feature
- How to clean up by deleting the merged branch afterward

## The Direction of a Merge

This trips up almost everyone at first, so let's be crystal clear:

> You merge *the other branch* **into** *the branch you are currently on*.

So to bring a `feature` branch's work into `main`, you first **switch to `main`**, and then run `git merge feature`. You are standing on the receiving branch and pulling the other one in.

```bash
# Step 1: stand on the branch that should RECEIVE the work
git switch main

# Step 2: merge the feature branch INTO main
git merge feature
```

## Fast-Forward Merge

The simplest case. Imagine `main` has not changed at all since you branched off. Your `feature` branch is simply `main` plus a few extra commits ahead.

Before merging:

```text
A ──> B ──> C   <── main
              \
               D ──> E   <── feature
```

Here `main` is still at `C`, and `feature` is just two commits further along the same line. Git can merge by simply **sliding the `main` pointer forward** to where `feature` is. No new commit is needed — the history is already a straight line.

After a fast-forward merge:

```text
A ──> B ──> C ──> D ──> E   <── main, feature
```

This is called a **fast-forward** because Git just fast-forwards the `main` label up to catch up with `feature`. Both pointers now sit on the same commit.

```bash
git switch main
git merge feature
# Output: Updating c3d4e5f..e5f6a7b
#         Fast-forward
#          file.txt | 2 ++
#          1 file changed, 2 insertions(+)
```

The word `Fast-forward` in the output tells you exactly which kind of merge happened.

## Three-Way Merge

Now the more interesting case. Suppose **both** branches moved forward after they split. You committed on `feature`, *and* someone (maybe you) also committed on `main`.

Before merging:

```text
A ──> B ──> C ──> F   <── main
              \
               D ──> E   <── feature
```

Now `main` cannot simply slide forward — the two branches have **diverged**. They share history up to `C`, but then went their separate ways: `main` to `F`, and `feature` to `E`.

To combine them, Git looks at three commits — hence "three-way":

1. The common ancestor (`C`),
2. The tip of `main` (`F`),
3. The tip of `feature` (`E`),

then it builds a brand-new commit that combines both sets of changes. This new commit is called a **merge commit**.

After a three-way merge:

```text
A ──> B ──> C ──> F ──> M   <── main
              \        /
               D ──> E
```

Commit `M` is the merge commit. Notice it has **two parents**: `F` (the previous tip of `main`) and `E` (the tip of `feature`). It ties the two timelines back together.

```bash
git switch main
git merge feature
# Output: Merge made by the 'recursive' strategy.
#          file.txt | 3 +++
#          1 file changed, 3 insertions(+)
```

When Git makes a merge commit, it usually opens your text editor with a default message like `Merge branch 'feature'`. You can keep it as-is and save.

> A merge commit is the only kind of commit with **two parents**. That is what records that two branches came together here.

## Fast-Forward vs Three-Way: Quick Summary

| Situation | What Git does | Result |
|-----------|---------------|--------|
| `main` did **not** move since branching | Slides `main` forward | Fast-forward, no new commit |
| `main` **did** move (both branches changed) | Builds a merge commit | Three-way merge, one new commit with two parents |

You do not choose between these — Git automatically picks the right one based on whether the branches diverged.

## The Typical Everyday Flow

Here is the workflow you will use again and again:

```bash
# 1. Create a branch and do your work
git switch -c add-greeting
echo "Hello!" > greeting.txt
git add greeting.txt
git commit -m "Add greeting"

# 2. When the feature is done, switch back to main
git switch main

# 3. Merge the finished work into main
git merge add-greeting

# 4. Delete the branch now that its work lives in main
git branch -d add-greeting
```

Let's walk through what each step does:

- **Step 1** isolates your work on its own branch.
- **Step 2** moves you onto the receiving branch, `main`.
- **Step 3** folds the feature's commits into `main`.
- **Step 4** cleans up — the branch label is no longer needed because its commits are now part of `main`'s history.

## Cleaning Up After a Merge

Once a branch is merged, the branch *name* has done its job. The commits are not lost when you delete it — they are now reachable from `main`. Deleting the label just keeps your branch list tidy.

```bash
# Safe delete — works because the work is now merged into main
git branch -d add-greeting
# Output: Deleted branch add-greeting (was 9a8b7c6).
```

Because you merged first, the safe `-d` will happily delete it. If Git complains that the branch is "not fully merged," that is a signal you have **not** actually merged it yet — double-check before reaching for the dangerous `-D`.

## Seeing the Merge in History

After merging, you can look at the shape of your history:

```bash
# Show a compact, graphical view of commits
git log --oneline --graph
# Output:
# *   m1m2m3 Merge branch 'feature'
# |\
# | * e5f6a7b Work on feature
# * | f1f2f3f Work on main
# |/
# * c3d4e5f Common starting point
```

That `|\` and `|/` shape is the merge commit tying two lines back together — exactly the diagram from earlier, drawn in text by Git.

## Common Mistakes

- **Merging in the wrong direction.** Remember: switch to the receiving branch first, then `git merge <other-branch>`. Running `git merge main` while on `feature` does the opposite of what most people intend.
- **Forgetting to switch branches before merging.** Always confirm with `git branch --show-current`.
- **Thinking deleting a merged branch loses commits.** The commits stay in `main`'s history; only the label is removed.
- **Expecting a merge commit every time.** If `main` did not move, you get a fast-forward with no new commit instead.
- **Using `-D` to delete a branch Git refused to `-d`.** That refusal usually means the work is *not* merged. Investigate before forcing.

## Exercises

1. Create a branch from `main`, make one commit on it, and merge it back without touching `main` in between. What kind of merge happens? How do you know from the output?
2. This time, make a commit on `main` *and* a commit on a feature branch before merging. What kind of merge happens now, and how many parents does the resulting commit have?
3. Write out, from memory, the four-step everyday flow for creating, merging, and cleaning up a feature branch.
4. After merging a branch, run `git log --oneline --graph`. Describe the shape you see and what the merge commit looks like.
5. Explain why `git branch -d` succeeds on a merged branch but refuses an unmerged one — and what that refusal is trying to protect you from.

## Key Takeaways

- `git merge <branch>` folds another branch's work **into the branch you are currently on**.
- A **fast-forward** merge happens when `main` has not moved; Git just slides the pointer forward, creating no new commit.
- A **three-way** merge happens when both branches diverged; Git creates a **merge commit** that has **two parents**.
- The everyday flow is: branch, work, `git switch main`, `git merge <branch>`, then `git branch -d <branch>`.
- Deleting a merged branch removes only the label — the commits live on in `main`.
- `git log --oneline --graph` lets you see the merge shape in your history.

---

[← Previous: Creating and Switching Branches](02-creating-and-switching-branches.md) | [Back to Course Index](../README.md) | [Next: Merge Conflicts →](04-merge-conflicts.md)
