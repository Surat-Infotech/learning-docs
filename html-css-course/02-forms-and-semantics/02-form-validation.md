# Form Validation

When a user fills out a form, you often need to check that they entered the right kind of information. This is called **validation**. HTML can do a surprising amount of this for you, without any JavaScript. In this lesson you will learn the built-in validation tools.

## What You'll Learn

- What validation is and why we need it
- How to make a field `required`
- How to limit the length of text with `minlength` and `maxlength`
- How to set number and date limits with `min`, `max`, and `step`
- How to match a custom pattern with `pattern`
- How input types like email and url validate automatically
- A first peek at the `:valid` and `:invalid` CSS states
- Why the browser is only a helper, and the server is the real guard

## What Is Validation?

**Validation** means checking that the data a user typed makes sense before the form is sent. For example, an email field should contain something that looks like an email, and an age field should not be a negative number.

HTML has **built-in validation**. When a field fails a rule, the browser stops the form from submitting and shows a small message. The best part: you often do not have to write any code to get this.

```html
<form>
  <input type="email" required>
  <button type="submit">Submit</button>
</form>
```

If the user leaves this empty or types something that is not an email, the browser blocks the submit and explains why.

## The `required` Attribute

The **`required`** attribute means a field cannot be left empty. If it is empty, the form will not submit.

```html
<label for="name">Name</label>
<input type="text" id="name" name="name" required>
```

You do not write `required="true"`. Just the word `required` on its own is enough. Attributes like this are called **boolean attributes**: they are either present or absent.

## Limiting Text Length

Sometimes you want text to be a certain length. Two attributes help here.

- **`minlength`**: the smallest number of characters allowed.
- **`maxlength`**: the largest number of characters allowed.

```html
<label for="username">Username</label>
<input type="text" id="username" name="username"
       minlength="3" maxlength="15" required>
```

Here the username must be between 3 and 15 characters. A nice detail: `maxlength` actually stops the user from typing more, while `minlength` only complains when they try to submit.

## Limiting Numbers and Dates

For number, range, and date inputs, you can set limits with three attributes.

- **`min`**: the smallest value allowed.
- **`max`**: the largest value allowed.
- **`step`**: how big each jump is between allowed values.

```html
<label for="age">Age</label>
<input type="number" id="age" name="age" min="18" max="120">
```

This only accepts ages from 18 to 120.

The **`step`** attribute controls the spacing. A `step` of 5 means only 0, 5, 10, 15, and so on are allowed.

```html
<label for="quantity">Quantity (in fives)</label>
<input type="number" id="quantity" name="quantity"
       min="0" max="50" step="5">
```

These also work for dates. This date field only accepts days in the year 2026:

```html
<input type="date" name="event" min="2026-01-01" max="2026-12-31">
```

## Matching a Pattern with `pattern`

Sometimes you need a very specific format that the standard types do not cover. The **`pattern`** attribute lets you describe the exact shape of the allowed text using a **regular expression** (often shortened to **regex**).

A regular expression is a small language for describing text patterns. You do not need to master it now, just recognize a few pieces.

```html
<label for="zip">ZIP code (5 digits)</label>
<input type="text" id="zip" name="zip"
       pattern="[0-9]{5}"
       title="Enter exactly 5 numbers">
```

Let us break down `[0-9]{5}`:

- `[0-9]` means "any single digit from 0 to 9".
- `{5}` means "exactly five of the thing before it".

So this pattern means "exactly five digits". The **`title`** attribute gives a hint that the browser shows if the pattern is not matched, so always add a helpful `title`.

Here is another example, for a simple code of three uppercase letters:

```html
<input type="text" name="code"
       pattern="[A-Z]{3}"
       title="Three uppercase letters, like ABC">
```

## Validation from the Input Type

Some `<input>` types validate all on their own, with no extra attributes.

```html
<input type="email" name="email">  <!-- must look like an email -->
<input type="url" name="site">     <!-- must look like a web address -->
```

- **`type="email"`** checks for an `@` symbol and a sensible shape, like `name@example.com`.
- **`type="url"`** checks for something that looks like a link, like `https://example.com`.

