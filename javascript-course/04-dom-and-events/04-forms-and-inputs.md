# Forms and User Input

Forms are how users **give you information** — their name, an email, a search term, a
choice. In this lesson you will learn to read what users type and choose, check that
it makes sense, and use it in your app. By the end, you will build a small to-do
adder. Let's collect some input! 📝

## What You'll Learn

- How to read input values with `.value`
- The `submit` event and stopping the page reload
- Basic **validation** (empty check, number check)
- **Checkboxes** and **radio buttons** with `.checked`
- **Select** dropdowns
- Building a form that adds items to a list
- Clearing inputs after submit
- A complete, working example

## Reading Input Values

A text input holds whatever the user typed. You read it with `.value`.

```html
<input id="username" placeholder="Type your name" />
<button id="show">Show name</button>

<script>
  const input = document.getElementById("username");
  const button = document.getElementById("show");

  button.addEventListener("click", () => {
    console.log(input.value); // whatever the user typed
  });
</script>
```

`.value` is always a **string** (text), even for number inputs. Keep that in mind —
it matters for validation later.

## The Submit Event

A `<form>` groups inputs together. When the user presses Enter or clicks a submit
button, the form fires a `submit` event.

But there is a catch: by default, submitting a form **reloads the page**. That
reload erases everything your JavaScript did. So almost always, the first line of
your submit handler is `event.preventDefault()` to stop the reload.

```html
<form id="myForm">
  <input id="name" placeholder="Your name" />
  <button type="submit">Send</button>
</form>

<script>
  const form = document.getElementById("myForm");
  const nameInput = document.getElementById("name");

  form.addEventListener("submit", (event) => {
    event.preventDefault(); // stop the page reload!
    console.log("You entered:", nameInput.value);
  });
</script>
```

> Listen for `submit` on the **form**, not on the button. That way it works whether
> the user clicks the button *or* presses Enter.

## Basic Validation

**Validation** means checking that the input is correct before you use it. Never
trust that the user typed something sensible. Here are two common checks.

### Empty Check

Make sure the user actually typed something. The `.trim()` method removes spaces
from the start and end, so a box with only spaces still counts as empty.

```js
form.addEventListener("submit", (event) => {
  event.preventDefault();

  const name = nameInput.value.trim();

  if (name === "") {
    console.log("Please enter your name.");
    return; // stop here; do not continue
  }

  console.log("Hello,", name);
});
```

> `return` ends the function early. We use it to stop when the input is invalid.

### Number Check

Remember: `.value` is always a string. To check or use it as a number, convert it
with `Number()`, then check with `isNaN`.

> `isNaN` means "is Not a Number". It returns `true` when the value is **not** a
> valid number.

```html
<input id="age" placeholder="Your age" />
<button id="checkAge">Check</button>

<script>
  const ageInput = document.getElementById("age");
  const checkBtn = document.getElementById("checkAge");

  checkBtn.addEventListener("click", () => {
    const age = Number(ageInput.value); // turn text into a number

    if (isNaN(age)) {
      console.log("Please enter a real number.");
      return;
    }

    if (age < 0) {
      console.log("Age cannot be negative.");
      return;
    }

    console.log("Your age is", age);
  });
</script>
```

## Checkboxes and Radio Buttons

Checkboxes and radio buttons do not use `.value` for on/off. They use `.checked`,
which is `true` (ticked) or `false` (not ticked).

### Checkboxes

A **checkbox** lets the user turn one option on or off.

```html
<label>
  <input type="checkbox" id="agree" /> I agree to the terms
</label>
<button id="submitBtn">Submit</button>

<script>
  const agree = document.getElementById("agree");
  const submitBtn = document.getElementById("submitBtn");

  submitBtn.addEventListener("click", () => {
    if (agree.checked) {
      console.log("Thanks for agreeing!");
    } else {
      console.log("You must agree first.");
    }
  });
</script>
```

### Radio Buttons

**Radio buttons** let the user pick **one** option from a group. Give them the same
`name` so only one can be selected at a time. To find the chosen one, look for the
checked radio.

```html
<label><input type="radio" name="size" value="small" /> Small</label>
<label><input type="radio" name="size" value="medium" /> Medium</label>
<label><input type="radio" name="size" value="large" /> Large</label>
<button id="orderBtn">Order</button>

<script>
  const orderBtn = document.getElementById("orderBtn");

  orderBtn.addEventListener("click", () => {
    // Find the radio in the "size" group that is checked
    const chosen = document.querySelector('input[name="size"]:checked');

    if (chosen === null) {
      console.log("Please pick a size.");
      return;
    }

    console.log("You chose:", chosen.value);
  });
</script>
```

## Select Dropdowns

A `<select>` is a dropdown menu. Read the chosen option with `.value`, just like a
text box.

```html
<select id="country">
  <option value="">-- Choose a country --</option>
  <option value="in">India</option>
  <option value="us">USA</option>
  <option value="uk">UK</option>
</select>
<button id="goBtn">Go</button>

<script>
  const country = document.getElementById("country");
  const goBtn = document.getElementById("goBtn");

  goBtn.addEventListener("click", () => {
    if (country.value === "") {
      console.log("Please choose a country.");
      return;
    }
    console.log("Country code:", country.value); // e.g. "in"
  });
</script>
```

