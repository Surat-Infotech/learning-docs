# Ignoring Files with .gitignore

Not every file in your project belongs in Git. Some are generated automatically,
some are huge, and some contain secrets. A `.gitignore` file is like a "do not
pack" list — it tells Git which files to leave out of the box entirely. This keeps
your repository clean, small, and safe.

## What You'll Learn

- Why you should ignore certain files (and which ones)
- How to create a `.gitignore` file
- The pattern syntax: globs, folders, comments, and negation with `!`
- The important rule that `.gitignore` only affects *untracked* files
- How to stop tracking a file you already committed, using `git rm --cached`
- Where to get ready-made `.gitignore` templates

## Why Ignore Files?

Some files just shouldn't live in your project's history:

- **Dependencies** like `node_modules/` — huge folders you can rebuild anytime
  from a config file. No need to store thousands of files.
- **Secrets** like `.env` — files holding passwords and API keys. Committing these
  is a serious security risk.
- **Build output** like `dist/` or `build/` — generated from your source code, so
  storing it is redundant.
- **OS and editor junk** like `.DS_Store` (macOS) or `.vscode/` — noise that's
  specific to one person's computer, not the project.

Ignoring these keeps your repository small, focused on real source code, and free
of secrets.

## Creating a `.gitignore`

A `.gitignore` is just a plain text file named exactly `.gitignore`, placed in
your project's root folder. Each line lists something to ignore.

```bash
touch .gitignore        # create the file
```

Open it in your editor and add patterns, one per line:

```text
# .gitignore
node_modules/
.env
dist/
.DS_Store
```

Save it, then check `git status` — the ignored files vanish from the list:

```bash
git status
```

```text
On branch main
nothing to commit, working tree clean
```

Even though `node_modules/` and `.env` may exist on disk, Git now pretends it
doesn't see them. Do commit the `.gitignore` file itself so your teammates share
the same rules:

```bash
git add .gitignore
git commit -m "Add .gitignore for dependencies and secrets"
```

## Pattern Syntax

`.gitignore` patterns are flexible. Here are the ones you'll use most.

### Plain names and folders

```text
secret.txt        # ignore a file named secret.txt anywhere
logs/             # the trailing slash means "this is a folder"
```

### Globs (wildcards)

The `*` symbol means "any number of any characters." This is called a **glob**.

```text
*.log             # ignore every file ending in .log
*.tmp             # ignore every .tmp file
temp*             # ignore anything starting with "temp"
```

So `*.log` matches `error.log`, `debug.log`, and `app.2026.log` all at once.

### Comments

Lines starting with `#` are comments — Git ignores them. Use them to explain your
rules.

```text
# Build output
dist/

# Environment secrets
.env
```

### Negation with `!`

Sometimes you want to ignore everything *except* one file. The `!` means "do NOT
ignore this." Order matters: ignore broadly first, then make an exception.

```text
*.log             # ignore all log files
!important.log    # ...but keep this one
```

Here Git ignores every `.log` file *except* `important.log`.

## The Key Rule: It Only Affects Untracked Files

This trips up almost every beginner, so read carefully:

**`.gitignore` only works on files Git isn't already tracking.**

If you committed a file *before* adding it to `.gitignore`, Git keeps tracking it.
The ignore rule is too late — Git already knows about the file.

```bash
# Suppose secret.env was committed last week.
echo "secret.env" >> .gitignore
git status
# secret.env STILL shows up as tracked — the rule was added too late
```

## Untracking a File: `git rm --cached`

To make Git forget a file it already tracks (while keeping the file on your disk),
use `git rm --cached`:

```bash
git rm --cached secret.env       # stop tracking, but keep the file locally
git commit -m "Stop tracking secret.env"
```

The `--cached` flag is important: it removes the file from Git's tracking **only**,
not from your computer. Without `--cached`, `git rm` would delete the actual file
too. For a whole folder, add `-r` (recursive):

```bash
git rm -r --cached node_modules/   # untrack an entire folder
git commit -m "Stop tracking node_modules"
```

After this, with the file listed in `.gitignore`, Git will leave it alone going
forward.

## Templates: Don't Start From Scratch

You don't have to memorize what to ignore for every language. Ready-made templates
exist:

- **gitignore.io** (now at toptal.com/developers/gitignore) — pick your language,
  framework, and OS, and it generates a complete `.gitignore` for you.
- **GitHub's gitignore repository** (github.com/github/gitignore) — a big
  collection of templates like `Node.gitignore`, `Python.gitignore`, and more.

For example, a Node.js project template already includes `node_modules/`, `.env`,
log files, and common OS junk — saving you the guesswork.

## Common Mistakes

- **Committing secrets before ignoring them.** Once `.env` is in history, adding it
  to `.gitignore` won't remove it. Untrack it with `git rm --cached` (and rotate
  any leaked keys).
- **Expecting `.gitignore` to untrack files.** It only affects *untracked* files;
  use `git rm --cached` for already-tracked ones.
- **Forgetting the trailing slash for folders.** Use `logs/` to clearly mean a
  folder.
- **Using `git rm` without `--cached`** and accidentally deleting the real file
  from disk.
- **Not committing the `.gitignore` itself,** so teammates don't get the rules.

## Exercises

1. Create a `.gitignore` and add `node_modules/`, `.env`, and `*.log`. Create those
   files/folders and confirm with `git status` that Git no longer lists them.
2. Add a rule that ignores all `.tmp` files *except* `keep.tmp`, using `!`. Test it
   by creating two `.tmp` files and checking `git status`.
3. Commit a file called `config.local`, then add it to `.gitignore` and observe
   that it's still tracked. Untrack it with `git rm --cached` and commit the change.
4. Visit a gitignore template site, generate a `.gitignore` for a language you use,
   and read through the entries — identify three you understand and one you don't.
5. Add comments to your `.gitignore` grouping the rules (for example, "# Secrets",
   "# Build output") so it's easy to read later.

## Key Takeaways

- `.gitignore` is a "do not pack" list: it tells Git which files to leave out.
- Common things to ignore: dependencies (`node_modules/`), secrets (`.env`), build
  output (`dist/`), and OS junk (`.DS_Store`).
- Patterns support globs (`*.log`), folders (`logs/`), comments (`#`), and negation
  (`!keep.this`).
- `.gitignore` only affects **untracked** files; it can't untrack what's already
  committed.
- Use `git rm --cached <file>` to stop tracking a file while keeping it on disk.
- Use templates from gitignore.io or GitHub's gitignore repo instead of starting
  from scratch.

---

[← Previous: Undoing Changes](04-undoing-changes.md) | [Back to Course Index](../README.md) | [Next: Understanding Branches →](../02-branching-and-merging/01-understanding-branches.md)