These checks are basic. They confirm the *shape* is reasonable, not that the email or website truly exists. Still, they catch many simple typing mistakes.

## A Quick Look at `:valid` and `:invalid`

Browsers know whether each field currently passes its rules. With CSS, you can react to this using the **`:valid`** and **`:invalid`** states. We will cover CSS properly in later lessons, but here is a small preview so you know it exists.

```css
input:invalid {
  border: 2px solid red;
}

input:valid {
  border: 2px solid green;
}
```

This gives a field a red border while its contents break a rule, and a green border once they pass. It is a friendly way to guide the user as they type. **A "state" like `:invalid` is a condition the element is in right now**, and CSS can style based on it.

## Custom Messages

The default messages the browser shows ("Please fill out this field") are fine, but a little generic. You can change them, but doing so nicely requires JavaScript, which comes later in your learning. For now, the most useful tool you have is the **`title`** attribute on a `pattern` field, which adds your own hint to the message.

```html
<input type="text" name="phone"
       pattern="[0-9]{10}"
       title="Enter your 10-digit phone number with no spaces">
```

## Turning Validation Off with `novalidate`

Sometimes, while building or testing, you want the form to submit even if the rules are not met. The **`novalidate`** attribute on the `<form>` turns off all built-in validation.

```html
<form action="/save" method="post" novalidate>
  <input type="email" name="email" required>
  <button type="submit">Submit</button>
</form>
```

With `novalidate`, the form submits no matter what. Use this carefully. It is mostly handy during development.

## Client-Side Is a Convenience, Server-Side Is the Real Safety

This is the most important idea in the whole lesson.

All the validation above happens in the user's browser. This is called **client-side validation** (the "client" is the user's browser). It is fast and friendly. It catches mistakes instantly without waiting for the server.

But here is the catch: a clever or careless user can turn it off, change the page, or send data straight to your server without using the form at all. So client-side validation is a **convenience**, not a guarantee.

The real protection happens on the **server** (the computer that receives and stores the data). This is **server-side validation**. The server must check the data again, because it can never fully trust what arrives.

Think of it like a bouncer at a club. The sign on the door ("must be 18") is client-side validation, polite and helpful. The bouncer actually checking IDs at the entrance is server-side validation, the real rule that cannot be skipped. You learn server-side validation when you study back-end programming. For now, just remember:

> Client-side validation makes the form pleasant. Server-side validation keeps it safe. You need both.

## Common Mistakes

- Writing `required="true"`. Just `required` on its own is correct.
- Relying only on client-side validation and trusting that data is clean.
- Forgetting a `title` on a `pattern` field, leaving the user confused about the format.
- Using `pattern` on a `type="number"` field (patterns are for text inputs).
- Setting a `min` higher than `max`, so no value can ever be valid.
- Leaving `novalidate` on a finished, live form by accident.

## Exercises

1. Make a form with a required name field (text) and a required email field. Try submitting it empty and read the browser's messages.
2. Add a username field that must be between 4 and 12 characters using `minlength` and `maxlength`.
3. Create a number field for "tickets" that only allows 1 to 6 in steps of 1, then change the `step` to 2 and see what happens.
4. Build a text field with a `pattern` that requires exactly 4 digits (a PIN). Add a helpful `title`.
5. Add the `:invalid` and `:valid` CSS rules from this lesson to a small form and watch the border color change as you type a valid email.

## Key Takeaways

- HTML can validate forms for you, with no JavaScript needed.
- `required` makes a field mandatory; it is a boolean attribute.
- `minlength`/`maxlength` limit text; `min`/`max`/`step` limit numbers and dates.
- `pattern` matches a custom format using a regular expression; always add a `title`.
- `type="email"` and `type="url"` validate their shape automatically.
- `:valid` and `:invalid` let CSS respond to a field's current state.
- `novalidate` turns validation off, mainly for testing.
- Client-side validation is a convenience; server-side validation is the real safety net. Use both.

---

[← Back to Course Index](../README.md) | [Next: Semantic HTML →](03-semantic-html.md)
