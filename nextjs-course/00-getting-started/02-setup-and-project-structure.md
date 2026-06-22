# Setup and Project Structure

Time to build something real! 🚀 In this lesson you'll install the tools you need,
create a brand-new Next.js project with a single command, and take a friendly tour
of the folders it makes for you. Think of it like getting the keys to a new apartment
and walking through every room before you move in. Let's set up your workspace!

## What You'll Learn

- The **prerequisites** you need (Node.js and npm)
- How to create a project with `create-next-app` and what the prompts mean
- How to run the **development server** and view your site
- A tour of the generated folders (`app/`, `public/`, `next.config.js`, `package.json`)
- The special files inside `app/` (`layout.tsx`, `page.tsx`, `globals.css`)
- How to edit your first page and see **hot reload** in action
- The difference between the **dev**, **build**, and **start** scripts

## Prerequisites: Node.js and npm

To run Next.js, you need **Node.js** installed on your computer. **Node.js** lets
JavaScript run outside the browser — Next.js uses it to build and serve your site.

When you install Node.js, you also get **npm** (Node Package Manager). **npm** is a
tool that downloads and manages the code libraries your project depends on.

Check whether you already have them. Open your terminal and run:

```bash
node --version   # should print something like v20.x or higher
npm --version    # should print a version number too
```

