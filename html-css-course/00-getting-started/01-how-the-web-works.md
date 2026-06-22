# How the Web Works

Before you build websites, it helps to understand what a website really is and what happens when you visit one. This lesson is all about the big picture. We won't write any code yet, we'll just build a clear mental model so everything that comes later makes sense.

## What You'll Learn

- What a website actually is
- What a web browser does
- The difference between a **client** and a **server**
- What a **URL** is and what its parts mean
- What an HTTP **request** and **response** are, in plain terms
- The three languages of the web: HTML, CSS, and JavaScript
- How a browser turns plain text into the pages you see

## What Is a Website?

A **website** is just a collection of files that live on a computer somewhere in the world. These files contain text, images, and instructions. When you visit a website, your computer asks for those files and shows them to you.

That's it. A website is not magic. It's files. The skill you're learning in this course is how to write those files so that a browser knows what to do with them.

A single screen of a website, like the page you're reading now, is called a **web page**. A website is usually made up of many web pages linked together.

## The Browser: Your Window to the Web

A **web browser** is a program that lets you view websites. You're using one right now. Common browsers include Google Chrome, Firefox, Safari, and Microsoft Edge.

Think of the browser as a **translator and a TV screen combined**. The website sends it a bunch of text instructions, and the browser reads those instructions and paints them onto your screen as something you can see and click.

The most important idea: **the browser reads files written in special languages and turns them into the colorful, clickable pages you know.** Your job as a web developer is to write those files.

## Client and Server: A Conversation Between Two Computers

When you use the web, two computers are talking to each other.

- The **client** is your computer (or phone). It is the one *asking* for something.
- The **server** is a powerful computer that *stores* websites and *sends* them out when asked.

Here's a simple analogy. Imagine ordering food at a restaurant.

- **You are the client.** You sit at the table and ask for a meal.
- **The kitchen is the server.** It has the food, prepares it, and hands it back to you.

You don't go into the kitchen yourself. You just ask, and you receive. The web works the same way. Your browser (the client) asks a server for a web page, and the server sends it back.

## What Is a URL?

A **URL** (Uniform Resource Locator) is the address of a web page. It's the text you type into the top bar of your browser, like:

```text
https://www.example.com/about
```

Think of a URL like a postal address for a house. It tells your browser exactly where to go. Let's break this one into parts.

```text
https://   www.example.com   /about
   |             |              |
protocol       domain         path
```

- **`https://`** is the **protocol**. It's the set of rules the computers agree to use when talking. `https` means "secure," so the conversation is private.
- **`www.example.com`** is the **domain**. This is the name of the server you want to reach, like the name of a specific restaurant.
- **`/about`** is the **path**. It points to a specific page on that website, like asking for a specific dish from the menu.

When you type a URL and press Enter, you are telling your browser, "Go to this address and bring me back what's there."

## HTTP: How the Browser and Server Talk

So how does the client actually ask the server for a page? It uses a set of rules called **HTTP** (HyperText Transfer Protocol). Don't worry about the long name. Just think of HTTP as the polite language both computers speak.

The conversation always has two parts:

### The Request

The **request** is the question your browser asks. When you visit a URL, your browser sends a message that basically says:

```text
"Hello server. Please send me the page at /about."
```

### The Response

The **response** is the answer the server sends back. It usually contains the files for the page, along with a small status note. You may have seen one famous status note: **404**, which means "I looked, but I couldn't find that page."

A successful response says **200**, which simply means "OK, here it is."

So the full cycle is:

```text
1. You type a URL and press Enter.
2. Your browser (client) sends a REQUEST to the server.
3. The server finds the files and sends back a RESPONSE.
4. Your browser reads the files and shows you the page.
```

This whole back-and-forth usually happens in less than a second.

## The Three Languages of the Web

Every modern web page is built from three languages. Each one has a clear job. The best way to understand them is to imagine building a **house**.

### HTML — The Structure

**HTML** (HyperText Markup Language) is the **structure** of the page. It defines what content exists: headings, paragraphs, images, buttons, and links.

