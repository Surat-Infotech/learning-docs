# Objects

Arrays are great for ordered lists. But sometimes you want to describe **one
thing** with many details. For that, you use an **object**.

An object stores data as **key/value pairs**. Think of a contact card: it has a
name, a phone number, and an email. Each label (key) points to a value. Objects
let you model real-world things in code.

## What You'll Learn

- What an object is and why it is useful
- How to create objects
- Two ways to read properties: dot and bracket
- Adding, changing, and deleting properties
- Methods (functions that live inside an object)
- Shorthand properties (a handy shortcut)
- Nested objects (objects inside objects)
- Looping with `for...in`, `Object.keys`, `Object.values`, `Object.entries`
- Checking if a property exists
- Arrays of objects (a very common pattern)

## What Is an Object?

An object is a collection of **key/value pairs**. The key is a label (a string).
The value is the data. Together one pair is called a **property**.

```js
const person = {
  name: "Sam",
  age: 30,
  isStudent: false,
};

console.log(person); // => { name: "Sam", age: 30, isStudent: false }
```

Here `name`, `age`, and `isStudent` are keys. `"Sam"`, `30`, and `false` are
their values. Notice we use curly braces `{ }` and a colon `:` between key and
value, with commas between pairs.

## Creating Objects

The normal way is the **object literal** with `{ }`.

```js
const car = {
  brand: "Toyota",
  year: 2020,
};

const empty = {}; // an empty object, ready to fill later
```

## Dot vs Bracket Access

You read a value by its key. There are two ways.

### Dot notation (most common)

```js
const person = { name: "Sam", age: 30 };

console.log(person.name); // => "Sam"
console.log(person.age);  // => 30
```

### Bracket notation

```js
const person = { name: "Sam", age: 30 };

console.log(person["name"]); // => "Sam"
```

Dot notation is shorter and you will use it most. So when do you need brackets?

1. When the key has a space or special character:

```js
const data = { "first name": "Sam" };
console.log(data["first name"]); // => "Sam"
// console.log(data.first name); // this is a syntax error
```

2. When the key is stored in a **variable**:

```js
const person = { name: "Sam", age: 30 };
const key = "age";

console.log(person[key]); // => 30   (uses the value of key)
console.log(person.key);  // => undefined (looks for a key literally named "key")
```

## Adding, Changing, and Deleting Properties

You can change objects after you create them. Even with `const`, you can change
the **inside** of the object.

### Add or change

Assign to a key. If the key exists, it changes. If not, it is added.

```js
const person = { name: "Sam" };

person.age = 30;        // add a new property
person.name = "Sammy";  // change an existing one

console.log(person); // => { name: "Sammy", age: 30 }
```

### Delete

Use the `delete` keyword to remove a property.

```js
const person = { name: "Sam", age: 30 };

delete person.age;
console.log(person); // => { name: "Sam" }
```

## Methods (Functions Inside Objects)

A value can be a function. A function stored in an object is called a **method**.
It usually does something related to that object.

```js
const dog = {
  name: "Rex",
  bark: function () {
    return "Woof!";
  },
};

console.log(dog.bark()); // => "Woof!"  (note the parentheses to call it)
```

Inside a method, the keyword `this` refers to the object itself. So you can read
the object's own properties.

```js
const person = {
  name: "Sam",
  greet: function () {
    return `Hi, I am ${this.name}`;
  },
};

console.log(person.greet()); // => "Hi, I am Sam"
```

There is a shorter way to write methods:

```js
const person = {
  name: "Sam",
  greet() {              // shorthand method (no "function" keyword)
    return `Hi, I am ${this.name}`;
  },
};

console.log(person.greet()); // => "Hi, I am Sam"
```

## Shorthand Properties

When a variable name matches the key name, you can write it once.

```js
const name = "Sam";
const age = 30;

// long way:
const person1 = { name: name, age: age };

// shorthand (same result):
const person2 = { name, age };

console.log(person2); // => { name: "Sam", age: 30 }
```

This is a common shortcut you will see everywhere.

## Nested Objects

A value can be another object. This is a **nested object**. It models things that
have parts.

```js
const user = {
  name: "Sam",
  address: {
    city: "Pune",
    zip: "411001",
  },
};

console.log(user.address.city); // => "Pune"
```

Read `user.address.city` left to right: first `user`, then its `address`, then
the `city` inside that.

