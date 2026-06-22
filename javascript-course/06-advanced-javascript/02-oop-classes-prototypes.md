# Objects, Classes, and Prototypes (OOP)

So far you have made objects by hand, one at a time. But what if you need 100
similar objects — say, 100 users or 100 enemies in a game? Typing them all out
would be painful. **Object-Oriented Programming (OOP)** gives you a better way:
you build a *blueprint* once, then stamp out as many objects as you like.

This lesson goes slowly because OOP introduces a few new words. Take your time,
type the examples, and the ideas will click.

## What You'll Learn

- What OOP means in plain words
- Objects as **blueprints**
- The **prototype chain** (objects inheriting from objects)
- Constructor functions (the old way), briefly
- Modern **`class`** syntax: constructor, methods, properties
- The **`new`** keyword
- **Inheritance** with `extends` and `super`
- **Getters and setters**
- **Static** methods
- How **`this`** behaves inside classes

---

## What Is OOP in Simple Words?

OOP is a way of organizing code around **objects**. An object bundles together:

- **Data** (called *properties*) — what it *is*. Example: a dog has a `name`.
- **Behavior** (called *methods*) — what it *does*. Example: a dog can `bark()`.

Instead of having loose variables and functions scattered everywhere, you keep
related data and behavior together inside one object. That makes code easier to
think about, because it mirrors how we see the real world: things that have
properties and can do actions.

---

## Objects as Blueprints

Think of a **cookie cutter**. The cutter is the blueprint. Each cookie you cut
out is a separate object made from that same shape. In OOP:

- The **class** is the cookie cutter (the blueprint).
- Each **object** made from it is called an **instance**.

You design the cutter once. Then you make many cookies, each with its own
details (one has sprinkles, one has chocolate chips), but all sharing the same
basic shape.

---

## The Prototype Chain (Explained Gently)

Here is JavaScript's secret: objects can **inherit** from other objects. When you
ask an object for a property and it does not have one, JavaScript looks at its
**prototype** — a kind of "fallback" object. If the prototype does not have it
either, JavaScript checks the prototype's prototype, and so on. This trail is the
**prototype chain**.

Think of it like asking your family for help. You ask your parent. If they cannot
help, they ask *their* parent. The question travels up the chain until someone
answers or you reach the top (where there is no more chain, which is `null`).

```js
const animal = {
  eats: true,
};

// `rabbit` uses `animal` as its prototype (its fallback).
const rabbit = Object.create(animal);
rabbit.hops = true;

console.log(rabbit.hops); // => true  (found on rabbit itself)
console.log(rabbit.eats); // => true  (not on rabbit, found on animal)
```

`rabbit` does not have `eats`, but its prototype `animal` does, so JavaScript
finds it up the chain. You usually do not write `Object.create` by hand —
classes handle prototypes for you. But now you know what is happening underneath.

---

## Constructor Functions (The Old Way), Briefly

Before the `class` keyword, people made blueprints using regular functions called
**constructor functions**. You may see this in older code:

```js
function Dog(name) {
  this.name = name;
}
Dog.prototype.bark = function () {
  return this.name + " says Woof!";
};

const d = new Dog("Rex");
console.log(d.bark()); // => Rex says Woof!
```

This works, but it is a little clunky. Modern JavaScript gives us a cleaner way
that does the same thing under the hood: **classes**.

---

## Modern Class Syntax

A `class` is the blueprint. Here is a simple one:

```js
class Dog {
  // The constructor runs when you create a new object.
  constructor(name, breed) {
    this.name = name;   // a property
    this.breed = breed; // another property
  }

  // A method (behavior).
  bark() {
    return this.name + " says Woof!";
  }
}
```

- `constructor` is a special method that runs once, when the object is created.
- `this` refers to **the new object being built**.
- `bark()` is a method shared by every dog made from this class.

---

## The `new` Keyword

To make an object from a class, use `new`:

```js
const rex = new Dog("Rex", "Labrador");

console.log(rex.name);   // => Rex
console.log(rex.breed);  // => Labrador
console.log(rex.bark()); // => Rex says Woof!

const bella = new Dog("Bella", "Poodle");
console.log(bella.bark()); // => Bella says Woof!
```

`new` does four things for you, quietly:

1. Creates a fresh empty object.
2. Links it to the class's prototype (so it gets the methods).
3. Runs the `constructor` with `this` set to that new object.
4. Returns the finished object.

`rex` and `bella` are two **separate instances**. Each has its own `name`, but
both share the same `bark` method through the prototype.

---

## Inheritance With `extends` and `super`

Often a class is a more specific version of another class. A `Dog` *is an*
`Animal`. We can share the common parts using `extends`.

```js
// The parent (base) class — general stuff.
class Animal {
  constructor(name) {
    this.name = name;
  }

  eat() {
    return this.name + " is eating.";
  }

  describe() {
    return this.name + " is an animal.";
  }
}

// The child class — Dog is a kind of Animal.
class Dog extends Animal {
  constructor(name, breed) {
    super(name);     // call the parent constructor first
    this.breed = breed;
  }

  bark() {
    return this.name + " says Woof!";
  }
}
```

