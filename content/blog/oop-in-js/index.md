---
title: Types of OOP in JavaScript
date: "2020-11-4"
description: "A review of basic OOP strategies in JavaScript."
---

<!-- https://css-tricks.com/the-flavors-of-object-oriented-programming-in-javascript/?ck_subscriber_id=406386129#where-to-declare-properties-and-methods -->

1. Constructor functions
1. Classes
1. Objects linking to other objects (OLOO)
1. Factory functions

Classes vs. Factory functions

- Inheritance
- Encapsulation
- this
- Event listeners

## Vocabulary

- blueprint: common object
- instance: created objects
- Inheritance / subclassing: structuring code when you have multiple levels of blueprints
- encapsulation: hide information within the object so its not accessible

## Major Patterns

### Constructor Function

A function with a this keyword.

```js
function Human (firstName, lastName) {
  this.firstName = firstName
  this.lastName = lastName
}
const chris = new Human('Chris', 'Coyier')
```

### Class

The “syntactic sugar” of Constructor functions - an easier way of writing Constructor functions.

```js
class Human {
  constructor(firstName, lastName) {
    this.firstName = firstName
    this.lastName = lastName
  }
}
const chris = new Human('Chris', 'Coyier')
```

### OLOO

Define the blueprint as a normal object and use a method to initialize the instance.

```js
const Human = {
  init (firstName, lastName ) {
    this.firstName = firstName
    this.lastName = lastName
  }
}
const chris = Object.create(Human)
chris.init('Chris', 'Coyier')
```

or, if you return `this`:

```js
const Human = {
  init (firstName, lastName ) {
    this.firstName = firstName
    this.lastName = lastName

    return this
  }
}
const chris = Object.create(Human).init('Chris', 'Coyier')
```

### Factory Function

Factory functions are functions that return an object.

```js
function Human (firstName, lastName) {
  return {
    firstName,
    lastName
  }
}
const chris = Human('Chris', 'Coyier')
```

## Declaring properties and methods

### Constructor Function

Declare a property directly on an instance:

```js
function Human (firstName, lastName) {
  // Declares properties
  this.firstName = firstName
  this.lastname = lastName
  // Declares methods
  this.sayHello = function () {
    console.log(`Hello, I'm ${firstName}`)
  }
}
const chris = new Human('Chris', 'Coyier')
```

or declare properties on the Prototype:

```js
Human.prototype.sayHello = function () {
  console.log(`Hello, I'm ${this.firstName}`)
}
```

To make multiples easier use assign:

```js
Object.assign(Human.prototype, {
  method1 () { /*...*/ },
  method2 () { /*...*/ },
  method3 () { /*...*/ }
})
```

### Class Syntax

_Properties_ for each instance declared in the constructor:

```js
class Human {
  constructor (firstName, lastName) {
    this.firstName = firstName
      this.lastname = lastName

      this.sayHello = function () {
        console.log(`Hello, I'm ${firstName}`)
      }
  }
}
```

_Methods_ on the prototype:

```js
class Human (firstName, lastName) {
  constructor (firstName, lastName) { /* ... */ }

  sayHello () {
    console.log(`Hello, I'm ${this.firstName}`)
  }
}
```

### OLOO

```js
const Human = {
  init (firstName, lastName) {
    this.firstName = firstName
    this.lastName = lastName
    this.sayHello = function () {
      console.log(`Hello, I'm ${firstName}`)
    }
    return this
  }
}
const chris = Object.create(Human).init('Chris', 'Coyier')
```

_Methods_ on OLOO:

```js
const Human = {
  init () { /*...*/ },
  sayHello () {
    console.log(`Hello, I'm ${this.firstName}`)
  }
}
```

## Declaring properties and methods with Factory functions

```js
function Human (firstName, lastName) {
  return {
    firstName,
    lastName,
    sayHello () {
      console.log(`Hello, I'm ${firstName}`)
    }
  }
}
```

## Concepts

- Inheritance
- Encapsulation
- this

### Inheritance

The word “inheritance” is used when you receive things from somewhere (inheritance from your parents, inherited genes, inherit a process from your teacher).

Instances inherit from their blueprints.

v.s. _Subclassing_ - create a derivative blueprint from the parent blueprint by _extending_ it

```js
class Developer extends Human {
  constructor(firstName, lastName) {
    super(firstName, lastName)
  }
    // Add other methods
}
```

_super_ inits the constructor from Human.

Adding a method to the subclass:

```js
class Developer extends Human {
  code (thing) {
    console.log(`${this.firstName} coded ${thing}`)
  }
}
```

The process for Factory functiions (functions that return an object):

```js
function Human (firstName, lastName) {
  return {
    firstName,
    lastName,
    sayHello () {
      console.log(`Hello, I'm ${firstName}`)
    }
  }
}

