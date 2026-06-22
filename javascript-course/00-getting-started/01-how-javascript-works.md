# How JavaScript Works

Welcome to your very first lesson! 🎉 Before you write any code, it helps to know
*what* JavaScript is and *where* it runs. Think of this lesson as a friendly map.
No setup yet — just easy ideas. Let's go!

## What You'll Learn

- What JavaScript is and what people build with it
- Where JavaScript code can run (the **browser** and **Node.js**)
- What a **JavaScript engine** is, in plain words
- What it means that JavaScript runs "line by line"
- The difference between **client-side** and **server-side** code
- A tiny bit of history (just for fun)

## What Is JavaScript?

**JavaScript** is a *programming language*. A **programming language** is a way to
write instructions that a computer can follow. You write the instructions, and the
computer does exactly what you say.

People use JavaScript to build many things:

- **Websites** — the buttons, menus, and animations you click on.
- **Servers** — the programs behind a website that store and send data.
- **Apps** — phone apps and desktop apps can be built with JavaScript too.

Here is a tiny example of JavaScript. Do not worry about understanding it yet —
just notice that it looks like short English instructions:

```js
// This line shows a message
console.log("Hello! I am JavaScript.");
// => Hello! I am JavaScript.
```

> A **comment** is a note for humans. It starts with `//`. The computer ignores it.

Think of JavaScript like a recipe. A recipe is a list of steps. The cook (the
computer) follows your steps one at a time.

## Where Does JavaScript Run?

Code does not float in the air. It needs a *place* to run. JavaScript has two main
homes.

### 1. The Browser

A **browser** is the program you use to visit websites, like Chrome, Firefox, Edge,
or Safari. Every modern browser can run JavaScript. This is the original home of
JavaScript.

When a website wants a button to do something when you click it, JavaScript running
*inside your browser* makes that happen.

### 2. Node.js

For a long time, JavaScript could only run inside a browser. Then **Node.js** was
created. Node.js lets JavaScript run *outside* the browser — directly on a computer
or a server.

A **server** is just a computer that runs all the time and answers requests from
other computers (for example, sending you a web page when you visit a site).

So the same language, JavaScript, can run in two places:

| Place        | What it is              | Example use                       |
|--------------|-------------------------|-----------------------------------|
| **Browser**  | Where you view websites | Make a button change color        |
| **Node.js**  | Runs on your computer   | Build a server, run scripts/tools |

Analogy: Imagine you speak English. You can speak it at home (the browser) and also
at work (Node.js). Same language, different places.

## What Is a JavaScript Engine?

A **JavaScript engine** is the part of a browser or Node.js that actually *reads*
your JavaScript and *runs* it. It is the "brain" that understands the language.

You do not install the engine yourself — it comes built in.

- Chrome and Node.js use an engine called **V8**.
- Firefox uses one called **SpiderMonkey**.

You almost never think about the engine. Just know this: when you write JavaScript,
an engine quietly takes your words and turns them into actions the computer can do.

Analogy: The engine is like a translator. You write in JavaScript, and the engine
translates it into the language the computer truly speaks.

## JavaScript Runs Line by Line

JavaScript is an **interpreted** language. **Interpreted** means the engine reads
your code and runs it *step by step, from top to bottom*, one line at a time.

```js
console.log("Step 1"); // runs first
console.log("Step 2"); // runs second
console.log("Step 3"); // runs third
// => Step 1
// => Step 2
// => Step 3
```

The order matters! The engine starts at the top and works its way down, just like
you read a page from top to bottom.

## Client-Side vs Server-Side

These two words sound fancy, but they are simple.

- **Client-side** means code that runs on the *user's* computer — usually in the
  browser. The "client" is the person visiting the website. This code controls what
  you *see and click*.

- **Server-side** means code that runs on a *server* (often using Node.js). This
  code handles things behind the scenes, like saving your account or fetching data.

Analogy: Think of a restaurant.

- **Client-side** is the dining area — what the customer sees and touches.
- **Server-side** is the kitchen — where the work happens out of sight.

```js
// Example idea (not real code to run):
// Client-side: show a "Loading..." message to the user
// Server-side: look up the user's data and send it back
```

The great thing about JavaScript is that you can use it on *both* sides.

## A Tiny Bit of History

JavaScript was created in 1995 — very quickly, in about ten days! It was made to add
small interactive features to web pages. Over the years it grew into one of the most
popular languages in the world.

Fun fact: even though the name has "Java" in it, **JavaScript and Java are not the
same thing**. They are two different languages with similar names. Do not mix them
up!

## Common Mistakes

- **Thinking JavaScript and Java are the same.** They are different languages. This
  course is about *JavaScript*.
- **Believing JavaScript only works in the browser.** Thanks to Node.js, it also
  runs on servers and computers.
- **Forgetting that order matters.** Code runs top to bottom, line by line. A line
  near the bottom runs *after* the lines above it.
- **Worrying about the engine.** You do not need to understand how V8 works inside.
  Just know it runs your code for you.

## Key Takeaways

- JavaScript is a **programming language** used for websites, servers, and apps.
- It runs in two main places: the **browser** and **Node.js**.
- A **JavaScript engine** (like **V8**) reads and runs your code.
- JavaScript is **interpreted** — it runs **line by line**, top to bottom.
- **Client-side** = runs in the user's browser. **Server-side** = runs on a server.
- JavaScript is *not* the same as Java.

---

[← Back to Course Index](../README.md) &nbsp;&nbsp;|&nbsp;&nbsp; [Next: Setup and Running Your First Code](02-setup-and-running-code.md)
