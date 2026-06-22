# Installing and Configuring Git

Before you can use Git, you need to install it on your computer — just like you'd install any other app. Think of this lesson as setting up your toolbox before starting a project: a few minutes now saves a lot of headaches later. We'll install Git, confirm it works, and tell Git who you are so it can label your work correctly.

## What You'll Learn

- A quick, friendly intro to the terminal (command line)
- How to install Git on macOS, Windows, and Linux
- How to check your Git version with `git --version`
- How to do first-time setup: your name and email
- How to set the default branch name to `main`
- How to set your default text editor
- How to view your settings with `git config --list`
- What the global config file (`~/.gitconfig`) is

## A Quick Note on the Terminal

You'll be typing Git commands into a **terminal** (also called the *command line* or *shell*). It's a text-based window where you type commands and press Enter to run them — no buttons or menus.

- On **macOS**: open the **Terminal** app (find it with Spotlight: press `Cmd + Space`, type "Terminal").
- On **Windows**: use **Git Bash** (it comes with Git, more on that below) or **PowerShell**.
- On **Linux**: open your **Terminal** app.

Throughout this course, lines starting with `#` are just comments explaining what a command does — you don't type them. When you see a command like `git --version`, type exactly that and press Enter.

Don't be intimidated! You'll only need a handful of commands, and you'll learn them one at a time.

## Installing Git

### macOS

The easiest way is to just try running Git. Open Terminal and type:

```bash
git --version
```

If Git isn't installed, macOS will offer to install the developer tools for you — click **Install** and wait. Alternatively, if you use [Homebrew](https://brew.sh) (a popular package manager for Mac), you can run:

```bash
brew install git   # Install Git using Homebrew
```

### Windows

1. Go to [git-scm.com](https://git-scm.com) and download the installer for Windows.
2. Run the installer. The default options are fine for beginners — just keep clicking **Next**.
3. This also installs **Git Bash**, a terminal where you can run all the Git commands in this course.

### Linux

Use your distribution's package manager. For example:

```bash
# On Debian/Ubuntu:
sudo apt update
sudo apt install git

# On Fedora:
sudo dnf install git
```

## Checking Your Git Version

Once installed, confirm it's working by checking the version:

```bash
git --version
```

You should see something like:

```text
git version 2.43.0
```

The exact number doesn't matter much — if you see a version, Git is installed and ready. 🎉

## First-Time Setup: Tell Git Who You Are

Every time you save a snapshot (a **commit**), Git stamps it with your name and email — like signing your name on your work. So the first thing to do is tell Git who you are.

We use the `git config` command for this. The `--global` flag means "apply this setting to *all* my projects on this computer."

```bash
# Set your name (use your real name or a nickname)
git config --global user.name "Ada Lovelace"

# Set your email
git config --global user.email "ada@example.com"
```

> **Tip:** If you plan to use GitHub later, use the same email here that you'll use for your GitHub account.

## Set the Default Branch Name to `main`

A **branch** is a line of development in your project (we'll cover branches in detail later). By default, the very first branch Git creates is sometimes named `master`, but the modern standard is `main`.

Set `main` as your default so every new project starts with it:

```bash
git config --global init.defaultBranch main
```

Now whenever you start a new project, your starting branch will be called `main`.

## Set Your Default Editor

Sometimes Git opens a text editor so you can type a message (for example, when writing a commit message). You can choose which editor Git uses.

```bash
# Use VS Code (the --wait makes Git pause until you close the file)
git config --global core.editor "code --wait"

# Or use Nano, a simple beginner-friendly terminal editor
git config --global core.editor "nano"
```

If you don't set this, Git uses a default editor (often **Vim**), which can be confusing for beginners. Setting it to something familiar like VS Code or Nano is a friendly choice.

## Viewing Your Configuration

To see all your current settings, run:

```bash
git config --list
```

You'll see output similar to this:

```text
user.name=Ada Lovelace
user.email=ada@example.com
init.defaultbranch=main
core.editor=code --wait
```

To check just one setting, name it directly:

```bash
git config --global user.name   # Prints: Ada Lovelace
```

## What Is the Global Config File?

When you run `git config --global`, Git doesn't store the setting by magic — it writes it to a plain text file in your home folder called `.gitconfig`.

- On macOS and Linux, it lives at `~/.gitconfig` (the `~` is a shortcut for your home folder).
- On Windows, it's usually at `C:\Users\YourName\.gitconfig`.

You can peek at its contents:

```bash
cat ~/.gitconfig   # Print the global config file to the screen
```

It looks like this inside:

```text
[user]
    name = Ada Lovelace
    email = ada@example.com
[init]
    defaultBranch = main
[core]
    editor = code --wait
```

You *could* edit this file by hand, but using `git config` commands is safer and easier. It's good to know the file exists, though — it's just a simple text file storing your preferences.

## Common Mistakes

- **Forgetting to set your name and email.** Git will warn you (or refuse to commit) until you do. Set them once with `--global` and you're done.
- **Using quotes incorrectly.** Wrap values with spaces in quotes: `"Ada Lovelace"`, not `Ada Lovelace`.
- **Mixing up `user.name` (your identity) with a GitHub username.** The `user.name` setting is just a label on your commits, not a login.
- **Setting `core.editor "code"` without `--wait`.** Without `--wait`, Git won't pause for you to finish typing, which can cause confusing errors.
- **Typing the `#` comment lines.** Those are explanations, not commands.

## Exercises

1. Install Git on your computer, then run `git --version` and write down the version number you see.
2. Set your `user.name` and `user.email` using `git config --global`. Then verify each by running `git config --global user.name` and `git config --global user.email`.
3. Set your default branch name to `main` with `init.defaultBranch`.
4. Choose a default editor (VS Code or Nano) and configure it with `core.editor`.
5. Run `git config --list` and confirm all four of your settings appear. Then open `~/.gitconfig` and match each line to a setting you made.

## Key Takeaways

- You run Git commands in a **terminal** — a text window where you type and press Enter.
- Install Git from [git-scm.com](https://git-scm.com) (Windows), via Terminal/Homebrew (macOS), or your package manager (Linux).
- Confirm it works with `git --version`.
- Do first-time setup: `git config --global user.name` and `user.email` to label your commits.
- Set `init.defaultBranch main` to use the modern default branch name, and `core.editor` for a friendly editor.
- View settings with `git config --list`; they're stored in the plain-text **global config file** at `~/.gitconfig`.

---

[← Previous: What Is Version Control](01-what-is-version-control.md) | [Back to Course Index](../README.md) | [Next: The Three Areas of Git →](../01-fundamentals/01-the-three-areas.md)
