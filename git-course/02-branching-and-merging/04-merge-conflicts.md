# Resolving Merge Conflicts

Let's get the most important thing out of the way first: **merge conflicts are completely normal.** They are not errors, they are not a sign you did something wrong, and they are not scary once you have done one. Every developer deals with them regularly.

A conflict simply means Git reached a spot where two branches changed the *same lines*, and Git is honest enough to say, "I do not want to guess which version you want — please tell me." That is it. You are the human making a decision Git refuses to make blindly.

In this lesson you will see exactly why conflicts happen, what they look like, and a calm step-by-step way to resolve them.

## What You'll Learn

- Why merge conflicts happen (two branches edit the same lines)
- What conflict markers look like: `<<<<<<<`, `=======`, `>>>>>>>`
- How to read a real conflicted file
- A step-by-step process to resolve a conflict
- How `git status` guides you during a conflict
- How to finish with `git add` and `git commit`
- How to bail out with `git merge --abort`
- Simple habits that reduce how often conflicts happen

## Why Conflicts Happen

Git is very good at merging changes automatically — *as long as the changes are in different places*. If you edit the top of a file and your teammate edits the bottom, Git combines both with no trouble.

A conflict happens only when **both branches changed the same lines of the same file** (or one branch edited a line the other deleted). Git cannot know which version is correct, so it stops and hands the decision to you.

```text
main:    The cat sat on the mat.
feature: The cat sat on the rug.
                              ^^^ both changed this same word
```

Git looks at this and says: mat or rug? Only you know.

## A Conflict in Action

Let's create one on purpose so it is never a mystery. Start with a file on `main`:

```bash
# On main, create a file and commit
echo "Welcome to our website" > home.txt
git add home.txt
git commit -m "Add welcome line"
```

Make a branch and change the line there:

```bash
git switch -c feature
echo "Welcome to our amazing website" > home.txt
git add home.txt
git commit -m "Make welcome more exciting"
```

Now go back to `main` and change the *same line* differently:

```bash
git switch main
echo "Welcome to our friendly website" > home.txt
git add home.txt
git commit -m "Make welcome friendlier"
```

Both branches changed the same line. Now try to merge:

```bash
git merge feature
# Output:
# Auto-merging home.txt
# CONFLICT (content): Merge conflict in home.txt
# Automatic merge failed; fix conflicts and then commit the result.
```

There it is. Git paused the merge and is waiting for you. Nothing is broken — you are simply mid-merge.

## What the Conflict Markers Look Like

Open `home.txt` in your editor. Git has rewritten it to show **both** versions, wrapped in special markers:

```text
<<<<<<< HEAD
Welcome to our friendly website
=======
Welcome to our amazing website
>>>>>>> feature
```

Read it like this:

- `<<<<<<< HEAD` — everything below this line, up to the `=======`, is the version from **your current branch** (`HEAD`, here `main`).
- `=======` — the divider between the two versions.
- `>>>>>>> feature` — everything above this divider, down from `=======`, is the version from the branch you are **merging in** (`feature`).

So the top chunk is "ours" (`main`) and the bottom chunk is "theirs" (`feature`). Those `<<<`, `===`, and `>>>` lines are **not** part of your file — Git added them as scaffolding. Your job is to remove them and leave the correct content.

## Resolving the Conflict, Step by Step

### Step 1: See where you stand

```bash
git status
# Output:
# On branch main
# You have unmerged paths.
#   (fix conflicts and run "git commit")
#   (use "git merge --abort" to abort the merge)
#
# Unmerged paths:
#   (use "git add <file>..." to mark resolution)
#         both modified:   home.txt
```

`git status` is your map during a conflict. It lists every file marked `both modified` — those are the files you must fix. It even reminds you of the commands you will need.

### Step 2: Edit the file to what you actually want

Open `home.txt` and decide. You can keep one side, the other, or write something new entirely. Delete the `<<<<<<<`, `=======`, and `>>>>>>>` lines and leave only the final text. For example, you might combine both ideas:

