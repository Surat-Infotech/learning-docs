# What Is Next.js (and Why Use It)?

Welcome to your first lesson! 🎉 Before you write any code, let's understand
*what* Next.js is and *why* so many people use it. Think of Next.js like a
fully-equipped kitchen: React gives you the ingredients, and Next.js gives you
the stove, the oven, and the recipe cards so you can actually cook. No setup yet —
just friendly ideas. Let's go!

## What You'll Learn

- What a **framework** is, in plain words
- What **React** is, in one short paragraph
- What **Next.js** adds on top of React (routing, rendering, data, bundling)
- The gentle difference between **client-side**, **server-side**, and **static** rendering
- What kinds of apps you can build with Next.js
- The difference between the **App Router** and the **Pages Router** (we use App Router)
- When you might *not* need Next.js

## What Is a Framework?

A **framework** is a ready-made structure that helps you build software faster. It
makes many decisions for you and gives you tools that fit together nicely, so you
don't have to wire everything up by hand.

Analogy: building a website from scratch is like building a house starting with raw
lumber. A **framework** is like a pre-built frame for the house — the walls and
floors are already in place, so you just decorate the rooms.

Frameworks save you time. Instead of solving the same hard problems over and over
(like "how do I show a different page when the URL changes?"), the framework already
has a sensible answer built in.

## What Is React?

**React** is a popular JavaScript **library** for building user interfaces — the
visual parts of an app that people see and click. With React you build small,
reusable pieces called **components** (think: a button, a navbar, a card), and React
keeps the screen updated when your data changes. A **library** is a collection of
helpful code you call when you need it; React focuses on *one* job: drawing the UI.

> A **component** is a reusable chunk of UI you can use again and again, like a
> LEGO brick you snap into place.

React is powerful, but it is *just* the UI part. It does not tell you how to handle
pages, fetch data, or prepare your app for the internet. That's where Next.js comes in.

## What Does Next.js Add on Top of React?

**Next.js** is a **framework** built *on top of* React. You still write React
components, but Next.js adds the missing pieces you need to build a complete,
real-world website. Here are the big ones:

### 1. Routing (pages from folders)

**Routing** means deciding which page to show for each web address (URL). In
Next.js, you create pages just by making folders and files — no extra setup.

```text
app/
  page.tsx          -> the home page  (yoursite.com/)
  about/
    page.tsx        -> the about page (yoursite.com/about)
```

The folder structure *is* your URL structure. Simple and visual.

### 2. Server Rendering

Next.js can build your HTML on the **server** (a computer that runs all the time)
before sending it to the user. This makes pages load fast and helps search engines
read your content. We'll explain this more below.

### 3. Data Fetching

Next.js gives you easy, built-in ways to fetch data (for example, from a database or
another website) and show it on the page — often right on the server, before the
page even reaches the browser.

### 4. Bundling and Optimization

A **bundler** packs all your code and files into small, efficient pieces the browser
can download quickly. Next.js does this automatically. It also optimizes images,
fonts, and JavaScript so your site feels snappy — without you configuring anything.

Analogy: React gives you the engine. Next.js gives you the whole car — wheels,
steering, dashboard — so you can actually drive somewhere.

## Rendering: Client, Server, and Static (Gently)

"**Rendering**" just means *turning your code into the HTML a user sees*. There are
three common ways to do it. Don't memorize these — just get the feel.

### Client-Side Rendering (CSR)

The browser downloads a mostly-empty page plus some JavaScript, then *builds* the
page right there on the user's computer.

- ✅ Great for highly interactive screens.
- ⚠️ The first load can feel slow, and search engines may see an empty page.

### Server-Side Rendering (SSR)

The **server** builds the full HTML *for each request*, then sends it ready-to-view.

- ✅ Fast first paint and good for search engines.
- ✅ Perfect when content changes often or depends on the user.

### Static Rendering (SSG)

The HTML is built *once*, ahead of time (when you publish the site), and reused for
everyone.

- ✅ Blazing fast and cheap, because the page is already made.
- ⚠️ Best for content that rarely changes, like a blog post or docs.

Analogy for all three:

| Type        | Like ordering food...                              |
|-------------|----------------------------------------------------|
| **Client**  | You get raw ingredients and cook at home           |
| **Server**  | The kitchen cooks your dish fresh when you order   |
| **Static**  | Pre-made meals on a shelf, ready to grab instantly |

The good news: Next.js lets you mix and match these *per page*, and it picks smart
defaults so you usually don't have to think hard about it.

## What Can You Build With Next.js?

Lots of things! Next.js is flexible:

- **Marketing and landing pages** — fast, SEO-friendly sites.
- **Blogs and documentation** — mostly static, super quick.
- **Dashboards and web apps** — interactive tools behind a login.
- **E-commerce stores** — product pages, carts, checkout.
- **Full-stack apps** — Next.js can run backend code too, so the front and back live
  in one project.

If it's a website or web app, Next.js can probably build it.

## App Router vs Pages Router

Next.js has had two ways to organize a project. You may see both online, so it helps
to know the difference.

- **Pages Router** — the *older* system. Pages live in a `pages/` folder. Still
  works, but it's the legacy approach.
- **App Router** — the *newer*, recommended system (since Next.js 13). Pages live in
  an `app/` folder. It supports the latest features, like server components.

> 📌 **This course uses the App Router** (the `app/` folder) throughout. If a
> tutorial online uses a `pages/` folder, it's the older style — be careful mixing
> the two.

You don't need to understand every feature yet. Just remember: we use the **App
Router**, and our code lives in `app/`.

## When NOT to Use Next.js

Next.js is great, but it's not always the right tool. You might skip it when:

- **You're building a tiny static page.** A single HTML file may be all you need.
- **You don't need React at all.** If your site has almost no interactivity, plain
  HTML and CSS could be simpler.
- **You're learning core React basics in isolation.** Sometimes a plain React setup
  (or an online playground) is a gentler first step.
- **You need a non-web app**, like a native mobile app — other tools fit better.

Using a big framework for a one-page site is like renting a moving truck to carry a
single grocery bag. For *most* real websites, though, Next.js is a fantastic choice.

## Common Mistakes

- **Thinking Next.js replaces React.** It *builds on* React. You still write React
  components.
- **Confusing the App Router and Pages Router.** We use the **App Router** (`app/`
  folder). Don't follow `pages/`-based tutorials by accident.
- **Believing you must pick one rendering type forever.** Next.js lets you choose per
  page and gives smart defaults.
- **Reaching for Next.js for every tiny page.** For a single static page, simpler
  tools may be enough.

## Key Takeaways

- A **framework** gives you a ready-made structure so you build faster.
- **React** is a library for building UIs out of reusable **components**.
- **Next.js** is a framework on top of React that adds **routing**, **server
  rendering**, **data fetching**, **bundling**, and **optimization**.
- Pages can be **client-rendered**, **server-rendered**, or **static** — Next.js
  mixes them and picks smart defaults.
- You can build blogs, dashboards, stores, and full-stack apps.
- This course uses the **App Router** (the `app/` folder), not the Pages Router.

---

[← Back to Course Index](../README.md) | [Next: Setup and Project Structure →](02-setup-and-project-structure.md)
