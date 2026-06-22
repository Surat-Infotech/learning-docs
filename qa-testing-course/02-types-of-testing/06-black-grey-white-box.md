# Black, Grey, and White Box Testing

These three terms describe **how much you know about the software's insides** while
testing it. They sound technical, but the idea is simple: is the code a sealed box
you can't see into (black), fully transparent (white), or somewhere between (grey)?
As a manual tester you'll mostly do black box testing — but you should know all
three.

## What You'll Learn

- What black, white, and grey box testing mean
- Who typically does each
- Why manual QA is mostly black box
- How to choose the right view

## The Box Metaphor

Picture the software as a box. The question is: **can you see inside it?**

```text
   BLACK BOX            GREY BOX             WHITE BOX
   ┌────────┐           ┌────────┐           ┌────────┐
   │ ?????? │           │ ░░│code│           │ if(x){ │
   │ ?????? │           │ partial│           │  do(); │
   │ ?????? │           │  view  │           │ } else │
   └────────┘           └────────┘           └────────┘
   see only inputs      some internal        full view of
   & outputs            knowledge            the code
```

## Black Box Testing — Testing from the Outside

You test **only the behavior** — inputs and outputs — with **no knowledge of the
internal code**. You're a user: you click, type, and check results, without caring
*how* it works inside.

- **Who:** QA testers (most manual testing is here).
- **Focus:** does it behave correctly? Functional and non-functional testing are
  usually black box.
- **Example:** you enter a wrong password and check you get an error — you don't
  know or care what code runs.
- **Strength:** tests the software the way real users experience it.
- **Limitation:** you can't see *which* internal paths went untested.

This is your home base. All the design techniques in
[Module 3](../03-test-design-and-documentation/03-test-design-techniques.md)
(equivalence partitioning, boundary values, decision tables) are black box
techniques.

## White Box Testing — Testing from the Inside

You **can see the code** and design tests based on its internal structure — making
sure every line, branch, and path is exercised.

- **Who:** developers (it requires reading code).
- **Focus:** internal logic, code coverage, every `if`/`else` branch.
- **Example:** writing tests so that *both* the "discount applies" and "discount
  doesn't apply" code paths run.
- **Strength:** can find hidden logic errors and dead code.
- **Limitation:** requires programming skills; can miss missing requirements (you
  test what's *there*, not what *should* be there).

Unit testing (from [Module 1](../01-testing-fundamentals/05-levels-of-testing.md))
is typically white box.

## Grey Box Testing — A Bit of Both

You have **partial knowledge** of the internals — maybe you know the database
structure or the API design, but you still test from the outside. You combine a
user's view with some insider insight.

- **Who:** testers with some technical knowledge.
- **Focus:** use internal hints to design *smarter* black box tests.
- **Example:** you know orders are stored in a database table, so after placing an
  order you check the data was saved correctly *and* that the UI shows it.
- **Strength:** more targeted than pure black box, without needing full code access.

## Quick Comparison

| | Black Box | Grey Box | White Box |
| --- | --- | --- | --- |
| **Code knowledge** | None | Partial | Full |
| **Who** | QA testers | Technical testers | Developers |
| **Based on** | Requirements/behavior | Behavior + some internals | Code structure |
| **Skills needed** | No coding | Some technical | Programming |
| **Typical use** | System, acceptance, functional | API/integration testing | Unit testing |

## Where Does This Course Live?

Almost entirely in **black box** (with a little grey box once you learn about
[API testing](../05-tools-and-process/03-api-testing-basics.md) and databases).
You do not need to read code to be an excellent manual tester. You need to
understand behavior, requirements, and the user — which is exactly black box
testing.

## Exercises

1. Define black, white, and grey box testing in one sentence each.
2. For each, say who usually performs it and what knowledge they need.
3. You check that placing an order shows a success message *and* that the order
   appears in the admin database. Which box is this, and why?
4. Why can white box testing miss a missing requirement that black box testing
   might catch?

Next: [Module 3 — Test Design and Documentation](../03-test-design-and-documentation/01-test-scenarios-vs-cases.md)