In our house analogy, **HTML is the bricks and the frame**. It's the walls, the rooms, the doors, and the windows. Without HTML, there is no house at all. This is the very first language you'll learn, and it's the foundation of everything.

```text
HTML = the bricks and walls (structure)
```

### CSS — The Style

**CSS** (Cascading Style Sheets) is the **style** of the page. It controls how things look: colors, fonts, spacing, and layout.

In our analogy, **CSS is the paint, the wallpaper, and the decoration**. The house already stands without it, but CSS makes it beautiful and pleasant to live in. A plain HTML page works, but a styled page looks professional.

```text
CSS = the paint and decoration (style)
```

### JavaScript — The Behavior

**JavaScript** is the **behavior** of the page. It makes pages interactive: things that respond when you click, type, or scroll.

In our analogy, **JavaScript is the electricity and plumbing**. It's the light switches that turn on the lights and the taps that make water flow. It brings the house to life. (You'll meet JavaScript later, after you're comfortable with HTML and CSS.)

```text
JavaScript = the electricity (behavior)
```

Put together: **HTML builds it, CSS beautifies it, JavaScript animates it.**

## What a Web Page Really Is

Here's a secret that surprises many beginners: **a web page is just a text file.**

When you write HTML, you are typing plain text, the same kind of text you'd write in a simple notes app. There's nothing hidden. If you could peek inside any website, you'd find text files full of instructions.

This is great news. It means you don't need any expensive or fancy software to build a website. You just need a way to write text and a browser to read it.

## How the Browser Turns HTML Into a Page

When the browser receives your HTML text file, it goes through a process called **rendering**. "Rendering" simply means turning the instructions into a picture on screen.

Very simply, here's what happens:

```text
1. The browser reads your HTML text from top to bottom.
2. It understands the structure: "This is a heading, this is a paragraph."
3. It applies any CSS to decide colors, sizes, and positions.
4. It paints the final result onto your screen as pixels.
```

So the browser is like a builder reading a blueprint. You hand it the plan (your HTML and CSS), and it constructs the visible page for you. Every website you've ever visited went through exactly this process.

## Common Mistakes

- **Thinking websites are too complex to understand.** At their core, they are just text files that a browser reads. You can absolutely learn this.
- **Confusing the client and the server.** Remember: the client (your browser) *asks*, and the server *answers*.
- **Believing you need special, expensive software.** A simple text editor and a browser are all you need to start.
- **Mixing up the three languages.** HTML is structure, CSS is style, JavaScript is behavior. Keep the house analogy in mind.
- **Assuming a URL is random text.** Every part of a URL has a meaning and points your browser somewhere specific.

## Exercises

These exercises need no code. They build your understanding.

1. Open your browser and visit any website you like. Look at the URL in the address bar and try to name its three parts: the protocol, the domain, and the path.
2. In your own words, explain the difference between a client and a server using an example that is *not* a restaurant.
3. Pick a website you use often. Describe one thing on it that is **HTML** (a piece of content), one thing that is **CSS** (a color or style), and one thing that is **JavaScript** (something interactive).
4. Try visiting a web page that does not exist, for example by adding `/this-page-does-not-exist` to the end of a website's address. See what response you get. Can you spot a 404?
5. Explain to a friend or family member what happens, step by step, between pressing Enter on a URL and seeing the page appear.

## Key Takeaways

- A website is just a collection of files; a web page is a single screen of one.
- The **browser** reads those files and renders them into what you see.
- The **client** asks for pages; the **server** stores and sends them.
- A **URL** is a web address made of a protocol, a domain, and a path.
- **HTTP** is the language clients and servers use: a **request** asks, and a **response** answers.
- The three web languages are **HTML** (structure), **CSS** (style), and **JavaScript** (behavior), like the bricks, paint, and electricity of a house.
- A web page is plain text, and **rendering** is how the browser turns that text into a picture on screen.

---

[← Back to Course Index](../README.md) | [Next: Setup and Your First Page →](02-setup-and-your-first-page.md)
