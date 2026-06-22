# The Call Stack and Event Loop

Imagine a single cook in a small kitchen. There is only one cook. So only one
dish can be actively worked on at a time. But the cook is smart. When something
needs to bake in the oven for 10 minutes, the cook does not just stand and
stare at the oven. The cook starts other tasks while waiting, and comes back
when the oven beeps.

JavaScript works in a very similar way. It does one thing at a time, but it is
clever about waiting. This lesson explains *how* it does that. These are the
ideas behind every "async" feature you will learn next: callbacks, promises,
and `async`/`await`.

Do not worry if it feels strange at first. Read it slowly. The picture will
become clear.

## What You'll Learn

- What **synchronous** and **asynchronous** mean in plain words
- Why JavaScript is **single-threaded** (does one thing at a time)
- What the **call stack** is (a stack of running functions)
- What **blocking** means and why it is bad
- What the **event loop**, **callback queue**, and **microtask queue** are
- Why `setTimeout(fn, 0)` does **not** run right away

## Synchronous vs Asynchronous (In Plain Words)

**Synchronous** means "one thing at a time, in order". You do step 1, then step
2, then step 3. Each step waits for the one before it to finish.

```js
console.log("1. Wake up");
console.log("2. Brush teeth");
console.log("3. Eat breakfast");
// => 1. Wake up
// => 2. Brush teeth
// => 3. Eat breakfast
```

This runs top to bottom. Simple and predictable.

**Asynchronous** means "start something now, and continue with other work while
it finishes in the background". You do not stand and wait.

A real-world example: you put bread in the toaster (start the task), and while
it toasts, you pour your juice (do other work). When the toast pops up, you deal
with it. You did not freeze in front of the toaster.

The word "asynchronous" looks scary, but it just means: **not waiting in line**.
Some tasks (like getting data from the internet) take time. Async lets your
program keep going instead of freezing.

## JavaScript Is Single-Threaded

A **thread** is like a worker. JavaScript has only **one** worker. This is what
"single-threaded" means: JavaScript can run only one piece of code at a single
moment.

So how can it do async things? It hands slow jobs (like timers and downloads)
to the browser to handle in the background. The single worker stays free. When
the slow job is done, JavaScript picks the result back up. We will see exactly
how below.

> One worker, but a good system for not wasting time waiting. That is the whole
> trick.

## The Call Stack

The **call stack** is how JavaScript keeps track of which function is running
right now. A **stack** is like a pile of plates: you add to the top, and you
remove from the top. The last plate you put on is the first one you take off.

When you call a function, it gets **pushed** (added) onto the stack. When the
function finishes, it gets **popped** (removed) off the stack.

```js
function greet() {
  console.log("Hello");
}

function start() {
  greet(); // greet goes on top of start
}

start(); // start goes on the stack first
```

Here is what happens, step by step:

1. `start()` is called. Push `start` onto the stack.
2. Inside `start`, `greet()` is called. Push `greet` on top.
3. `greet` runs `console.log("Hello")`. That finishes. Pop `console.log`.
4. `greet` is done. Pop `greet`.
5. `start` is done. Pop `start`.
6. The stack is now empty.

The function on **top** of the stack is the one running right now.

## What Blocking Means

Because there is only one worker and one call stack, a **slow** function will
**block** everything else. Blocking means: nothing else can run until the slow
function finishes.

```js
// Imagine this loop takes 5 whole seconds to finish.
function slowTask() {
  let total = 0;
  for (let i = 0; i < 5000000000; i++) {
    total += i;
  }
  return total;
}

console.log("Start");
slowTask();          // BLOCKS here for ~5 seconds
console.log("End");  // only runs after slowTask finishes
```

During those 5 seconds, the page freezes. Buttons do not click. Nothing moves.
This is bad. Users hate frozen pages.

The fix is to do slow work **asynchronously**, so the single worker stays free.
This is the reason async exists.

## The Event Loop (The Chef Analogy)

Now the main idea. Picture a chef (the single JavaScript worker) in a kitchen.

- The **call stack** is the chef's current cooking station. The chef can only
  cook one dish at a time here.
- The **kitchen helpers** are the browser. When the chef gets a slow order
  ("bake this for 10 minutes"), the chef hands it to a helper and keeps cooking
  other things. The chef does **not** wait.
