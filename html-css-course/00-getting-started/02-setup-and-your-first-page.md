# Setup and Your First Page

Now that you understand what the web is, it's time to roll up your sleeves and build something. In this lesson you'll set up the simple tools you need and create your very first web page. By the end, you'll have a real HTML file open in your browser that you made yourself. That's a big milestone, so let's get to it.

## What You'll Learn

- The two tools every web developer needs
- How to download and set up **VS Code**, a free text editor
- How to create a project folder to keep your work tidy
- How to create your first `index.html` file
- How to write a minimal, working HTML page
- How to open your page in the browser
- The **edit, save, refresh** loop you'll use every day
- Why the file `index.html` is special
- A first peek at the browser's **DevTools**

## The Tools You Need

The wonderful thing about web development is that the tools are free and you probably already have half of them. You only need two things.

### 1. A Text Editor

A **text editor** is a program where you write your code. You could technically use a basic notes app, but a proper code editor makes your life much easier. It adds colors to your code, points out mistakes, and keeps everything organized.

We strongly recommend **VS Code** (short for Visual Studio Code). It's free, it works on Windows, Mac, and Linux, and it's the most popular editor in the world for this kind of work.

To get it:

```text
1. Go to https://code.visualstudio.com
2. Click the big download button (it detects your system automatically).
3. Run the installer and follow the steps, accepting the defaults.
4. Open VS Code when it finishes.
```

### 2. A Modern Web Browser

You also need a **web browser** to view your pages. You almost certainly already have one. Any modern browser works well: Google Chrome, Firefox, Microsoft Edge, or Safari. If you're not sure which to use, Chrome or Firefox are great choices.

That's all. With a text editor and a browser, you have everything you need to build websites.

## Creating a Project Folder

Before writing any code, let's make a home for your work. Keeping all your files in one **folder** (also called a directory) prevents a messy pile of files later.

Think of a project folder like a labeled box. Everything for this course goes inside it, so you always know where to find it.

Create a new folder somewhere easy to find, such as your Desktop or Documents. Name it something clear like:

```text
my-first-website
```

Tip: use lowercase letters and dashes instead of spaces in folder names for web projects. So `my-first-website` is better than `My First Website`. This avoids small problems later.

Now open this folder inside VS Code:

```text
1. Open VS Code.
2. Click "File" in the top menu, then "Open Folder...".
3. Choose your "my-first-website" folder and confirm.
```

VS Code will now show your folder on the left side. It's empty for now, but not for long.

## Creating Your First File

Inside VS Code, with your folder open, create a new file:

```text
1. Hover over your folder name on the left panel.
2. Click the "New File" icon (a small page with a plus).
3. Type the name: index.html
4. Press Enter.
```

You've just created an empty file called **`index.html`**. The `.html` part at the end is the **file extension**. It tells your computer, "This is an HTML file." Just like `.jpg` means an image and `.mp3` means a song, `.html` means a web page.

## Writing a Minimal Web Page

Now let's fill that empty file with a real, working web page. Type (or copy) the following into your `index.html` file:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>My First Page</title>
  </head>
  <body>
    <h1>Hello, World!</h1>
    <p>This is my very first web page. I made it myself.</p>
  </body>
