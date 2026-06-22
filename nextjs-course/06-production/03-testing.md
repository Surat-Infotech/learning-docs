# Testing Your Next.js App

Testing means writing code that checks your other code still works. It sounds
like extra effort, but it is really a safety net: when you change something next
month, your tests shout if you accidentally broke a feature.

Think of tests like a smoke detector in your house. You install it once, and from
then on it quietly watches. The day something catches fire, it warns you before
the whole place burns down. You will not test everything — just the parts where a
fire would hurt the most.

## What You'll Learn

- The "testing pyramid" and what each layer is for
- How to write a small component test with Vitest (or Jest) + React Testing Library
- How to write a simple end-to-end test with Playwright
- Why Server Components are awkward to test directly, and what to do instead
- The habit of testing your **logic and data functions** on their own
- How to decide what is actually worth testing

## The Testing Pyramid

A common way to picture testing is a pyramid. The bottom layer is the biggest
because you should have the most of those tests; the top is smallest.

```text
        /\
       /  \    End-to-end (E2E): few, slow, very realistic
      /----\
     /      \  Component / integration: more, medium speed
    /--------\
   /          \ Unit: many, fast, tiny pieces
  /------------\
```

- **Unit tests** check one small function in isolation. They are fast, so you can
  have hundreds.
- **Component tests** render a component and check what the user would see.
- **End-to-end (E2E) tests** drive a real browser through a whole flow (click,
  type, submit). They are the most realistic but the slowest, so you keep few.

## Unit and Component Testing with Vitest

**Vitest** is a fast test runner (Jest is an older, very similar one — pick
either). **React Testing Library** helps you render a component and inspect it
the way a user would, by looking for visible text rather than internal details.

```bash
# Install the testing tools
npm install -D vitest @testing-library/react @testing-library/jest-dom jsdom
```

```ts
// vitest.config.ts
import { defineConfig } from "vitest/config";

export default defineConfig({
  test: {
    environment: "jsdom", // a fake browser so components can render
    globals: true, // lets us use "test" and "expect" without importing
  },
});
```

### A simple unit test

Start with a plain function — these are the easiest and most valuable to test.

```ts
// lib/price.ts
export function formatPrice(cents: number): string {
  // Turn 1999 into "$19.99"
  return `$${(cents / 100).toFixed(2)}`;
}
```

```ts
// lib/price.test.ts
import { test, expect } from "vitest";
import { formatPrice } from "./price";

test("formats cents as dollars", () => {
  // We call the function and check we get the value we expect.
  expect(formatPrice(1999)).toBe("$19.99");
  expect(formatPrice(0)).toBe("$0.00");
});
```

```bash
# Run all tests once
npx vitest run
```

### A simple component test

Now render a component and check the screen.

```tsx
// app/components/Greeting.tsx
export default function Greeting({ name }: { name: string }) {
  return <h1>Hello, {name}!</h1>;
}
```

```tsx
// app/components/Greeting.test.tsx
import { render, screen } from "@testing-library/react";
import { test, expect } from "vitest";
import Greeting from "./Greeting";

test("shows the name", () => {
  render(<Greeting name="Sam" />); // render the component
  // Look for the text a real user would see.
  expect(screen.getByText("Hello, Sam!")).toBeDefined();
});
```

### Testing interaction

For a component with a button, simulate a click and check the result.

```tsx
// app/components/Counter.test.tsx
import { render, screen, fireEvent } from "@testing-library/react";
import { test, expect } from "vitest";
import Counter from "./Counter";

test("increments when clicked", () => {
  render(<Counter />);
  const button = screen.getByRole("button"); // find the button
  fireEvent.click(button); // pretend the user clicks it
  expect(screen.getByText("Clicked 1 times")).toBeDefined();
});
```

## End-to-End Testing with Playwright

**Playwright** opens a real browser, visits your running app, and acts like a
user. This is the closest you get to "does the whole thing actually work?"
(Cypress is a popular alternative that works in a similar way.)

