# Setup and Running Your First Code

It is time to get your tools ready and run real JavaScript! 🛠️ In this lesson you
will set up a few free programs and then write your very first line of code. Do not
worry — every step is small and clear. By the end, you will have said "Hello, world!"
to your computer.

## What You'll Learn

- How to install **VS Code** (a place to write code)
- How to install a modern **browser**
- How to install **Node.js** and check it works
- Three ways to run JavaScript
- How to write your first `console.log("Hello, world!")`
- What `console.log` actually does

## Step 1: Install VS Code

**VS Code** (full name: Visual Studio Code) is a free program for *writing* code. A
program like this is called a **code editor** — think of it as a smart notepad made
just for code. It colors your code and helps you spot mistakes.

How to install it:

1. Go to **https://code.visualstudio.com**.
2. Click the big download button for your computer (Windows, Mac, or Linux).
3. Open the downloaded file and follow the simple steps.

That's it! You now have a place to write code.

## Step 2: Install a Modern Browser

A **browser** is the program you use to visit websites. You probably already have
one. For this course, use a modern, up-to-date browser like:

- **Google Chrome** (https://www.google.com/chrome)
- **Firefox** (https://www.mozilla.org/firefox)
- **Microsoft Edge** (comes with Windows)

Any of these will work great. If you already have one installed, you are done with
this step.

## Step 3: Install Node.js

**Node.js** lets you run JavaScript *outside* the browser — right on your computer.
You will need it later in the course, so let's set it up now.

How to install it:

1. Go to **https://nodejs.org**.
2. Download the version marked **LTS** (it means "Long Term Support" — the stable,
   reliable one).
3. Open the file and follow the steps. Click "Next" through the screens.

### Check That Node.js Works

After installing, let's make sure it worked. Open your **terminal**.

> A **terminal** is a window where you type commands instead of clicking buttons.
> On Windows it is "Command Prompt" or "PowerShell". On Mac it is "Terminal". VS
> Code also has one built in (open it from the menu: **Terminal → New Terminal**).

In the terminal, type this command and press Enter:

```bash
node --version
```

If Node.js is installed, you will see a version number, like:

```bash
# => v20.11.0
```

Seeing a version number means it works! 🎉 The exact numbers may be different, and
that is perfectly fine.

## The Three Ways to Run JavaScript

There are three common ways to run JavaScript. Let's try each one.

### Way 1: The Browser Console

Every browser has a hidden tool for developers called the **console**. The
**console** is a place where you can type JavaScript and see the result right away.
It is the fastest way to try a small idea.

How to open it:

1. Open your browser.
2. Press the **F12** key (or right-click the page and choose **Inspect**).
3. Click the tab named **Console**.

Now type this and press Enter:

```js
console.log("Hello, world!");
// => Hello, world!
```

You should see the message appear. You just ran JavaScript!

### Way 2: A `<script>` Tag in an HTML File

A web page is written in **HTML** (a language for page structure). You can add
JavaScript to a page using a **`<script>` tag**. A **tag** is a small piece of HTML
wrapped in `< >` symbols that tells the browser what to do.

Let's make a tiny web page. In VS Code, create a file called `index.html` and put
this inside:

```html
<!DOCTYPE html>
<html>
  <head>
    <title>My First Page</title>
  </head>
  <body>
    <h1>My First Web Page</h1>

    <!-- Everything between the script tags is JavaScript -->
    <script>
      console.log("Hello from the script tag!");
      // => Hello from the script tag!
    </script>
  </body>
</html>
```

Now open this file in your browser (double-click `index.html`, or right-click it in
VS Code and choose to open it). Then press **F12** and look at the **Console** tab.
You will see your message there.

> Tip: A line like `<!-- this -->` is an **HTML comment**. The browser ignores it.
> It is just a note for humans.

### Way 3: Node with `node file.js`

You can also run JavaScript files directly with Node.js — no browser needed.

In VS Code, create a file called `hello.js` and put this inside:

```js
// hello.js
console.log("Hello, world!");
// => Hello, world!
```

Then open the terminal (**Terminal → New Terminal**) and run:

```bash
node hello.js
```

You will see the message printed right in the terminal:

```bash
# => Hello, world!
```

> Make sure your terminal is in the same folder as the file. The command
> `node hello.js` tells Node: "run the file named hello.js".

## What Does `console.log` Do?

You have now used `console.log` three times, so let's explain it.

`console.log(...)` is a built-in instruction that **prints a message** so you can
see it. Whatever you put inside the brackets gets shown — in the browser console or
in the terminal.

```js
console.log("I am learning JavaScript"); // => I am learning JavaScript
console.log(42);                         // => 42
console.log(2 + 3);                      // => 5
```

It is your most useful tool when you are learning. Whenever you want to check "what
is happening here?", `console.log` shows you the answer.

Analogy: `console.log` is like asking the computer to *say something out loud* so you
can hear it.

## Common Mistakes

- **Typing it as `console.Log` or `Console.log`.** JavaScript cares about capital
  letters. It must be exactly `console.log` — all lowercase.
- **Forgetting the quotes around text.** Text must be wrapped in quotes:
  `console.log("hi")`, not `console.log(hi)`. Without quotes, JavaScript thinks `hi`
  is a *thing it should look up* and will show an error.
- **Forgetting the brackets.** It must be `console.log("hi")`, with `( )`.
- **Running `node hello.js` in the wrong folder.** If you see "Cannot find module",
  your terminal is probably not in the folder where the file lives.
- **Looking for the console output on the page.** With the `<script>` tag, the
  message appears in the browser **Console** (F12), not on the visible page.

## Key Takeaways

- **VS Code** is where you *write* code; a **browser** and **Node.js** are where you
  *run* it.
- Check Node.js is installed with `node --version`.
- There are three ways to run JavaScript: the **browser console**, a **`<script>`
  tag** in an HTML file, and **`node file.js`** in the terminal.
- `console.log(...)` **prints a message** so you can see what your code is doing.
- JavaScript is case-sensitive — type `console.log` exactly.

---

[← Back to Course Index](../README.md) &nbsp;&nbsp;|&nbsp;&nbsp; [Next: Variables: `let`, `const`, `var`](../01-fundamentals/01-variables.md)
