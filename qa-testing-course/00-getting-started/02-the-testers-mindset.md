# The Tester's Mindset

Developers and testers look at the same screen and see different things. A
developer thinks, *"Here's how this works."* A tester thinks, *"Here's how this
might break."* That small shift in attitude is the single most important tool you
own. You don't need fancy tools to be a great tester — you need the right way of
thinking.

## What You'll Learn

- The core attitude that separates good testers from button-clickers
- Common ways software breaks that beginners forget to check
- Why "it works" is not the same as "it's done"
- How to ask better questions about a feature

## Build, Then Break

A builder's instinct is to confirm something works: type the right password, click
the right buttons, get the happy result. This is called the **happy path** — and
it's the *least* interesting path to a tester.

Your job is to wander off the happy path:

- What if the user leaves a field **blank**?
- What if they type **letters where numbers are expected**?
- What if they click **Submit twice**?
- What if their **internet drops** halfway through?
- What if they paste **10,000 characters** into a name field?
- What if they hit the **Back button** after paying?

A good tester is professionally curious and a little bit mischievous. You're not
trying to "break" things to be annoying — you're finding the breaks **before a real
user does.**

## The Magic Question: "What If?"

Every feature is a stack of assumptions. Your job is to poke at each one.

> Feature: "Users can upload a profile picture."

Assumptions hiding in that one sentence:

- What if the file is **not** a picture (a `.zip`, a `.exe`)?
- What if the picture is **huge** (50 MB)?
- What if it's **tiny** (1 pixel)?
- What if the upload is **interrupted**?
- What if they upload, then upload again — does the old one get replaced?
- What if **two users** upload at the same time?

You just turned one vague sentence into half a dozen real tests. That's the skill.

## "Works on My Machine" Is Not Enough

A feature is not done because it worked **once, on your computer, with your nice
internet, doing exactly the expected steps.** Real users have:

- Old phones and slow connections
- Different browsers and screen sizes
- Different languages, time zones, and keyboards
- A habit of doing things in a weird order

A tester mentally represents **all those users** the developer never met.

## Useful Categories to Probe

When you look at any feature, run through these buckets. They jog your memory for
"what if" questions:

- **Empty** — nothing entered, no items, blank state.
- **Wrong type** — text in a number field, emoji in a name.
- **Too much / too little** — very long, very short, zero, negative.
- **Boundaries** — exactly the minimum, exactly the maximum, one over.
- **Interruptions** — refresh, back button, lost connection, closed tab.
- **Repeats** — double-click, resubmit, duplicate entry.
- **Permissions** — logged out, wrong user, expired session.

We'll turn these instincts into formal *techniques* later (Module 3). For now,
just practice the **attitude**.

## Balance: Be Critical, Not Cruel

The mindset is skeptical about the *software*, never about the *people*. You will
work closely with developers, and the goal is a shared one: a great product. Report
bugs about the product, not blame about the person. We cover this teamwork in
[Module 4](../04-defect-management/04-working-with-developers.md).

## Exercises

1. Take this feature: *"Users can search for products by name."* Write **eight**
   "what if?" questions using the categories above.
2. Watch a friend or family member use an app. Note every moment they hesitate,
   tap the wrong thing, or look confused. Those are usability findings.
3. Open the login screen of any website. Try to make it show an error message in
   five different ways. Write down what you did and what happened.
4. Explain why "it worked when I tried it" is a weak statement for a tester to
   make.

Next: [Module 1 — Testing Fundamentals](../01-testing-fundamentals/01-sdlc-and-where-qa-fits.md)