</html>
```

Don't worry about understanding every line yet. We'll cover each piece in detail in the next lessons. For now, here's a quick, friendly tour so it doesn't look like a puzzle:

- **`<!DOCTYPE html>`** tells the browser, "This is a modern HTML page."
- **`<html>`** wraps the whole page. Everything lives inside it.
- **`<head>`** holds hidden information *about* the page, like its title.
- **`<title>`** is the name shown on the browser tab.
- **`<body>`** holds everything you actually *see* on the page.
- **`<h1>`** is a big heading.
- **`<p>`** is a paragraph of text.

Now **save the file**. This is important: your changes don't count until you save. Press `Ctrl + S` (or `Cmd + S` on a Mac). When a file is unsaved, VS Code shows a small dot next to its name. After saving, the dot turns back into an X.

## Opening Your Page in the Browser

Time for the exciting part: seeing your page come to life. There are two easy ways.

### Way 1: Double-Click the File

The simplest method needs no extra tools.

```text
1. Open your "my-first-website" folder in your computer's file explorer.
2. Find index.html.
3. Double-click it.
```

It should open in your default browser. You'll see a big "Hello, World!" heading and your paragraph below it. Congratulations, that's a real web page that you built.

### Way 2: Use the Live Server Extension (Recommended)

VS Code can be extended with small add-ons called **extensions**. A popular one named **Live Server** automatically refreshes your page every time you save, which saves you a lot of clicking.

To set it up:

```text
1. In VS Code, click the Extensions icon on the left (it looks like four squares).
2. Search for "Live Server".
3. Click "Install" on the one by Ritwick Dey.
4. Open your index.html file.
5. Right-click anywhere in the file and choose "Open with Live Server".
```

Your page opens in the browser, and now it updates on its own whenever you save. Magic.

## The Edit, Save, Refresh Loop

This is the rhythm you'll repeat thousands of times as a developer. Learn it now and it becomes second nature.

```text
1. EDIT    — change your code in VS Code.
2. SAVE    — press Ctrl + S (or Cmd + S).
3. REFRESH — reload the browser to see your change.
```

Try it now. Change the heading text in your file:

```html
<h1>Hello, World!</h1>
```

to something personal:

```html
<h1>Hello, I'm learning to code!</h1>
```

Save the file, then refresh your browser (press `F5`, or just save if you're using Live Server). You'll see your new heading appear. You just made your first edit to a website. That feedback loop, change then see the result, is the heart of how you'll learn.

## Why `index.html` Is Special

You might wonder why we named the file `index.html` instead of something like `home.html`. There's a good reason.

When a browser visits a website and isn't told which exact file to open, it automatically looks for one named **`index.html`** first. It's the agreed-upon default, like the front door of a house. Everyone knows to start there.

So `index.html` is, by tradition, the **home page** of any website. Naming your main page `index.html` means visitors land on it automatically. Keep this habit from day one.

## A First Peek at DevTools

Every modern browser has a hidden toolbox for developers, called **DevTools** (Developer Tools). You don't need to master it now, but let's take a quick peek so you know it exists.

With your page open in the browser, press the **`F12`** key. (On a Mac you can also right-click the page and choose "Inspect".)

A panel will open, usually on the side or bottom of the screen. It can look busy, so don't be alarmed. Find the tab labeled **Elements** (or **Inspector** in Firefox). There you'll see your HTML, the very code you wrote, displayed as a tidy tree.

```text
- DevTools lets you inspect any web page's code.
- The "Elements" tab shows the HTML structure.
- You can hover over lines to highlight parts of the page.
```

For now, just look around and then close it by pressing `F12` again. As you grow, DevTools will become one of your most useful friends for finding and fixing problems.

## Common Mistakes

- **Forgetting to save before refreshing.** If your change doesn't appear, you almost certainly forgot to save. Press `Ctrl + S`.
- **Saving with the wrong extension.** Make sure the file ends in `.html`, not `.txt`. If your browser shows the raw code instead of a page, the extension is likely wrong.
- **Using spaces or capital letters in file and folder names.** Stick to lowercase and dashes, like `my-first-website` and `index.html`.
- **Naming the home page something other than `index.html`.** Your main page should be `index.html` so browsers find it by default.
- **Editing the file in the browser instead of the editor.** You write code in VS Code, not in the browser window. The browser only shows the result.

## Exercises

1. Install VS Code and create a project folder named `my-first-website`. Open the folder inside VS Code.
2. Create an `index.html` file and paste in the minimal page from this lesson. Save it, then open it in your browser using either method.
3. Practice the edit, save, refresh loop: change the paragraph text to describe why you want to learn web development. Save and refresh to see your change.
4. Add a second paragraph below the first one by copying the `<p>...</p>` line and changing its text. Save and refresh.
5. Open DevTools with `F12`, find the **Elements** tab, and locate your `<h1>` heading in the code tree. Then close DevTools.

## Key Takeaways

- You only need two free tools: a text editor (**VS Code**) and a **web browser**.
- Keep your work organized inside a clearly named **project folder**.
- A web page is a file ending in **`.html`** that you write in your editor.
- Always **save** your file before refreshing the browser to see changes.
- The **edit, save, refresh** loop is the daily rhythm of web development.
- **`index.html`** is the special default home page that browsers look for first.
- **DevTools** (`F12`) lets you peek at and inspect the code behind any page.

---

[← Back to Course Index](../README.md) | [Next: The HTML Document Structure →](../01-html-fundamentals/01-document-structure.md)
