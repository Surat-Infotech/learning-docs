# Test Case Design Techniques

You can't test every possible input
([principle 2](../01-testing-fundamentals/04-principles-of-testing.md)). So how do
you pick the *few* inputs that find the *most* bugs? You use **test design
techniques** — proven, systematic ways to choose high-value test cases instead of
guessing. Master these four and you'll test smarter, not harder.

## What You'll Learn

- Equivalence partitioning
- Boundary value analysis
- Decision table testing
- State transition testing
- When to reach for each

## Why We Need Techniques

Imagine a field that accepts ages 18–60. That's 43 valid numbers, plus infinite
invalid ones (17, 5, -3, 200, "abc"...). Testing them all is impossible and
pointless. These techniques tell you *which few* to actually test.

## 1. Equivalence Partitioning (EP)

**Idea:** group inputs into "classes" that the software should treat the *same way*,
then test just **one value from each class**. If one value in a class works, they
probably all do.

> **Example — age field accepting 18–60:**
>
> | Partition | Example value to test | Expected |
> | --- | --- | --- |
> | Below range (invalid) | 10 | Rejected |
> | In range (valid) | 35 | Accepted |
> | Above range (invalid) | 75 | Rejected |
> | Non-numeric (invalid) | "abc" | Rejected |
>
> Instead of testing dozens of numbers, you test **one per group** — four tests
> cover the idea.

## 2. Boundary Value Analysis (BVA)

**Idea:** bugs love **edges**. Off-by-one errors (`<` vs `<=`) cluster at the
boundaries of a range. So test *right at* and *just around* each boundary.

For the range 18–60, test the values **on and around each edge**:

```text
   ... 16  17 │ 18  19 ...        ... 59  60 │ 61  62 ...
              ▲                              ▲
        lower boundary                 upper boundary

   Test: 17 (just below), 18 (min), 19 (just above)
         59 (just below), 60 (max), 61 (just above)
```

- `17` → should be **rejected** (just outside)
- `18` → should be **accepted** (minimum valid)
- `60` → should be **accepted** (maximum valid)
- `61` → should be **rejected** (just outside)

BVA and EP are best friends: EP picks the *groups*, BVA hammers the *edges* of those
groups. Together they catch a huge share of input bugs with very few tests.

## 3. Decision Table Testing

**Idea:** when behavior depends on a **combination of conditions**, a table makes
sure you cover every combination instead of forgetting one.

> **Example — free shipping rules:** free shipping if the user is a member **OR**
> the order is over $50.
>
> | Rule | Member? | Order > $50? | Free shipping? |
> | --- | --- | --- | --- |
> | 1 | Yes | Yes | ✅ Yes |
> | 2 | Yes | No | ✅ Yes |
> | 3 | No | Yes | ✅ Yes |
> | 4 | No | No | ❌ No |
>
> Each row becomes a test case. Now you can't *accidentally* forget the "non-member,
> small order" case — it's right there as rule 4.

Decision tables shine for business rules with multiple inputs (discounts,
permissions, eligibility).

## 4. State Transition Testing

**Idea:** some features move through **states**, and the rules depend on which state
you're in. You test the valid *and* invalid transitions between states.

> **Example — a login that locks after 3 failed attempts:**

```text
   [Logged Out] --wrong password--> [1 Fail] --wrong--> [2 Fail] --wrong--> [LOCKED]
        ▲                                                                      │
        └──────────────── correct password resets ────────────────────────────┘
```

Test cases from this map:

- Logged out → 1 wrong attempt → still allowed to try
- 3 wrong attempts → account **locked**
- A correct password before lockout → resets the counter
- Trying to log in *while locked* → blocked even with the right password

State transition testing is perfect for wizards, order statuses (pending → shipped →
delivered), and anything with "modes."

## Choosing a Technique

| Situation | Technique |
| --- | --- |
| A field accepts a range or set of values | Equivalence Partitioning |
| There are min/max limits or numeric edges | Boundary Value Analysis |
| Behavior depends on combinations of conditions | Decision Table |
| The feature moves through states/steps | State Transition |

You'll often combine several. These are all **black box** techniques — you need *no*
code access to use them.

## Exercises

1. A password field requires 8–16 characters. Use **EP** to define the partitions,
   then **BVA** to list the exact lengths you'd test.
2. Build a **decision table** for: "a user gets a discount if they have a coupon AND
   the order is over $20."
3. Draw a **state transition** diagram for a traffic-light or a simple order status
   (pending → paid → shipped).
4. Why do EP and BVA work so well *together*?

Next: [Test Plans and Test Strategy](04-test-plans.md)