- When a helper finishes (the timer rings, the data arrives), it does not
  interrupt the chef. Instead, it puts a note in a **waiting line of orders**.
- The **event loop** is the rule the chef follows: *"Whenever my station is
  empty, I pick up the next note from the waiting line and cook it."*

So the flow is:

1. Run all the normal code on the call stack.
2. Slow jobs are sent to the browser to handle in the background.
3. When a slow job finishes, its follow-up code waits in a queue.
4. The **event loop** checks: is the call stack empty? If yes, take the next
   item from the queue and run it.

A **queue** is a line, like at a shop: first in, first out. The opposite of a
stack.

## The Callback Queue and the Microtask Queue

There are actually **two** waiting lines, and one is more important than the
other.

- The **callback queue** (also called the "task queue") holds follow-up code
  from things like `setTimeout` and clicks.
- The **microtask queue** holds follow-up code from **promises** (you will
  learn promises soon).

**The rule:** after the call stack is empty, the event loop empties the **whole
microtask queue first**, and only then takes **one** item from the callback
queue. Microtasks (promises) cut to the front of the line.

You do not need to memorize this today. Just remember: **promises run before
timers**. This explains the surprising output below.

```js
console.log("1: start");

setTimeout(() => {
  console.log("2: timeout"); // callback queue (waits longer)
}, 0);

Promise.resolve().then(() => {
  console.log("3: promise"); // microtask queue (runs first)
});

console.log("4: end");

// => 1: start
// => 4: end
// => 3: promise
// => 2: timeout
```

The order is `1, 4, 3, 2`. The normal code (`1` and `4`) runs first. Then the
promise microtask (`3`). Then the timer callback (`2`) last.

## Why `setTimeout(fn, 0)` Does Not Run Immediately

`setTimeout(fn, 0)` looks like it should run `fn` right now, with zero delay.
But it does **not**.

The `0` means "wait at least 0 milliseconds", not "run instantly". The function
`fn` is sent to the browser, then placed in the callback queue. The event loop
will only run it **after** the current code on the call stack is completely
finished.

```js
console.log("A");

setTimeout(() => {
  console.log("B");
}, 0);

console.log("C");

// => A
// => C
// => B   (runs last, even though the delay is 0)
```

Think of it like this: the chef must finish the dish in their hands before
picking up any note from the waiting line, even a note that says "do this with
zero delay". `setTimeout(fn, 0)` is a polite way of saying *"run this as soon as
you are free, but not before"*.

## Common Mistakes

- **Thinking JavaScript does many things at once.** It does one thing at a time.
  The async trick is to hand slow jobs to the browser, not to run code in
  parallel.
- **Expecting `setTimeout(fn, 0)` to run instantly.** It always waits until the
  current code finishes first.
- **Writing slow, blocking loops.** A long synchronous loop freezes the whole
  page. Prefer async for slow work.
- **Confusing a stack and a queue.** A **stack** is last-in-first-out (call
  stack). A **queue** is first-in-first-out (the waiting lines).
- **Forgetting promises jump ahead of timers.** Microtasks (promises) run
  before callback-queue tasks (timers).

## Exercises

1. Without running it, predict the output order of this code, then run it to
   check:
   ```js
   console.log("one");
   setTimeout(() => console.log("two"), 0);
   console.log("three");
   ```
2. Add a `Promise.resolve().then(() => console.log("four"))` line to the code
   above. Predict where `"four"` will appear in the output, then verify.
3. In your own words, explain to a friend (or write down) what "blocking" means
   and why a frozen web page is an example of it.
4. Explain the chef analogy in your own words: what is the call stack, what is
   the waiting line, and what is the event loop?

## Key Takeaways

- **Synchronous** = one thing at a time, in order. **Asynchronous** = start
  something and keep going while it finishes in the background.
- JavaScript is **single-threaded**: only one piece of code runs at a time.
- The **call stack** tracks running functions. It is last-in, first-out.
- **Blocking** is when slow code stops everything else (a frozen page).
- The **event loop** runs waiting code only when the call stack is empty.
- The **microtask queue** (promises) runs before the **callback queue**
  (timers).
- `setTimeout(fn, 0)` waits for the current code to finish; it never runs
  instantly.

---

[← Back to Course Index](../README.md) | [Next: Callbacks →](02-callbacks.md)
