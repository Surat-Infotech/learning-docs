# Finding Bugs with git bisect

A bug appears. You know it worked fine last month, but somewhere in the last
200 commits, something broke it. Checking each commit one by one would take all
day. There has to be a smarter way to find the exact commit that introduced the
problem.

There is, and it is delightfully clever: `git bisect`. It plays a guessing game
with you. Remember "guess the number between 1 and 100" — where the smart move
is always to guess the middle, halving the range each time? Bisect does exactly
that with your commits. Instead of checking 200 commits, it finds the culprit in
about 8 steps. That is the magic of **binary search**.

Once you have used bisect a couple of times, hunting down regressions stops
feeling like detective drudgery and starts feeling like a game you always win.

## What You'll Learn

- What `git bisect` does and the binary-search idea behind it
- How to start a bisect session
- How to mark commits as `good` or `bad`
- How to test each step Git checks out
- How to end the session with `git bisect reset`
- How to automate the whole hunt with `git bisect run`
- A full worked example

## The Binary Search Idea

Suppose you have 200 commits and somewhere in there a bug crept in. You know:

- Some old commit was **good** (the bug was not there).
- Your latest commit is **bad** (the bug is there).

Instead of testing all 200, bisect checks out the commit in the **middle** and
asks you: is the bug here or not? Your answer eliminates half the commits at
once.

```text
good ......................... bad
         ^ test the middle

If middle is good:   good ........|......... bad   (search right half)
If middle is bad:    good .....|............ bad   (search left half)
```

It repeats, halving the suspect range each time, until only one commit remains —
the exact commit that introduced the bug. Halving 200 commits takes only about
8 questions (because 2 to the 8th is 256).

## Starting a Bisect Session

You begin by telling Git you want to bisect, then marking one known-bad and one
known-good commit:

```bash
git bisect start          # begin the hunt
git bisect bad            # the current commit has the bug
git bisect good a1b2c3d   # this older commit was fine
```

The moment you give it a good and a bad commit, Git automatically checks out a
commit roughly halfway between them and waits for your verdict.

## Testing Each Step

Now Git has dropped you onto a middle commit. Your job is to **test** whether
the bug is present. Run your app, run a test, click the button — whatever
reveals the bug. Then tell Git what you found:

```bash
git bisect good           # this commit does NOT have the bug
# or
git bisect bad            # this commit DOES have the bug
```

After each answer, Git jumps to the next middle commit and waits again. You
keep testing and answering. Git also prints how many revisions are left and
roughly how many steps remain, so you can see the range shrinking.

Eventually Git announces the result:

```text
e4f5g6h is the first bad commit
commit e4f5g6h
Author: Dev <dev@example.com>
    Refactor payment rounding
```

That is your culprit — the commit where the bug first appeared.

## Ending the Session

When you are done, you **must** end the bisect to return to your normal branch.
Bisect leaves you on a detached commit, so this step matters:

```bash
git bisect reset          # go back to where you started before bisecting
```

This restores `HEAD` to your original branch and tip. Forgetting this leaves
you stranded on an old commit, which is confusing but harmless — just run
`git bisect reset` whenever you remember.

## A Full Worked Example

Let's hunt a bug where a function now returns the wrong total.

```bash
# Start the session
git bisect start

# Mark current state as broken
git bisect bad

# Mark a release from last month that we know worked
git bisect good v1.2.0
```

Git checks out a middle commit:

```text
Bisecting: 95 revisions left to test after this (roughly 7 steps)
[d4e5f6g] Add discount support
```

You run your test. The total is correct here, so:

```bash
git bisect good
```

Git jumps again:

```text
Bisecting: 47 revisions left to test after this (roughly 6 steps)
[h7i8j9k] Update tax calculation
```

You test — the bug is present:

```bash
git bisect bad
```

You repeat a few more times. Finally:

```text
e4f5g6h is the first bad commit
    Refactor payment rounding
```

Found it. Now clean up:

```bash
git bisect reset
```

You are back on your branch, with the exact commit that caused the bug
identified — ready to inspect or fix it.

## Automating with git bisect run

Testing by hand is fine, but if you can write a **script or command** that
exits with `0` when the code is good and a non-zero code when it is bad, Git can
do the entire hunt by itself.

```bash
git bisect start
git bisect bad
git bisect good v1.2.0

# Let Git test every step automatically
git bisect run ./test-the-bug.sh
```

Git checks out each middle commit, runs your script, reads the exit code to
decide good or bad, and moves on — no input from you. It prints the first bad
commit when finished. This is incredibly powerful for large histories.

A common shortcut is to point it at your test suite directly:

```bash
git bisect run npm test          # use the project's tests as the judge
```

Your test command should **fail** (non-zero exit) on the buggy commits and
**pass** (exit 0) on the good ones. By convention, an exit code of `125` tells
bisect to skip a commit it cannot test (for example, one that will not build).

## Common Mistakes

- **Mixing up good and bad.** "Good" means the bug is absent; "bad" means it is
  present. Reversing them sends bisect the wrong way.
- **Forgetting `git bisect reset`.** This leaves you stranded on an old detached
  commit. Always reset when finished.
- **Choosing a "good" commit that is actually broken.** Pick a known-good
  starting point you are confident about, or the search will be wrong.
- **A flaky test in `bisect run`.** If your test sometimes passes and sometimes
  fails for the same code, automation gives misleading results. Make the test
  reliable first.
- **Not handling unbuildable commits.** Use exit code `125` (or `git bisect
  skip`) for commits your script cannot evaluate.

## Exercises

1. In a repo with several commits, introduce a bug a few commits back, then use
   `git bisect` by hand to find which commit introduced it. End with
   `git bisect reset`.
2. Explain, with rough math, why bisecting 1000 commits takes only about 10
   steps.
3. Write a tiny shell script that exits `0` when a value is correct and `1` when
   it is wrong, then use `git bisect run` with it to find a bug automatically.
4. Start a bisect, mark a good and bad commit, then immediately run
   `git bisect reset` and confirm you are back on your original branch.
5. In your own words, explain the difference between marking a commit `good` and
   `bad`, and what happens after each.

## Key Takeaways

- `git bisect` uses binary search to find the commit that introduced a bug in
  about log-base-2 steps instead of checking every commit.
- Start with `git bisect start`, then mark `bad` (bug present) and `good` (bug
  absent) commits.
- Test each commit Git checks out and answer `good` or `bad` to halve the range.
- Always finish with `git bisect reset` to return to your branch.
- Automate the entire hunt with `git bisect run <command>`, where exit code 0
  means good, non-zero means bad, and 125 means skip.

---

[← Previous: Tags and Releases](05-tags-and-releases.md) | [Back to Course Index](../README.md) | [Next: Rewriting History Safely →](../06-mastery/01-rewriting-history-safely.md)