```text
Welcome to our friendly and amazing website
```

The file now contains exactly what you want, with **no markers left**.

> Tip: search your file for `<<<<<<<` to make sure you did not miss any markers. Leftover markers will break your code or text.

### Step 3: Mark it resolved with git add

Once the file looks right, tell Git you have handled it by staging it:

```bash
git add home.txt
```

Adding the file is how you say "this conflict is resolved." Check progress again:

```bash
git status
# Output:
# All conflicts fixed but you are still merging.
#   (use "git commit" to conclude merge)
```

### Step 4: Finish the merge with a commit

```bash
git commit
# Git opens an editor with a prepared message like:
#   Merge branch 'feature'
# Save and close to finish.
```

That commit is the merge commit. The merge is now complete, and your conflict is history.

```text
A ──> B ──> C ──> F ──> M   <── main
              \        /
               D ──> E
```

Commit `M` is your finished merge — the one you just resolved by hand.

## Backing Out: git merge --abort

Sometimes you start a merge, see the conflicts, and decide you are not ready to deal with them right now. You can cancel the whole thing and return to exactly how the project was before you ran `git merge`:

```bash
git merge --abort
```

This is a safe undo button for an in-progress merge. It throws away the half-done merge and restores your branch to its pre-merge state. Nothing is lost — you simply try again later.

## Tips to Avoid Conflicts

You can never eliminate conflicts entirely, but a few habits make them rare and small:

- **Merge often.** The longer a branch lives apart from `main`, the more chances for the same lines to drift. Bring changes together frequently.
- **Keep changes focused.** Small, single-purpose branches touch fewer lines and conflict less.
- **Pull updates before starting big work.** Begin from the latest `main` so you are not building on stale code.
- **Communicate with teammates.** If two people must edit the same file, a quick heads-up prevents surprises.
- **Format consistently.** Agreeing on indentation and style means edits do not collide over trivial whitespace.

## Common Mistakes

- **Panicking.** A conflict is a normal pause, not a failure. Read `git status` and work through it calmly.
- **Leaving conflict markers in the file.** Forgetting to delete a `<<<<<<<`, `=======`, or `>>>>>>>` line breaks the file. Search for them before committing.
- **Forgetting to `git add` after editing.** Editing the file is not enough; staging it is how you tell Git the conflict is resolved.
- **Editing files unrelated to the conflict.** Only the files `git status` lists as `both modified` need fixing.
- **Not knowing about `--abort`.** If you feel stuck, `git merge --abort` safely returns you to where you started.

## Exercises

1. Recreate the conflict from this lesson on purpose: make two branches that change the same line of the same file, then merge. Read the markers and identify which chunk came from which branch.
2. During the conflict, run `git status`. Which file is listed, and what label does Git put on it?
3. Resolve the conflict by combining both versions into one new sentence. Confirm there are no markers left, then `git add` and `git commit`.
4. Start a merge that conflicts, then run `git merge --abort`. Confirm with `git status` that you are back to a clean state.
5. List three habits that reduce how often merge conflicts happen, and explain in one sentence why each helps.

## Key Takeaways

- A merge conflict happens when **two branches change the same lines** — Git stops and asks you to decide.
- Conflicts are **normal** and expected, not errors.
- Git marks conflicts with `<<<<<<< HEAD` (your branch), `=======` (divider), and `>>>>>>> branch` (the incoming branch).
- Resolve by editing the file to what you want, **removing all markers**, then `git add` and `git commit`.
- `git status` guides you, listing every file that is `both modified`.
- `git merge --abort` safely cancels an in-progress merge and restores your starting point.
- Merging often and keeping branches small makes conflicts rarer and easier.

---

[← Previous: Merging](03-merging.md) | [Back to Course Index](../README.md) | [Next: Rebasing →](05-rebasing.md)
