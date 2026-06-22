# Aliases and Configuration Tips

After a few weeks with Git, you will notice you type the same long commands over
and over. **Aliases** are personal shortcuts — like a speed-dial button on a
phone — that turn a long command into a tiny one. And Git's **configuration**
lets you set sensible defaults once so Git behaves the way you want everywhere.
A little setup here pays off every single day you use Git.

## What You'll Learn

- How to create Git aliases with `git config --global alias.<name> <command>`
- A handful of genuinely useful aliases, including a pretty `git lg` log
- How to edit your `~/.gitconfig` file directly
- Handy settings: `pull.rebase`, `init.defaultBranch`, `core.editor`, `color.ui`,
  and `autoSetupRemote`
- The difference between **Git aliases** and **shell aliases**, and when to use
  each

## Creating a Git Alias

A Git alias maps a short name to a longer Git command. You create one with:

```bash
git config --global alias.<short-name> "<the full command>"
```

The `--global` flag means "for all my repositories." Here is the classic example:

```bash
# Now "git st" runs "git status":
git config --global alias.st status
```

Once set, you type `git st` instead of `git status`. Notice you do **not** repeat
the word `git` in the command — `git` is already implied.

## Useful Aliases to Start With

Here are aliases worth setting up on day one.

```bash
# Short command names:
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit

# Unstage a file (the opposite of git add):
git config --global alias.unstage "reset HEAD --"

# See the last commit:
git config --global alias.last "log -1 HEAD"
```

### A pretty log: `git lg`

The plain `git log` is verbose. This alias produces a compact, colorful,
graph-style history that is far easier to read:

```bash
git config --global alias.lg "log --oneline --graph --decorate --all"
```

Now `git lg` shows something like:

```text
* a1b2c3d (HEAD -> main) Add login form
* d4e5f6a Fix typo in README
| * 9z8y7x (feature/signup) Start signup page
|/
* 0a1b2c3 Initial commit
```

The `--graph` draws the branch lines, `--decorate` shows branch names, and
`--all` includes every branch.

## Editing `~/.gitconfig` Directly

Every alias and setting you create is written to a plain text file in your home
folder called `~/.gitconfig`. You can open and edit it by hand, which is handy for
adding several things at once.

```bash
# Open the global config in your editor:
git config --global --edit
```

Inside, aliases and settings look like this:

```text
[user]
    name = Vinay
    email = vinay@example.com

[alias]
    st = status
    co = checkout
    lg = log --oneline --graph --decorate --all

[core]
    editor = code --wait
```

Editing the file directly does the exact same thing as running `git config`
commands — pick whichever you find easier.

## Handy Settings Everyone Should Know

### `init.defaultBranch` — name of the starting branch

Sets what new repositories call their first branch (commonly `main`).

```bash
git config --global init.defaultBranch main
```

### `pull.rebase` — keep history tidy on pull

By default `git pull` creates a merge commit when your branch and the remote have
both moved. Setting `pull.rebase` to `true` instead **replays** your commits on
top, avoiding noisy merge commits.

```bash
git config --global pull.rebase true
```

### `core.editor` — which editor Git opens

Git opens an editor for commit messages and interactive rebases. Point it at the
one you like.

```bash
# Use VS Code and wait until you close the tab:
git config --global core.editor "code --wait"

# Or use nano (simple, beginner-friendly):
git config --global core.editor "nano"
```

### `color.ui` — colorful output

Turns on colors in `git status`, `git diff`, and more so changes are easier to
scan.

```bash
git config --global color.ui auto
```

### `push.autoSetupRemote` — stop the upstream nag

The first time you push a new branch, Git normally complains that there is no
upstream and tells you to type a long `--set-upstream` command. Turning this on
makes Git set it up for you automatically.

```bash
git config --global push.autoSetupRemote true
```

Now `git push` on a brand-new branch just works.

### Checking what you have set

```bash
# List every setting and where it came from:
git config --global --list
```

## Git Aliases vs Shell Aliases

There are **two** kinds of shortcuts, and they live in different places.

### Git aliases (what we have used so far)

A Git alias only shortens the part *after* `git`. You still type the word `git`.

```bash
git config --global alias.st status
git st        # runs git status
```

### Shell aliases (in your terminal config)

A **shell alias** lives in your terminal's config file (like `~/.zshrc` or
`~/.bashrc`) and can shorten the *entire* command, including the word `git`
itself.

```bash
# Add these lines to ~/.zshrc, then restart the terminal:
alias g='git'
alias gst='git status'
alias gco='git checkout'
```

```bash
gst           # runs git status — even shorter!
```

### Which should you use?

```text
Git alias    → lives in ~/.gitconfig, works wherever Git runs, still type "git"
Shell alias  → lives in ~/.zshrc etc., can drop "git" entirely, terminal-specific
```

Use **Git aliases** for Git-specific shortcuts you want everywhere Git works
(including some tools). Use **shell aliases** when you want to drop the word `git`
or combine non-Git commands. Many people use both: Git aliases for `lg`, `unstage`,
and shell aliases for ultra-short `g`, `gst`.

## Common Mistakes

- **Repeating `git` inside a Git alias.** The value of `alias.st` is `status`, not
  `git status`. Git adds the `git` for you.
- **Forgetting quotes on multi-word aliases.** `alias.lg "log --oneline --graph"`
  needs quotes, or the shell splits it into pieces.
- **Setting things without `--global` and wondering why they vanish.** Without
  `--global`, the setting only applies to the current repository.
- **Confusing the two alias types.** If `git st` works but `st` does not, you made
  a Git alias, not a shell alias — that is expected.
- **Editing `~/.gitconfig` with broken syntax.** A missing section header like
  `[alias]` makes Git ignore your entries. Run `git config --list` to verify.

## Exercises

1. Set up the four short aliases (`st`, `co`, `br`, `ci`) and confirm each works
   with a quick command like `git st`.
2. Create the `git lg` pretty-log alias and run it in a repo with a few branches.
   Compare its output to plain `git log`.
3. Open `~/.gitconfig` with `git config --global --edit` and read your `[alias]`
   section. Add an `unstage` alias by hand, then test it.
4. Turn on `push.autoSetupRemote` and `color.ui auto`, then create a new branch and
   push it — confirm you are no longer asked to set an upstream.
5. Add a shell alias `g='git'` to your shell config, reload the terminal, and run
   `g status`. Explain in one sentence how this differs from a Git alias.

## Key Takeaways

- Create Git aliases with `git config --global alias.<name> "<command>"`; do not
  repeat the word `git` in the value.
- A pretty `git lg` log (`log --oneline --graph --decorate --all`) makes history
  far easier to read.
- All global settings live in `~/.gitconfig`, which you can edit by hand with
  `git config --global --edit`.
- Useful defaults: `init.defaultBranch`, `pull.rebase`, `core.editor`,
  `color.ui`, and `push.autoSetupRemote`.
- **Git aliases** shorten the part after `git`; **shell aliases** can replace the
  whole command. Use whichever fits — many people use both.

---

[← Previous: Git Hooks](02-git-hooks.md) | [Back to Course Index](../README.md) | [Next: Troubleshooting Common Problems →](04-troubleshooting.md)
