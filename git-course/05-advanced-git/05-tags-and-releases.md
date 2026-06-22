# Tags and Releases

Commits are great, but their names are random-looking hashes like `a1b2c3d`.
Imagine trying to tell a teammate "go to the version we shipped to customers"
and having to read out a string of gibberish. There has to be a better way.

There is: **tags**. A tag is a friendly, permanent label you stick on a
specific commit, like a bookmark that says "this is version 1.0". Think of it
like putting a sticky note on an important page of a book so you can flip
straight back to it. When you release software, you tag the exact commit you
shipped, and forever after you can find it by name.

This lesson shows how tags work and how they power GitHub Releases.

## What You'll Learn

- What a tag is and why it is useful
- The difference between lightweight and annotated tags
- How to tag the current commit or a specific past commit
- How to push tags to a remote (one tag or all of them)
- The basics of semantic versioning (`vMAJOR.MINOR.PATCH`)
- How to list and delete tags
- How GitHub Releases are built on top of tags

## What Is a Tag?

A **tag** is a named pointer to a single commit. Unlike a branch, a tag does
not move — it permanently marks one specific commit. That makes tags perfect
for marking release points like `v1.0.0`, `v2.1.3`, and so on.

To see existing tags in a repo:

```bash
git tag           # list all tags
```

## Lightweight vs Annotated Tags

Git has two kinds of tags, and the difference matters.

A **lightweight tag** is just a simple name pointing at a commit — nothing
more. It is like scribbling a quick label:

```bash
git tag v1.0.0          # create a lightweight tag on the current commit
```

An **annotated tag** stores extra information: who made it, when, and a
message. It is a full object in Git's database, like a labeled, dated, signed
bookmark:

```bash
git tag -a v1.0.0 -m "First public release"
```

Here `-a` means "annotated" and `-m` provides the message, just like with
commits.

> **Recommendation:** use annotated tags for releases. They record who tagged,
> when, and why — valuable information you will be glad to have later.

To see a tag's details, including its message and the commit it points to:

```bash
git show v1.0.0
```

## Tagging a Specific Commit

By default a tag marks your current commit (`HEAD`). But you can tag any commit
in history by giving its hash:

```bash
git log --oneline                          # find the commit you want
git tag -a v0.9.0 a1b2c3d -m "Beta build"  # tag that specific commit
```

This is handy when you forgot to tag a release at the time and want to add the
label afterward.

## Pushing Tags to a Remote

Here is a surprise that catches many beginners: **`git push` does not push
tags.** Tags stay on your machine until you explicitly send them.

To push one specific tag:

```bash
git push origin v1.0.0       # push just this tag
```

To push every tag you have at once:

```bash
git push origin --tags       # push all tags
```

After this, your tags appear on GitHub (or wherever your remote lives) and your
teammates can fetch them.

## Semantic Versioning Basics

When you name release tags, it helps to follow a shared convention so the
numbers actually mean something. The most popular is **semantic versioning**,
written as `vMAJOR.MINOR.PATCH`, for example `v2.4.1`.

Each part has a meaning:

```text
v  2  .  4  .  1
   |     |     |
   |     |     +-- PATCH: backward-compatible bug fixes
   |     +-------- MINOR: new features, nothing breaks
   +-------------- MAJOR: breaking changes
```

The rules in plain English:

- **Bump PATCH** (`2.4.1` to `2.4.2`) for a bug fix that changes no behavior.
- **Bump MINOR** (`2.4.1` to `2.5.0`) when you add a feature that does not break
  existing usage. Reset PATCH to 0.
- **Bump MAJOR** (`2.4.1` to `3.0.0`) when you make a breaking change. Reset
  MINOR and PATCH to 0.

So just by reading `v2.4.1 -> v3.0.0`, a user knows "something might break, read
the notes". That predictability is the whole point.

## Listing and Filtering Tags

To list all tags, or only ones matching a pattern:

```bash
git tag                      # all tags
git tag -l "v1.*"            # only tags starting with v1.
```

## Deleting Tags

Made a tag by mistake? Delete it locally:

```bash
git tag -d v1.0.0            # delete the tag on your machine
```

If you already pushed it, deleting locally is not enough — you must also delete
it on the remote:

```bash
git push origin --delete v1.0.0    # delete the tag on the remote
```

Be cautious about deleting tags others may already rely on, the same way you
are careful rewriting shared history.

## GitHub Releases Are Built on Tags

A **GitHub Release** is a feature that wraps a tag with extra presentation:
release notes, a title, and downloadable files (like a compiled program or a
zip of the source). Under the hood, every release points at a tag.

The typical workflow:

```bash
git tag -a v1.2.0 -m "Add export feature"
git push origin v1.2.0
```

Then on GitHub, you go to the **Releases** page, choose that tag, write
human-friendly release notes (what changed, known issues, thanks), optionally
attach files, and publish. Anyone visiting the page sees a clean, named release
they can download — all anchored to the exact commit your tag marks.

This is how most open-source projects ship versions: tag the commit, push the
tag, write a release.

## Common Mistakes

- **Forgetting to push tags.** Plain `git push` leaves tags behind. Use `git
  push origin <tag>` or `--tags`.
- **Using lightweight tags for releases.** Prefer annotated tags so you capture
  who, when, and why.
- **Inconsistent version numbers.** Pick a scheme (semantic versioning is a
  great default) and stick to it.
- **Deleting a pushed tag only locally.** You must also run `git push origin
  --delete <tag>` to remove it from the remote.
- **Treating tags like branches.** Tags do not move and are not for ongoing
  work; they permanently mark one commit.

## Exercises

1. Create an annotated tag `v1.0.0` on your current commit with a message, then
   inspect it with `git show v1.0.0`.
2. Find an older commit with `git log --oneline` and tag it as `v0.9.0`.
3. Push a single tag to a remote with `git push origin v1.0.0`, then list the
   tags on the remote (or check GitHub).
4. Given a project currently at `v1.4.2`, decide the next version number for:
   a bug fix, a new non-breaking feature, and a breaking change.
5. Create a lightweight tag, then delete it both locally and (if pushed) on the
   remote.

## Key Takeaways

- A tag is a permanent, friendly label pointing at one specific commit.
- Annotated tags (`git tag -a -m`) record who, when, and why; prefer them for
  releases over lightweight tags.
- Tags are not pushed automatically — use `git push origin <tag>` or `--tags`.
- Semantic versioning (`vMAJOR.MINOR.PATCH`) makes version numbers meaningful.
- GitHub Releases wrap a tag with notes and downloads, anchored to that commit.

---

[← Previous: Reflog and Recovery](04-reflog-and-recovery.md) | [Back to Course Index](../README.md) | [Next: Finding Bugs with Bisect →](06-bisect.md)