If you see version numbers, you're ready. If not, download Node.js from
[nodejs.org](https://nodejs.org) and pick the **LTS** version (LTS means "Long-Term
Support" — the stable, recommended choice).

> 📌 Next.js 14/15 needs a recent Node.js (version 18.18 or newer). The LTS version
> is always a safe bet.

## Creating Your First Project

There's a single command that builds a complete starter project for you. In your
terminal, navigate to the folder where you keep your code, then run:

```bash
npx create-next-app@latest my-first-app
```

Let's unpack that:

- **`npx`** runs a tool without permanently installing it.
- **`create-next-app@latest`** is the official Next.js project generator (the
  `@latest` means "use the newest version").
- **`my-first-app`** is the folder name for your new project. Pick any name.

### Answering the Prompts

The generator asks you a few questions. Here are sensible answers for this course:

```text
Would you like to use TypeScript?              › Yes
Would you like to use ESLint?                  › Yes
Would you like to use Tailwind CSS?            › Yes
Would you like to use `src/` directory?        › No
Would you like to use App Router? (recommended) › Yes
Would you like to customize the import alias?  › No
```

What each one means in plain words:

- **TypeScript** — a version of JavaScript that catches mistakes as you type. We use
  it, so our files end in `.tsx`.
- **ESLint** — a tool that warns you about likely bugs and messy code.
- **Tailwind CSS** — a styling tool that lets you design with small class names.
- **`src/` directory** — choose **No** so the `app/` folder sits at the top level.
- **App Router** — choose **Yes**. This course uses the App Router.
- **Import alias** — keep the default. It's just a shortcut for import paths.

When it finishes, move into your new project folder:

```bash
cd my-first-app
```

## Running the Development Server

Now start the **development server** — a local mini web server that runs your site
on your own computer while you build it.

```bash
npm run dev
```

You'll see output like this:

```text
  ▲ Next.js 15.x
  - Local:        http://localhost:3000
  ✓ Ready in 1.2s
```

Open [http://localhost:3000](http://localhost:3000) in your browser. You should see
the default Next.js welcome page. 🎉

> **`localhost`** means "this very computer." The `3000` is the **port** — think of
> it as a specific door number where your site is listening.

To stop the server later, press `Ctrl + C` in the terminal.

## A Tour of the Folders

Open the project in your code editor. Here's a simplified view of what was created:

```text
my-first-app/
├── app/                  # your pages and layouts live here
│   ├── layout.tsx        # the shared wrapper around every page
│   ├── page.tsx          # the home page (yoursite.com/)
│   └── globals.css       # global styles for the whole app
├── public/               # static files: images, icons, etc.
├── next.config.js        # configuration for Next.js
├── package.json          # project info, scripts, and dependencies
└── node_modules/         # installed libraries (don't edit by hand)
```

Let's walk through the important ones.

### `app/` — Where Your Site Lives

This is the heart of your project. Every page and shared layout goes here. Because we
chose the **App Router**, folders inside `app/` become URLs (you saw this idea in the
previous lesson).

### `public/` — Static Files

Anything in `public/` is served as-is at the root of your site. For example,
`public/logo.png` is available at `yoursite.com/logo.png`. Great for images and icons.

### `next.config.js` — Configuration

This file holds settings for Next.js. You can ignore it for now — the defaults are
fine while you learn.

### `package.json` — The Project's ID Card

This file lists your project's name, its **dependencies** (the libraries it uses),
and its **scripts** (handy commands). Open it and look for the `scripts` section:

```json
// package.json
{
  "scripts": {
    "dev": "next dev",     // start the development server
    "build": "next build", // prepare an optimized version for the internet
    "start": "next start"  // run that optimized version
  }
}
```

You run these with `npm run <name>`, like `npm run dev`.

## The Special Files Inside `app/`

A few filenames have *special meaning* to Next.js. The App Router looks for these
exact names.

### `page.tsx` — A Page

A file named `page.tsx` becomes a visible page. The one at `app/page.tsx` is your
home page. Open it and you'll find a React component:

```tsx
// app/page.tsx
export default function Home() {
  return <h1>Welcome to my first app!</h1>;
}
```

> A **component** is a function that returns the UI to display. The `export default`
> tells Next.js, "this is the page to render."

### `layout.tsx` — The Shared Wrapper

A `layout.tsx` wraps your pages with shared stuff, like a header or footer that
should appear on *every* page. The root layout also defines the basic HTML structure:

```tsx
// app/layout.tsx
import "./globals.css"; // load global styles

export default function RootLayout({
  children, // the page content goes here
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body>{children}</body>
    </html>
  );
}
```

`children` is a placeholder where the current page is slotted in. Whatever page the
user visits appears right there inside the layout.

### `globals.css` — App-Wide Styles

This file holds styles that apply to your entire app. The layout imports it once, and
those styles reach every page.

## Editing Your First Page (and Hot Reload)

Let's change something and watch the magic. Make sure `npm run dev` is still running,
then open `app/page.tsx` and edit the text:

```tsx
// app/page.tsx
export default function Home() {
  return (
    <main>
      <h1>Hello, Next.js! 👋</h1>
      <p>I just edited my first page.</p>
    </main>
  );
}
```

Save the file and look at your browser. It updates *instantly* — you don't need to
refresh! This is called **hot reload**: Next.js watches your files and refreshes the
page automatically when you save.

Analogy: hot reload is like a mirror that updates the moment you change your outfit.
No need to step away and come back.

## The Build and Start Scripts

`npm run dev` is for *building* your site. When you're ready to put it on the
internet, you use two other scripts.

### `npm run build`

This creates an **optimized** version of your app — minified, bundled, and ready for
real users.

```bash
npm run build
```

```text
✓ Compiled successfully
✓ Generating static pages
Route (app)                Size
┌ ○ /                      5.2 kB
```

### `npm run start`

This runs the optimized version you just built, the way real users would experience
it:

```bash
npm run start
```

> 💡 Use **`dev`** while you're coding. Use **`build`** then **`start`** to test the
> finished, production version before deploying.

## Common Mistakes

- **Forgetting to `cd` into the project folder** before running `npm run dev`. The
  command must run *inside* your project.
- **Editing files in `node_modules/`.** That folder holds installed libraries — never
  edit it by hand. Write your code in `app/`.
- **Renaming special files.** `page.tsx` and `layout.tsx` must keep those exact names,
  or the App Router won't recognize them.
- **Closing the terminal and wondering why the site is gone.** The dev server only
  runs while `npm run dev` is active.
- **Mixing up `dev` and `start`.** Use `dev` for coding; `build` + `start` for the
  production version.

## Exercises

1. **Check your tools.** Run `node --version` and `npm --version`. Confirm you have
   Node.js 18.18 or newer. If not, install the LTS version.
2. **Create a project.** Run `npx create-next-app@latest my-first-app` with the prompt
   answers shown above, then `cd my-first-app` and `npm run dev`. Open
   `http://localhost:3000`.
3. **Edit the home page.** Change the text in `app/page.tsx` and save. Watch the
   browser update instantly thanks to hot reload.
4. **Explore the structure.** Open `package.json` and find the `scripts` section. List
   what `dev`, `build`, and `start` each do in your own words.
5. **Build for production.** Stop the dev server (`Ctrl + C`), then run `npm run build`
   followed by `npm run start`. Visit the site and notice it still works.

## Key Takeaways

- You need **Node.js** (LTS) and **npm** installed before starting.
- `npx create-next-app@latest` scaffolds a full project; choose **TypeScript**,
  **ESLint**, **Tailwind**, and the **App Router**.
- `npm run dev` starts the local development server at `http://localhost:3000`.
- Your code lives in **`app/`**; static files go in **`public/`**.
- Special files: **`page.tsx`** (a page), **`layout.tsx`** (shared wrapper), and
  **`globals.css`** (app-wide styles).
- **Hot reload** updates the browser the instant you save a file.
- Use **`dev`** to build; use **`build`** then **`start`** for the production version.

---

[← Previous: What Is Next.js](01-what-is-nextjs.md) | [Back to Course Index](../README.md) | [Next: Components and JSX →](../01-react-essentials/01-components-and-jsx.md)
