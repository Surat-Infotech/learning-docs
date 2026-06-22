# npm and Package Management

When you build real projects, you don't write everything from scratch. Other people have already written useful code and shared it for free. **npm** is how you get that code into your project. This lesson explains what npm is and how to use it.

## What You'll Learn

- What npm is and why it helps you
- How to check your Node and npm versions
- How to start a project with `npm init` and what `package.json` is
- How to install packages
- The difference between dependencies and devDependencies
- What `node_modules` and `package-lock.json` are
- How to run scripts with `npm run`
- How to use a package in your code
- Semantic versioning basics (`^` and `~`)
- What `npx` is

## What Is npm?

**npm** stands for *Node Package Manager*. Think of it as a giant free library of code. Other developers write small bits of useful code called **packages** (also called "libraries" or "modules") and share them. With npm you can download any of these packages and use them in your own project.

For example, instead of writing your own code to pick a random color, you can install a package that already does it.

npm comes automatically when you install **Node.js**. Node.js is a program that lets you run JavaScript on your computer (not just in the browser).

## Check Your Versions

First, make sure Node and npm are installed. Open your terminal and type these commands.

```bash
# Show the version of Node.js you have installed
node -v

# Show the version of npm you have installed
npm -v
```

If you see version numbers (like `v20.11.0`), you are ready. If you see an error, install Node.js from the official website first. npm comes with it.

## Starting a Project: npm init

Every npm project has a special file called **package.json**. This file is like an ID card for your project. It stores the project name, version, and a list of packages your project uses.

To create it, go to your project folder and run:

```bash
# The -y flag means "yes to all questions" and uses default answers
npm init -y
```

This creates a `package.json` file. Here is a simple example of what it looks like:

```json
{
  "name": "my-project",
  "version": "1.0.0",
  "scripts": {
    "start": "node index.js"
  },
  "dependencies": {},
  "devDependencies": {}
}
```

Let's understand each part:

- **name** — the name of your project.
- **version** — the current version of your project.
- **scripts** — shortcut commands you can run (more on this below).
- **dependencies** — packages your project needs to run.
- **devDependencies** — packages you only need while developing (like testing tools).

## Installing Packages

To add a package, use `npm install` (you can also write `npm i` for short).

```bash
# Install a package called "dayjs" (a tool for working with dates)
npm install dayjs
```

After this, npm does three things:

1. Downloads the package into a folder called `node_modules`.
2. Adds the package to the `dependencies` list in `package.json`.
3. Creates or updates a file called `package-lock.json`.

### Dependencies vs devDependencies

- **dependencies** are packages your project needs to actually run. Example: a date tool your app uses.
- **devDependencies** are packages you only need while building and testing. Example: a testing tool. Your finished app does not need them to run.

To install a package as a devDependency, add `--save-dev` (or `-D`):

```bash
# Install a testing tool, but only for development
npm install --save-dev jest
```

## node_modules and Why You Don't Commit It

When you install packages, they all go into a folder named **node_modules**. This folder can become very large because packages often depend on other packages.

You should **not** save (commit) this folder to Git (a tool for saving versions of your code). Why? Because:

- It is huge and slows everything down.
- Anyone can recreate it by running `npm install`.

The list of packages lives in `package.json`. So when someone copies your project, they just run:

```bash
# Reads package.json and downloads all listed packages
npm install
```

This rebuilds `node_modules` for them. We tell Git to ignore the folder using a file called `.gitignore` (you will learn about this in the Git lesson).

## package-lock.json

The **package-lock.json** file records the *exact* versions of every package that was installed, including the smaller packages your packages depend on.

Why does this matter? It makes sure everyone on your team installs the *same* versions. This avoids the classic problem of "it works on my computer but not on yours." You should commit this file to Git. You don't edit it by hand.

## Running Scripts with npm run

The **scripts** section in `package.json` lets you create shortcut commands. For example:

```json
{
  "scripts": {
    "start": "node index.js",
    "test": "jest"
  }
}
```

Now you can run them like this:

```bash
# Runs "node index.js"
npm run start

# "start" and "test" are special, so you can skip the word "run"
npm start
npm test

# For any other script name you must use "run"
npm run build
```

Scripts save you from typing long commands again and again.

## Using a Package in Your Code

Once a package is installed, you bring it into your file with `import` or `require`.

Modern JavaScript uses `import`:

```js
// Bring in the dayjs package
import dayjs from "dayjs";

// Use it to show today's date
console.log(dayjs().format("YYYY-MM-DD"));
```

Older Node projects use `require`:

```js
// The older way to bring in a package
const dayjs = require("dayjs");

console.log(dayjs().format("YYYY-MM-DD"));
```

To use `import`, your `package.json` usually needs `"type": "module"`. If you see an error, that's often the fix.

## Semantic Versioning Basics

Package versions have three numbers, like `2.4.1`. This is called **semantic versioning**:

- First number (**major**) — big changes that may break your code.
- Second number (**minor**) — new features that don't break anything.
- Third number (**patch**) — small bug fixes.

In `package.json` you often see symbols before the version:

```json
{
  "dependencies": {
    "dayjs": "^1.11.0",
    "lodash": "~4.17.0"
  }
}
```

- **^ (caret)** — allow new minor and patch updates. `^1.11.0` allows `1.12.0` but not `2.0.0`.
- **~ (tilde)** — allow only patch updates. `~4.17.0` allows `4.17.5` but not `4.18.0`.

These symbols help you get safe updates without surprises.

## npx Briefly

**npx** is a tool that comes with npm. It lets you run a package *without installing it first*. This is handy for tools you only need once.

```bash
# Create a new React app without installing the tool permanently
npx create-react-app my-app
```

npx downloads the tool, runs it, and cleans up. Great for one-time jobs.

## Common Mistakes

- **Committing node_modules to Git.** Never do this. Add it to `.gitignore`.
- **Forgetting to run `npm install`** after copying a project. Without it, `node_modules` is empty and nothing works.
- **Deleting package-lock.json.** Keep it. It keeps versions consistent for your team.
- **Putting runtime packages in devDependencies** (or the reverse). Use `--save-dev` only for tools you need during development.
- **Editing files inside node_modules.** Changes there get wiped on the next install.

## Exercises

1. Create a new folder, run `npm init -y`, and open the `package.json` file. Find the `name` and `version` fields.
2. Install the `dayjs` package, then look inside `package.json` to confirm it was added to `dependencies`.
3. Add a script called `"hello"` that runs `node index.js`. Create a small `index.js` that prints a message, then run it with `npm run hello`.
4. Install `jest` as a devDependency using `--save-dev`, and notice where it appears in `package.json`.

## Key Takeaways

- **npm** is a free library of code packages, and it comes with Node.js.
- **package.json** describes your project and lists its packages.
- Use `npm install <name>` to add packages; use `--save-dev` for development-only tools.
- Never commit **node_modules**; anyone can rebuild it with `npm install`.
- **package-lock.json** locks exact versions so everyone gets the same setup.
- Use **scripts** and `npm run` to save typing.
- `^` allows minor updates, `~` allows only patch updates.
- **npx** runs a tool once without installing it.

---

[← Back to Course Index](../README.md) | [Next: Git and GitHub Basics →](./02-git-and-github.md)
