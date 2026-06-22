# Forms and Inputs

Forms are how websites collect information from people. Every time you log in, search for something, or sign up for a newsletter, you are using a form. In this lesson you will learn the building blocks of forms in HTML.

## What You'll Learn

- What a `<form>` is and what `action` and `method` do
- Why labels matter and how `for` and `id` connect them to inputs
- The many types of `<input>` you can use
- How `name` and `value` carry data
- How to use `<textarea>`, `<select>`, and `<button>`
- How to group related fields with `<fieldset>` and `<legend>`
- How to build a complete, working form

## The `<form>` Element

A **form** is a container. It wraps all the fields where a user types or chooses things. When the user is done, they "submit" the form, and the browser sends the collected data somewhere.

```html
<form action="/signup" method="post">
  <!-- inputs go here -->
</form>
```

Think of a form like an envelope. The fields inside are the letter, and the form decides where to mail it and how.

### The `action` Attribute

The **`action`** attribute is the address where the data is sent. It is usually a URL on the server.

```html
<form action="/login">
```

If you leave `action` out, the data is sent back to the same page.

### The `method` Attribute: GET vs POST

The **`method`** attribute tells the browser *how* to send the data. There are two common choices.

- **GET**: The data is added to the end of the URL, like `/search?q=cats`. Good for searches and things you might want to bookmark. The data is visible in the address bar.
- **POST**: The data is sent quietly in the background, not in the URL. Good for passwords, sign-ups, and anything private or that changes data on the server.

```html
<form action="/search" method="get">  <!-- shows up in the URL -->
<form action="/login" method="post">  <!-- hidden from the URL -->
```

A simple rule: use **GET** to *fetch* things, and **POST** to *send* or *change* things.

## Labels and Why They Matter

A **`<label>`** is the text that describes a field, like "Email" or "Password". Labels are not just decoration. They make forms easier to use for everyone.

When a label is connected to an input:

- Clicking the label puts the cursor in the field (a bigger, easier target).
- Screen readers (tools that read pages aloud for people who cannot see) announce the label when the user reaches the field.

### Connecting `for` and `id`

You connect a label to an input by matching the label's **`for`** attribute to the input's **`id`** attribute. They must be the same text.

```html
<label for="email">Email address</label>
<input type="email" id="email" name="email">
```

Here `for="email"` points to `id="email"`. They are now a pair. Think of `id` as a name tag on the input, and `for` as the label calling out that name.

## The `<input>` Element

The **`<input>`** element is the most common form field. One little tag can become many different things, depending on its **`type`**.

```html
<input type="text">
```

`<input>` has no closing tag. Here are the types you will use most.

### Text-like Inputs

```html
<input type="text" name="username">       <!-- any short text -->
<input type="email" name="email">          <!-- expects an email address -->
<input type="password" name="pass">        <!-- hides what you type -->
<input type="number" name="age">           <!-- numbers only, with up/down arrows -->
<input type="tel" name="phone">            <!-- phone numbers -->
<input type="url" name="website">          <!-- web addresses -->
<input type="date" name="birthday">        <!-- shows a date picker -->
```

The browser gives some of these helpful behaviors. For example, `type="email"` will check that the value looks like an email, and on phones `type="number"` brings up a number keypad.

### Choice Inputs: Checkbox and Radio

A **checkbox** lets the user turn one option on or off.

```html
<label>
  <input type="checkbox" name="subscribe" value="yes">
  Send me the newsletter
</label>
```

**Radio buttons** let the user pick *one* option out of several. To make radios work as a group, give them all the **same `name`**.

```html
<p>Choose a size:</p>
<label><input type="radio" name="size" value="small"> Small</label>
<label><input type="radio" name="size" value="medium"> Medium</label>
<label><input type="radio" name="size" value="large"> Large</label>
```

Because they share `name="size"`, choosing one automatically unchecks the others. That is the difference: checkboxes are independent, radios are "pick one".

### Other Useful Inputs

```html
<input type="file" name="photo">           <!-- lets the user upload a file -->
<input type="range" name="volume" min="0" max="100"> <!-- a slider -->
<input type="color" name="favColor">        <!-- a color picker -->
<input type="hidden" name="userId" value="42"> <!-- invisible to the user -->
```

A **hidden** input is not shown on the page at all. It still sends its value when the form is submitted. It is handy for carrying along extra information the user does not need to see.

## The `name` and `value` Attributes

These two attributes are how data actually travels.

- **`name`** is the label on the box of data. The server uses it to know what each piece of data is.
- **`value`** is what is inside the box. For text fields the user types it. For checkboxes, radios, and hidden fields, you set it yourself.

```html
<input type="text" name="city" value="London">
```

If you submit this, the server receives `city = London`. **A field without a `name` is not sent at all**, so always give your fields a `name`.

## The `<textarea>` Element

A **`<textarea>`** is a bigger text box for longer writing, like a message or a comment. Unlike `<input>`, it has an opening and closing tag, and any text between them is the starting value.