```bash
# Install Playwright and its browsers
npm install -D @playwright/test
npx playwright install
```

```ts
// e2e/home.spec.ts
import { test, expect } from "@playwright/test";

test("home page shows the title", async ({ page }) => {
  await page.goto("http://localhost:3000"); // visit the running app
  // Check a heading is visible on the page.
  await expect(page.getByRole("heading", { name: "My blog" })).toBeVisible();
});

test("user can navigate to about", async ({ page }) => {
  await page.goto("http://localhost:3000");
  await page.getByRole("link", { name: "About" }).click(); // click a link
  await expect(page).toHaveURL(/.*about/); // we ended up on /about
});
```

```bash
# Start your app first, then run the E2E tests
npm run dev
npx playwright test
```

E2E tests are powerful but slow and a little fragile, so test only your most
important journeys: sign up, log in, add to cart, check out.

## Testing Server Components (The Caveat)

Async Server Components are tricky to render inside a normal test runner because
they run on the server and can talk to a database. The current testing tools do
not render them as smoothly as Client Components.

The practical advice: **do not fight it.** Instead, pull the logic out of the
component into plain functions, and test those functions directly.

```tsx
// app/products/page.tsx  (Server Component)
import { getProducts } from "@/lib/products";

export default async function ProductsPage() {
  const products = await getProducts(); // the logic lives in lib/
  return <ul>{products.map((p) => <li key={p.id}>{p.name}</li>)}</ul>;
}
```

```ts
// lib/products.test.ts
import { test, expect } from "vitest";
import { getProducts } from "./products";

test("getProducts returns a list", async () => {
  const products = await getProducts(); // test the data function directly
  expect(Array.isArray(products)).toBe(true);
});
```

Then let your few Playwright E2E tests confirm the full page renders in a real
browser. Between unit-testing the data functions and E2E-testing the page, you
cover the Server Component without wrestling the renderer.

## What Is Worth Testing?

You do not need 100% coverage. Spend your effort where a bug would hurt:

- **Business logic:** price calculations, validation, permissions.
- **Tricky functions:** anything with branches, edge cases, or money involved.
- **Critical user journeys:** the few flows that, if broken, lose you users.

Skip testing trivial things (a component that only prints a prop) and never test
the framework itself — Next.js already tests Next.js.

## Common Mistakes

- **Trying to render an async Server Component in a unit test.** Extract the logic
  into a function and test that, then cover the page with E2E.
- **Testing implementation details** like internal state names. Test what the
  user sees and does instead.
- **Writing only E2E tests.** They are slow and flaky in large numbers; lean on
  fast unit tests for most coverage.
- **Forgetting to start the app before Playwright runs.** E2E tests need a live
  server at the URL they visit.
- **Chasing 100% coverage.** Aim for confidence on the parts that matter, not a
  vanity number.

## Exercises

1. Install Vitest and write a unit test for a small function (for example, a
   `formatPrice` or `slugify` helper). Run it and watch it pass.
2. Write a component test for a `Greeting` component that checks the visible text.
3. Add a `Counter` component test that clicks the button and asserts the new
   count appears.
4. Set up Playwright and write one E2E test that visits your home page and checks
   a heading is visible.
5. Take a Server Component that fetches data, move the fetch into a `lib/`
   function, and write a unit test for that function instead of the component.

## Key Takeaways

- The testing pyramid: many fast **unit** tests, some **component** tests, few
  realistic **E2E** tests.
- Use Vitest (or Jest) with React Testing Library for unit and component tests;
  check what the user sees.
- Use Playwright (or Cypress) for end-to-end tests of your most important flows.
- Server Components are awkward to render in tests, so test their **logic and data
  functions** directly and cover the page with E2E.
- Test where bugs would hurt most; do not chase 100% coverage.

---

[← Previous: Performance and Optimization](02-performance-optimization.md) | [Back to Course Index](../README.md) | [Next: Deployment →](04-deployment.md)
