# What Is QA (and Why It Matters)

Imagine a car factory. Before any car leaves the building, someone checks the
brakes, the lights, the doors, and the engine. They are not the people who *built*
the car — their whole job is to make sure it is safe and works before a customer
ever sits in it. In software, that person is **Quality Assurance (QA)**.

## What You'll Learn

- What "quality" actually means in software
- The difference between Quality Assurance, Quality Control, and Testing
- What a QA tester does day to day
- Why companies pay for QA at all
- Manual QA vs automation QA (and where this course fits)

## What Is Quality?

Quality is simply: **does the software do what users need, reliably, without
unpleasant surprises?** A high-quality app:

- Does what it promises (you click "Save" and it actually saves)
- Is easy to use (people can figure it out without a manual)
- Does not crash or lose your data
- Behaves well even when the user does something unexpected
- Works on the devices and browsers your users actually have

Quality is not "zero bugs" — that is impossible. Quality is **knowing the state of
the product** and shipping with confidence instead of crossed fingers.

## QA vs QC vs Testing

These three words get mixed up constantly. Here is the simple version:

| Term | What it focuses on | Example |
| --- | --- | --- |
| **Quality Assurance (QA)** | The *process*. Preventing bugs. | "Let's review requirements before we build, so we don't build the wrong thing." |
| **Quality Control (QC)** | The *product*. Finding bugs. | "Let's test the login screen and report what's broken." |
| **Testing** | One activity inside QC. | "I ran the login test cases; two failed." |

Think of it as **prevention (QA) vs detection (QC)**, with **testing** being the
hands-on detection work. In real jobs these titles blur — a "QA Tester" usually
does all three. Don't worry about being a purist; just know the ideas.

## What Does a QA Tester Actually Do?

A typical day might include:

- **Reading requirements** — understanding what a feature is *supposed* to do.
- **Writing test cases** — step-by-step checks like "enter a wrong password →
  expect an error message."
- **Running tests** — actually clicking through the app and comparing what happens
  to what *should* happen.
- **Reporting bugs** — writing clear reports developers can act on.
- **Re-testing fixes** — confirming a fix really worked and broke nothing else.
- **Talking to the team** — in stand-ups, planning, and bug triage meetings.

Notice: a lot of QA is **thinking and communicating**, not just clicking.

## Why Companies Pay for QA

Because bugs are expensive — and they get more expensive the later you find them.

```text
   Cost to fix a bug, by when it's caught:

   Requirements  →  Design  →  Coding  →  Testing  →  Production
       $            $$          $$$        $$$$         $$$$$$$$
   (cheap to fix here)                       (very costly here)
```

A bug caught while reading requirements is a quick conversation. The same bug
caught after launch can mean angry customers, refunds, emergency fixes at 2 a.m.,
and damaged trust. Good QA pushes bug-catching **earlier**, where it's cheap. This
idea is sometimes called **"shift left."**

## Manual QA vs Automation QA

- **Manual QA** — a human runs the tests by hand: clicking, typing, observing,
  using judgment. Great for new features, exploratory testing, and anything where
  human eyes matter (does this *look* right? does it *feel* confusing?).
- **Automation QA** — code runs the tests automatically and repeatedly. Great for
  large, repetitive checks like running 500 regression tests every night.

They are not rivals — strong teams use both. **This course teaches manual QA
fundamentals**, the foundation every tester needs. Automation builds *on top* of
these skills; you cannot automate a test you don't first understand manually.

## Exercises

1. In your own words, explain the difference between QA and QC to a friend who has
   never heard the terms.
2. Pick an app you use daily. List three things that make it feel "high quality"
   and one thing that annoys you (a quality gap).
3. Why does fixing a bug in production cost more than fixing it during design?
   Give a concrete example.
4. Would you use manual or automated testing to check a brand-new screen no one
   has ever seen? Why?

Next: [The Tester's Mindset](02-the-testers-mindset.md)