```html
<label for="message">Your message</label>
<textarea id="message" name="message" rows="4" cols="40"></textarea>
```

The `rows` and `cols` attributes set the starting size (rows of text and characters wide).

## The `<select>` Element

A **`<select>`** is a dropdown menu. Each choice inside is an **`<option>`**.

```html
<label for="country">Country</label>
<select id="country" name="country">
  <option value="us">United States</option>
  <option value="ca">Canada</option>
  <option value="mx">Mexico</option>
</select>
```

The `value` of the chosen option is what gets sent. The text between the tags is what the user sees.

### Grouping Options with `<optgroup>`

If you have many options, you can group them with **`<optgroup>`** and a `label`.

```html
<select name="food">
  <optgroup label="Fruit">
    <option value="apple">Apple</option>
    <option value="banana">Banana</option>
  </optgroup>
  <optgroup label="Vegetables">
    <option value="carrot">Carrot</option>
    <option value="pea">Pea</option>
  </optgroup>
</select>
```

The group labels are headings only. They cannot be selected.

## The `<button>` Element

A **`<button>`** is something the user clicks. It has three `type` values.

```html
<button type="submit">Send</button>   <!-- submits the form -->
<button type="reset">Clear</button>   <!-- empties all fields -->
<button type="button">Help</button>   <!-- does nothing on its own -->
```

- **submit**: sends the form. This is the default if you do not write a type.
- **reset**: clears every field back to its starting value.
- **button**: a plain button that does nothing until you add JavaScript later.

## Placeholders

The **`placeholder`** attribute shows faint hint text inside a field before the user types.

```html
<input type="text" name="search" placeholder="Search for a recipe...">
```

A placeholder is a *hint*, not a label. The hint disappears as soon as the user types, so it cannot replace a real `<label>`. Always use both.

## Grouping Fields with `<fieldset>` and `<legend>`

A **`<fieldset>`** draws a box around a set of related fields. A **`<legend>`** gives that box a title. This is especially helpful for groups of radio buttons.

```html
<fieldset>
  <legend>Shipping speed</legend>
  <label><input type="radio" name="speed" value="standard"> Standard</label>
  <label><input type="radio" name="speed" value="express"> Express</label>
</fieldset>
```

This groups the choices visually and tells screen readers that these fields belong together.

## A Complete Example Form

Here is everything put together into one sign-up form.

```html
<form action="/signup" method="post">
  <fieldset>
    <legend>Create your account</legend>

    <p>
      <label for="fullname">Full name</label>
      <input type="text" id="fullname" name="fullname" placeholder="Jane Doe">
    </p>

    <p>
      <label for="email">Email</label>
      <input type="email" id="email" name="email">
    </p>

    <p>
      <label for="password">Password</label>
      <input type="password" id="password" name="password">
    </p>

    <p>
      <label for="country">Country</label>
      <select id="country" name="country">
        <option value="us">United States</option>
        <option value="ca">Canada</option>
        <option value="other">Other</option>
      </select>
    </p>
  </fieldset>

  <fieldset>
    <legend>Preferences</legend>
    <label>
      <input type="checkbox" name="newsletter" value="yes">
      Email me news and updates
    </label>
  </fieldset>

  <button type="submit">Sign up</button>
  <button type="reset">Clear form</button>
</form>
```

## Common Mistakes

- Forgetting the `name` attribute, so the field's data is never sent.
- Mismatching `for` and `id`, which breaks the label-input connection.
- Using a `placeholder` instead of a `<label>`. The hint vanishes when typing starts.
- Giving radio buttons different `name` values, so the user can pick more than one.
- Putting a `type="submit"` button outside the `<form>`, so it cannot submit anything.
- Forgetting that `<input>` has no closing tag, while `<textarea>` and `<select>` do.

## Exercises

1. Build a contact form with three fields: name (text), email (email), and a message (`<textarea>`). Connect each field to a label using `for` and `id`. Add a submit button.
2. Create a group of three radio buttons for "favorite season". Wrap them in a `<fieldset>` with a `<legend>` that says "Pick a season". Make sure they share the same `name`.
3. Make a small order form using a `<select>` dropdown with at least four options grouped into two `<optgroup>` sections. Add a `type="number"` input for quantity.
4. Add a checkbox to your contact form that says "Send me a copy". Give it a `name` and a `value`.
5. Change a form's `method` from `post` to `get` and explain in one sentence where the data would appear differently.

## Key Takeaways

- A `<form>` wraps fields; `action` is where data goes and `method` is how it travels.
- Use **GET** to fetch and **POST** to send private or changing data.
- Always pair a `<label>` with its input using matching `for` and `id`.
- `<input>` becomes many fields depending on its `type`.
- `name` identifies the data and `value` holds it; no `name` means no data sent.
- Use `<textarea>` for long text, `<select>` for dropdowns, and `<button type="submit">` to send.
- Group related fields with `<fieldset>` and `<legend>`.

---

[← Back to Course Index](../README.md) | [Next: Form Validation →](02-form-validation.md)