## Building a Form That Adds to a List

Now let's combine everything: read input, validate, create an element, add it to a
list, and clear the input. This is the heart of many real apps (to-do lists, chat
boxes, comment forms).

```html
<form id="todoForm">
  <input id="todoInput" placeholder="Add a task" />
  <button type="submit">Add</button>
</form>
<ul id="todoList"></ul>

<script>
  const form = document.getElementById("todoForm");
  const input = document.getElementById("todoInput");
  const list = document.getElementById("todoList");

  form.addEventListener("submit", (event) => {
    event.preventDefault(); // stop reload

    const task = input.value.trim();

    // Validate: must not be empty
    if (task === "") {
      console.log("Please type a task.");
      return;
    }

    // Create a new list item
    const item = document.createElement("li");
    item.textContent = task;
    list.append(item);

    // Clear the input for the next task
    input.value = "";

    // Put the cursor back in the box (nice touch)
    input.focus();
  });
</script>
```

Type a task, press Enter, and watch it appear in the list while the box clears,
ready for the next one. 🎉

### Clearing Inputs After Submit

Notice these two lines:

```js
input.value = ""; // empty the text box
input.focus(); // put the cursor back so the user can keep typing
```

Setting `.value = ""` clears the box. Calling `.focus()` is a small nicety that puts
the cursor back, so the user can keep adding items without clicking again.

## A Complete Working Example

Here is a full page that ties it all together: text input, a checkbox, validation,
adding to a list, and clearing. You can save this as an `.html` file and open it in a
browser to try it.

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Shopping List</title>
    <style>
      .important {
        font-weight: bold;
        color: crimson;
      }
      .error {
        color: red;
      }
    </style>
  </head>
  <body>
    <h1>Shopping List</h1>

    <form id="shopForm">
      <input id="itemInput" placeholder="Item name" />
      <label>
        <input type="checkbox" id="importantBox" /> Important
      </label>
      <button type="submit">Add Item</button>
    </form>

    <p id="message" class="error"></p>
    <ul id="shopList"></ul>

    <script>
      const form = document.getElementById("shopForm");
      const input = document.getElementById("itemInput");
      const importantBox = document.getElementById("importantBox");
      const message = document.getElementById("message");
      const list = document.getElementById("shopList");

      form.addEventListener("submit", (event) => {
        event.preventDefault();

        const name = input.value.trim();

        // Validate
        if (name === "") {
          message.textContent = "Please enter an item name.";
          return;
        }

        message.textContent = ""; // clear any old error

        // Build the new list item
        const item = document.createElement("li");
        item.textContent = name;

        // If "Important" is checked, add a class
        if (importantBox.checked) {
          item.classList.add("important");
        }

        list.append(item);

        // Reset the form for the next item
        input.value = "";
        importantBox.checked = false;
        input.focus();
      });
    </script>
  </body>
</html>
```

This little app reads text, reads a checkbox, validates, styles items, adds them to a
list, and resets — using everything from this whole module. Great work getting here!

## Common Mistakes

- **Forgetting `event.preventDefault()`.** The page reloads and your work vanishes.
- **Treating `.value` as a number.** It is always a string; convert with `Number()`
  before doing math or number checks.
- **Not trimming input.** A box of only spaces passes a naive empty check; use
  `.trim()`.
- **Using `.value` for checkboxes.** Checkboxes and radios use `.checked`
  (`true`/`false`), not `.value`, to tell if they are selected.
- **Listening on the button instead of the form.** Listen for `submit` on the
  `<form>` so Enter works too.
- **Forgetting to clear inputs.** Reset `.value` (and `.checked`) after submit for a
  smooth experience.

## Exercises

1. Build a form with one text input that adds the typed word to a `<ul>` on submit,
   prevents the reload, and clears the box afterward.
2. Add validation: if the box is empty (after trimming), show an error message in a
   paragraph instead of adding an item.
3. Add a number input for "quantity". Convert it with `Number()` and reject anything
   that is not a positive number.
4. Add a checkbox labeled "Urgent". When checked, give the new list item a class that
   makes it red and bold.

## Key Takeaways

- Read text inputs and dropdowns with `.value` (always a string).
- Listen for `submit` on the **form** and call `event.preventDefault()` to stop the
  reload.
- **Validate** input: use `.trim()` for empty checks and `Number()` + `isNaN` for
  number checks.
- Checkboxes and radios use `.checked`; find a chosen radio with
  `input[name="..."]:checked`.
- Build list items with `createElement` + `append`, then clear inputs with
  `.value = ""` and `.checked = false`.
- Use `.focus()` to send the cursor back for a smooth, friendly form.

---

[← Back to Events and Event Listeners](03-events.md) &nbsp;&nbsp;|&nbsp;&nbsp; [Back to Course Index](../README.md)