function Developer (firstName, lastName) {
  const human = Human(firstName, lastName)
  return Object.assign({}, human, {
    // Properties and methods go here
  })
}
```

Adding a method:

```js
function Developer (firstName, lastName) {
  const human = Human(firstName, lastName)
  return Object.assign({}, human, {
    code (thing) {
      console.log(`${this.firstName} coded ${thing}`)
    }
  })
}
```

Methods can be overwritten

```js
class Developer extends Human {
  sayHello () {
    // Calls the parent method
    super.sayHello()
    // Additional stuff to run
    console.log(`I'm a developer.`)
  }
}
const chris = new Developer('Chris', 'Coyier')
```

Or in a factory function:

```js
function Developer (firstName, lastName) {
  const human = Human(firstName, lastName)
  return Object.assign({}, human, {
      sayHello () {
        // Calls the parent method
        human.sayHello()
        // Additional stuff to run
        console.log(`I'm a developer.`)
      }
  })
}
const chris = new Developer('Chris', 'Coyier')
```

## Composition over Inheritance.

Composition is the act of combining two things into one.

```js
const one = { one: 'one' }
const two = { two: 'two' }
const combined = Object.assign({}, one, two)
```

Or

```js
const skills = {
  code (thing) { /* ... */ },
  design (thing) { /* ... */ },
  sayHello () { /* ... */ }
}

class DesignerDeveloper {
  constructor (firstName, lastName) {
    this.firstName = firstName
    this.lastName = lastName

    Object.assign(this, {
      code: skills.code,
      design: skills.design,
      sayHello: skills.sayHello
    })
  }
}

class Designer {
  constructor (firstName, lastName) {
    this.firstName = firstName
    this.lastName = lastName

    Object.assign(this, {
      design: skills.design,
      sayHello: skills.sayHello
    })
  }
}

class Developer {
  constructor (firstName, lastName) {
    this.firstName = firstName
    this.lastName = lastName

    Object.assign(this, {
      code: skills.code,
      sayHello: skills.sayHello
    })
  }
}

const chris = new DesignerDeveloper('Chris', 'Coyier')
```

## Encapsulation

enclosing one thing inside another thing so the thing inside doesn’t leak out

```js
const food = 'Hamburger'

function sayFood () {
  console.log(food)
}
sayFood()
```

Closures - functions wrapped in functions.

```js
function outsideFunction () {
  const food = 'Hamburger'
  console.log('Called outside')

  return function insideFunction () {
    console.log('Called inside')
    console.log(food)
  }
}

// Calls `outsideFunction`, which returns `insideFunction`
// Stores `insideFunction` as variable `fn`
const fn = outsideFunction()

// Calls `insideFunction`
fn()
```

Private vars and public methods.

Not:

```js
class Car {
  constructor () {
    // Denotes that `_fuel` is private. Don't use it!
    this._fuel = 50
  }

  getFuel () {
    return this._fuel
  }

  setFuel (value) {
    this._fuel = value
    // Caps fuel at 100 liters
    if (value > 100) this._fuel = 100
  }
}
```

But: declare the private variable outside the constructor first

```js
class Car {
  // Declares private variable
  #fuel
  constructor () {
    // Use private variable
    this.#fuel = 50
  }
}
```

Encapsulation with Classes requires prepending # to the private variable. clunky.

Use factory functions

```js
function Car () {
  const fuel = 50

  return {
    get fuel () {
      return fuel
    },

    set fuel (value) {
      fuel = value
      if (value > 100) fuel = 100
    }
  }
}

const car = new Car()
console.log(car.fuel) // 50

car.fuel = 3000
console.log(car.fuel) // 100
```

## this variable

refers to the instance

in a class:

```js
class Human {
  constructor (firstName, lastName) {
    this.firstName = firstName
    this.lastName = lastName
    console.log(this)
  }
}

const chris = new Human('Chris', 'Coyier')
```

In constructor function

```js
function Human (firstName, lastName) {
  this.firstName = firstName
  this.lastName = lastName
  console.log(this)
}

const chris = new Human('Chris', 'Coyier')
```

## Counter with Classes

```html
<div class="counter">
  <p>
    Count: <span>0</span>
    <button>Increase Count</button>
  </p>
</div>
```

```js
class Counter () {
  constructor (counter) {
    // Do stuff
  }
}

// Usage
const counter = new Counter(document.querySelector('.counter'))
```

THis was a waste of time