Now `Dog` automatically gets everything from `Animal`:

```js
const rex = new Dog("Rex", "Labrador");

console.log(rex.eat());  // => Rex is eating.   (from Animal)
console.log(rex.bark()); // => Rex says Woof!   (from Dog)
console.log(rex.breed);  // => Labrador
```

Two important pieces:

- **`extends`** says "Dog inherits from Animal."
- **`super(name)`** calls the parent's constructor. You must call `super` before
  using `this` in a child constructor.

### Overriding a Method

A child can replace a parent method by defining one with the same name. You can
even call the parent's version using `super.methodName()`:

```js
class Dog extends Animal {
  constructor(name, breed) {
    super(name);
    this.breed = breed;
  }

  // Replace the parent's describe()
  describe() {
    return super.describe() + " More exactly, a " + this.breed + " dog.";
  }
}

const rex = new Dog("Rex", "Labrador");
console.log(rex.describe());
// => Rex is an animal. More exactly, a Labrador dog.
```

---

## Getters and Setters

A **getter** lets you read a computed value as if it were a plain property. A
**setter** lets you run code when someone assigns a value. They make objects feel
natural to use.

```js
class Circle {
  constructor(radius) {
    this.radius = radius;
  }

  // getter: read like a property, no ()
  get area() {
    return Math.PI * this.radius * this.radius;
  }

  // setter: runs when you assign to .diameter
  set diameter(value) {
    this.radius = value / 2;
  }
}

const c = new Circle(5);
console.log(c.area); // => 78.539... (no parentheses!)

c.diameter = 20;     // triggers the setter
console.log(c.radius); // => 10
```

Notice `c.area` has **no parentheses** — that is the getter working. Setters are
great for validation, like rejecting negative numbers.

---

## Static Methods

A **static** method belongs to the class itself, not to individual objects. Use
it for helper functions related to the class but that do not need a specific
instance.

```js
class MathHelper {
  static double(n) {
    return n * 2;
  }
}

console.log(MathHelper.double(8)); // => 16

// You do NOT call it on an instance:
const m = new MathHelper();
// m.double(8); // => TypeError: m.double is not a function
```

A common real example is a factory method that creates instances:

```js
class User {
  constructor(name) {
    this.name = name;
  }

  static fromUppercase(name) {
    return new User(name.toUpperCase());
  }
}

const u = User.fromUppercase("amy");
console.log(u.name); // => AMY
```

---

## `this` Inside Classes

Inside a method, **`this` refers to the specific object the method was called
on**. When you write `rex.bark()`, `this` is `rex`. When you write
`bella.bark()`, `this` is `bella`. That is how the same method can give different
results for different objects.

```js
class Counter {
  constructor() {
    this.count = 0;
  }
  increment() {
    this.count++; // `this` is whichever counter you called it on
    return this.count;
  }
}

const a = new Counter();
const b = new Counter();
console.log(a.increment()); // => 1
console.log(a.increment()); // => 2
console.log(b.increment()); // => 1  (b has its own count)
```

> Heads up: if you pass a method around on its own (like to `setTimeout`), `this`
> can get "lost." A common fix is to use an arrow function, because arrow
> functions keep the surrounding `this`. See the lesson on
> [The `this` Keyword](../02-functions-and-scope/05-the-this-keyword.md) for the
> full story.

---

## Common Mistakes

- **Forgetting `new`.** Calling `Dog("Rex")` without `new` does not build an
  object correctly. Always use `new Dog("Rex")`.
- **Using `this` before `super()`** in a child constructor throws an error. Call
  `super(...)` first.
- **Calling a getter with parentheses.** `c.area` is correct; `c.area()` will
  fail because `area` is a value, not a function.
- **Calling a static method on an instance.** Static methods live on the class:
  `MathHelper.double(8)`, not `instance.double(8)`.
- **Losing `this`** when passing a method as a callback. Bind it or use an arrow
  function wrapper.

---

## Exercises

1. Create a class `Rectangle` with `width` and `height` properties and a method
   `area()`. Make two rectangles and print their areas.

2. Add a getter `perimeter` to `Rectangle` so you can read `rect.perimeter`
   without parentheses.

3. Create a class hierarchy: `Vehicle` (with a `name` and a `move()` method) and
   a child class `Car` that `extends Vehicle`, adds a `wheels` property, and
   overrides `move()` to mention driving. Use `super` properly.

4. Add a **static** method `Vehicle.compareSpeed(a, b)` that takes two vehicles
   with a `speed` property and returns the faster one's name.

---

## Key Takeaways

- **OOP** bundles data (properties) and behavior (methods) into objects.
- A **class** is a blueprint; **`new`** stamps out **instances**.
- The **prototype chain** lets objects fall back to other objects for properties.
- **`extends`** gives inheritance; **`super`** calls the parent constructor or
  methods.
- **Getters/setters** let methods look and feel like plain properties.
- **Static** methods belong to the class, not to instances.
- Inside a method, **`this`** is the object the method was called on.

---

[← Back to ES Modules](./01-es-modules.md) | [Next: Iterators and Generators →](./03-iterators-and-generators.md)
