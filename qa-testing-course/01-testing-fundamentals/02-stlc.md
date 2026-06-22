# The Software Testing Life Cycle (STLC)

The SDLC describes how *software* is built. The **Software Testing Life Cycle
(STLC)** zooms in on the testing part and breaks *it* into clear stages. Knowing
the STLC tells you what a structured testing effort looks like — so you're never
just "clicking around hoping to find bugs."

## What You'll Learn

- The phases of the testing life cycle
- What "entry criteria" and "exit criteria" mean
- The artifacts (documents) testers produce at each phase
- How the STLC sits inside the bigger SDLC

## The Phases

```text
  1. Requirement   2. Test       3. Test Case   4. Environment  5. Test       6. Closure
     Analysis    → Planning   →   Development  →    Setup     →  Execution →
  (understand)   (strategy)      (write tests)    (get ready)   (run them)   (wrap up)
```

### 1. Requirement Analysis

Read and understand what's being built. Ask questions about anything vague.
Identify what's testable and what isn't.

- **Output:** a list of questions, and clarity on what to test.

### 2. Test Planning

Decide the **strategy**: what to test, what *not* to test, who does it, what tools,
how long, what risks. The team lead often owns this.

- **Output:** a **Test Plan** (covered in
  [Module 3](../03-test-design-and-documentation/04-test-plans.md)).

### 3. Test Case Development

Write the actual **test cases** and **test data** — the step-by-step checks you'll
run.

- **Output:** test cases and test data sets.

### 4. Test Environment Setup

Get the place where you'll test ready: the right app version, test accounts,
sample data, devices, browsers.

- **Output:** a working environment to test in.

### 5. Test Execution

Run the test cases. Mark each **Pass** or **Fail**. Log bugs for failures. Retest
fixes.

- **Output:** test results and bug reports.

### 6. Test Cycle Closure

Wrap up. Did we test enough? What did we learn? Write a summary report and capture
lessons for next time.

- **Output:** a **Test Closure / Summary Report**.

## Entry and Exit Criteria

Two phrases you'll hear a lot. They are simply checklists that gate each phase.

- **Entry criteria** — what must be true *before* you can start a phase.
  > *Entry for Test Execution:* test cases are written, the environment is ready,
  > and the build is deployed.
- **Exit criteria** — what must be true *before* you can call a phase done.
  > *Exit for Test Execution:* all planned test cases run, no open critical bugs,
  > results documented.

Criteria stop two common mistakes: starting work before you're ready, and
declaring "done" when you actually aren't.

## STLC Inside SDLC

The STLC is not separate from the SDLC — it runs **alongside** it. In Agile, a
mini-STLC happens inside *every sprint*: analyze the story, plan, write cases, set
up, execute, close — in two weeks, then repeat.

```text
  SDLC stage:   Requirements   Design    Development        Testing       Deployment
  STLC phase:   Analysis    →  Planning  Case Dev + Setup →  Execution  →  Closure
                (testers start thinking here, not at the Testing stage)
```

Notice testers begin in the Requirements stage — not waiting idly until code is
ready.

## Exercises

1. List the six STLC phases in order and write one sentence describing each.
2. Write a sensible **entry** criterion and **exit** criterion for the *Test
   Execution* phase.
3. Why is it risky to start Test Execution before the environment setup is
   finished?
4. In an Agile sprint of two weeks, how would the six STLC phases be compressed?

Next: [Verification vs Validation](03-verification-vs-validation.md)
