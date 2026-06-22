# Remotes and GitHub

So far your work has lived on one computer: yours. That is fine until your laptop
breaks, or until a teammate wants to see your code. A **remote** fixes this. A
remote is a copy of your repository (your project's history) that lives somewhere
else, usually on the internet. Think of it as the shared copy in the cloud that
everyone syncs to, like a shared folder that the whole team can read from and
write to.

In this lesson you will set up a free account on **GitHub** (the most popular
home for remotes) and connect your local project to it.

## What You'll Learn

- What a remote is and why you need one
- The difference between your **local** repo and a **remote** repo
- What GitHub, GitLab, and Bitbucket are
- How to create a GitHub account and a new repository
- How to connect your project with `git remote add origin <url>`
- How to list your remotes with `git remote -v`
- Why the main remote is usually named `origin`
- The two ways to authenticate: **HTTPS** and **SSH**, at a high level

## What Is a Remote?

A **remote** is simply a version of your repository hosted somewhere other than
your own machine. Most of the time that "somewhere" is a server on the internet,
but it could also be another computer on your office network.

Your **local** repository is the one on your hard drive, where you run commands
like `git add` and `git commit`. The **remote** repository is the shared copy.
You move work between them on purpose, with commands you will learn in the next
lesson (`push` to send up, `pull` to bring down).

```text
   Your laptop (local)            The cloud (remote)
   ┌──────────────────┐           ┌──────────────────┐
   │  my-project/      │  push →   │  my-project       │
   │  commits A,B,C    │           │  commits A,B,C    │
   │                   │  ← pull   │                   │
   └──────────────────┘           └──────────────────┘
```

### Why bother with a remote?

- **Backup.** If your laptop dies, your work is safe in the cloud.
- **Sharing.** Teammates can see and download your code.
- **Collaboration.** Several people can work on the same project at once.
- **History anywhere.** You can clone your project onto a new machine in seconds.

## GitHub, GitLab, and Bitbucket

These are **hosting services** for Git repositories. They are websites that store
your remotes and add helpful features on top of plain Git, like a web page for
your code, bug tracking, and code review.

- **GitHub** — the most popular, owned by Microsoft. Home to millions of open
  source projects. We will use it in this course.
- **GitLab** — similar features, often used by companies that want to host their
  own private server.
- **Bitbucket** — made by Atlassian, popular with teams that also use Jira.

The good news: Git itself works the same way with all of them. Learn GitHub and
the others will feel familiar.

## Creating a GitHub Account

1. Go to [https://github.com](https://github.com).
2. Click **Sign up** and enter an email, password, and username.
3. Verify your email address (check your inbox for a link or code).

That is it. A free account is enough for everything in this course, including
private repositories.

## Creating a Repository on GitHub

A **repository** (or "repo") on GitHub is the online home for one project.

1. Click the **+** in the top-right corner, then **New repository**.
2. Give it a **name**, for example `my-project`.
3. Choose **Public** (anyone can see it) or **Private** (only you and people you
   invite).
4. Leave "Initialize this repository" **unchecked** if you already have a local
   project. Check it only if you want GitHub to create the repo first.
5. Click **Create repository**.

GitHub now shows you a page with a **URL** and some commands. The URL is the
address of your remote. It looks like one of these:

```text
HTTPS:  https://github.com/your-username/my-project.git
SSH:    git@github.com:your-username/my-project.git
```

## Connecting Your Local Repo to the Remote

Back in your terminal, inside your project folder, you tell Git where the remote
lives with `git remote add`:

```bash
# Add a remote named "origin" pointing at your GitHub repo
git remote add origin https://github.com/your-username/my-project.git
```

Let's break that command down:

- `git remote add` — "register a new remote."
- `origin` — the **name** we are giving this remote (more on this below).
- the URL — **where** the remote lives.

### Checking your remotes with `git remote -v`

To see which remotes your project knows about, run:

```bash
git remote -v
```

```text
origin  https://github.com/your-username/my-project.git (fetch)
origin  https://github.com/your-username/my-project.git (push)
```

The `-v` means "verbose" (show the full URLs). You see two lines: one for
**fetch** (downloading) and one for **push** (uploading). For now they point to
the same place, which is normal.

## What Does `origin` Mean?

`origin` is just a **nickname** for the remote's URL. Instead of typing the long
`https://github.com/...` address every time, you type `origin`.

There is nothing magical about the word `origin`. It is simply the default name
Git suggests for the first remote, the place your project "originated" from. You
could name it `github` or `backup` if you wanted, but everyone uses `origin`, so
stick with it.

```bash
# These mean the same thing if origin points to that URL:
git push origin main
git push https://github.com/your-username/my-project.git main
```

## HTTPS vs SSH: Two Ways to Authenticate

Before Git can push your code to GitHub, it needs to prove **who you are**. This
is called **authentication**. You will pick one of two methods.

### HTTPS

The URL starts with `https://`. When you push, Git asks for proof. Modern GitHub
does **not** accept your account password here. Instead you use a **Personal
Access Token (PAT)**, which is a long secret string you generate on GitHub that
acts like a password just for Git.

```text
HTTPS URL:  https://github.com/your-username/my-project.git
Proof:      a Personal Access Token (created in GitHub settings)
```

- **Easier to start with**, works through most firewalls.
- You create a token under **Settings → Developer settings → Personal access
  tokens** on GitHub, then paste it when Git asks for a password.

### SSH

The URL starts with `git@github.com:`. SSH uses a pair of **keys**: a **private
key** that stays secret on your computer and a **public key** that you upload to
GitHub. They work together to prove your identity, with no password to type each
time.

```text
SSH URL:    git@github.com:your-username/my-project.git
Proof:      an SSH key pair (private key on your machine, public key on GitHub)
```

- A little setup up front, very convenient afterward.
- You generate keys with `ssh-keygen` and add the public one under **Settings →
  SSH and GPG keys** on GitHub.

For this course, **either is fine.** If you just want to get going, use HTTPS
with a token. If you push code often, SSH saves you typing. We will keep the
detailed setup out of scope here, but GitHub's docs walk through both step by
step.

## Common Mistakes

- **Adding the wrong URL.** If you typo the URL, pushes fail. Fix it with
  `git remote set-url origin <correct-url>` instead of removing and re-adding.
- **Naming the remote something odd.** Most tools and tutorials assume the remote
  is called `origin`. Use that name unless you have a strong reason not to.
- **Using your password over HTTPS.** GitHub stopped accepting account passwords
  for Git. If you get an authentication error, you likely need a Personal Access
  Token, not your login password.
- **Confusing local and remote.** Creating a remote does **not** upload your code.
  It only records the address. You still have to `push` (next lesson).
- **Initializing the repo on GitHub when you already have local commits.** This
  can create a conflicting history. Leave the "Initialize" boxes unchecked when
  connecting an existing project.

## Exercises

1. Create a free GitHub account if you do not have one, then create a new
   repository called `git-practice`. Leave it empty (do not initialize it).
2. In an existing local project (or a new one you make with `git init`), run
   `git remote add origin <your-url>` using the URL from your new repo.
3. Run `git remote -v` and confirm you see two lines, both pointing at your repo.
4. Run `git remote remove origin`, then check `git remote -v` (it should be
   empty). Add the remote back again.
5. In your own words, write one sentence explaining the difference between a
   local repo and a remote repo.

## Key Takeaways

- A **remote** is a copy of your repo hosted elsewhere, usually in the cloud.
- Your **local** repo is on your machine; the **remote** is the shared copy.
- **GitHub**, **GitLab**, and **Bitbucket** are services that host remotes.
- `git remote add origin <url>` connects your project to a remote.
- `git remote -v` lists your remotes and their URLs.
- `origin` is just the conventional nickname for your main remote.
- **HTTPS** (with a Personal Access Token) and **SSH** (with key pairs) are the
  two ways to prove who you are when pushing.

---

[← Previous: Rebasing](../02-branching-and-merging/05-rebasing.md) | [Back to Course Index](../README.md) | [Next: Clone, Push, Pull, and Fetch →](02-clone-push-pull-fetch.md)