You can mix objects and arrays freely:

```js
const team = {
  title: "Builders",
  members: ["Sam", "Alex"],
};

console.log(team.members[0]); // => "Sam"
```

## Looping Over Objects

### for...in

`for...in` loops over the **keys** of an object.

```js
const person = { name: "Sam", age: 30 };

for (const key in person) {
  console.log(key, person[key]); // use brackets because key is a variable
}
// => name "Sam"
// => age 30
```

### Object.keys, Object.values, Object.entries

These turn an object into an array, which is often easier to work with.

```js
const person = { name: "Sam", age: 30 };

console.log(Object.keys(person));   // => ["name", "age"]
console.log(Object.values(person)); // => ["Sam", 30]
console.log(Object.entries(person));
// => [["name", "Sam"], ["age", 30]]   (an array of [key, value] pairs)
```

Once you have an array, you can use array methods like `forEach`:

```js
const prices = { apple: 2, banana: 1 };

Object.entries(prices).forEach(([item, price]) => {
  console.log(`${item} costs $${price}`);
});
// => "apple costs $2"
// => "banana costs $1"
```

## Checking If a Property Exists

Sometimes you need to know whether a key is present.

The clearest way is the `in` operator:

```js
const person = { name: "Sam" };

console.log("name" in person); // => true
console.log("age" in person);  // => false
```

You can also check the value, but be careful: a key might exist with a value of
`undefined`. The `in` operator avoids that confusion.

```js
const person = { name: "Sam" };

console.log(person.age === undefined); // => true (but does the key exist? unclear)
console.log("age" in person);          // => false (clearly the key is missing)
```

## Arrays of Objects (Very Common)

A super common pattern is an **array of objects**: a list where each item is an
object. This is how you store many records, like users or products.

```js
const users = [
  { name: "Sam", age: 30 },
  { name: "Alex", age: 25 },
  { name: "Jo", age: 40 },
];

console.log(users[0].name); // => "Sam"
```

Now you can use array methods on the list:

```js
// get all names
const names = users.map((user) => user.name);
console.log(names); // => ["Sam", "Alex", "Jo"]

// keep only users 30 or older
const older = users.filter((user) => user.age >= 30);
console.log(older);
// => [{ name: "Sam", age: 30 }, { name: "Jo", age: 40 }]

// find one user by name
const found = users.find((user) => user.name === "Alex");
console.log(found); // => { name: "Alex", age: 25 }
```

This combination of arrays and objects is the backbone of most real apps.

## Common Mistakes

- **Using dot with a variable key.** `person.key` looks for a literal `"key"`.
  Use `person[key]` to use the variable's value.
- **Forgetting parentheses on a method.** `dog.bark` gives the function itself;
  `dog.bark()` runs it.
- **Confusing objects and arrays.** Objects use named keys `{ }`; arrays use
  numbered indexes `[ ]`.
- **Expecting `for...in` to give values.** It gives **keys**. Use `person[key]`
  to get the value.
- **Comparing objects with `===`.** Two objects are equal only if they are the
  exact same object, not if they look alike.

## Exercises

1. Create an object `book` with keys `title`, `author`, and `year`. Print the
   title with dot notation and the year with bracket notation. Then add a `genre`
   property and delete the `year` property.
2. Add a method `describe()` to the `book` object that returns a sentence like
   `"Title by Author"` using `this`.
3. Loop over the `book` object with `for...in` and print each key and value. Then
   do the same using `Object.entries` and `forEach`.
4. Create an array of three `product` objects, each with `name` and `price`. Use
   `map` to get a list of names, and `filter` to keep products under $20.

## Key Takeaways

- An **object** stores data as **key/value pairs** using `{ }`.
- Read values with **dot** (`obj.key`) or **brackets** (`obj["key"]`). Use
  brackets for special keys or variable keys.
- Add/change with assignment; remove with `delete`.
- A **method** is a function inside an object; `this` refers to that object.
- **Shorthand** lets you write `{ name }` instead of `{ name: name }`.
- Objects can nest inside objects and mix with arrays.
- Loop with `for...in`, or use `Object.keys/values/entries`.
- Use `"key" in obj` to check if a property exists.
- **Arrays of objects** are everywhere; combine them with `map`, `filter`, and
  `find`.

---

[← Back to Course Index](../README.md) | [Next: Strings and Template Literals →](04-strings.md)
