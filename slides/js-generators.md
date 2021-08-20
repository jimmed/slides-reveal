---
title: "JS: The Weird Parts ─ Generators"
theme: black
highlightTheme: atom-one-dark
verticalSeparator: --
transition: fade
backgroundTransition: fade
showNotes: true
slideNumber: true
---

<style type="text/css">
.reveal code {
  font-family: "Iosevka Term", monospace;
}
</style>

<h1 class="r-fit-text">JavaScript: The Weird Parts</h1>
<h2 class="r-fit-text">GENERATORS</h2>
<h3 class="r-fit-text">Part 2 of a series of incoherent ramblings by
<a href="https://github.com/jimmed" target="_blank">Jim</a></h3>

---

### Objects & Arrays

<aside class="notes">
In the old days of JS (ES3) there were only objects and arrays.

Arrays are used for ordered lists, indexed by a number.

Objects are used as unordered key/value mappings, keyed by a string.

</aside>

```js []
var alice = {
  name: "Alice",
  admin: false,
}

var bob = {
  name: "Bob",
  admin: true,
}

var users = [alice, bob]
```

--

### Iterating over objects

<aside class="notes">
  Iterating over the properties of an object isn't so unpleasant; for..in loop
</aside>

```js []
var alice = { name: "Alice", admin: false }

for (var key in alice) {
  console.log(key + ": " + alice[key])
}
```

--

### Iterating over arrays

<aside class="notes">
  If you wanted to iterate over an array, you needed a for loop
</aside>

```js []
var users = [alice, bob, chico]

var userNames = []
for (var i = 0; i < i.length; i++) {
  userNames[i] = users[i].name
}

// userNames => ['Alice', 'Bob', 'Chico']
```

--

### "Functional Programming"

<aside class="notes">
  This lead many developers to implement functions like this, à la underscore/lodash
</aside>

```js []
function map(array, callback) {
  var results = []

  for (let i = 0; i < array.length; i++) {
    results[i] = callback(array[i], i, array)
  }

  return results
}
```

--

<aside class="notes">
  Allowed for tidier iteration of arrays
</aside>

```js
var userNames = map(users, function (user) {
  return user.name
})

// userNames => ['Alice', 'Bob', 'Chico']
```

--

### ES5

<aside class="notes">
  ES5 added a bunch of methods to the array prototype
</aside>

```js
var users = [alice, bob, chico, dave]

var userNames = users.map(function (user) {
  return user.name
})

// userNames => ['Alice', 'Bob', 'Chico', 'Dave']
```

--

### ES6

<aside class="notes">
  ES6 added among other things, arrow functions
</aside>

```js
const users = [alice, bob, chico, dave]

const userNames = users.map(user => user.name)
```

--

### (Arrow functions ftw!)

```js
const allNonAdminUsersHaveALongName = users
  .filter(user => !user.admin)
  .map(user => user.name)
  .every(name => name.length > 5)
```

--

### ES2015 Sets

<aside class="notes">
  ES2015 added new data structures
</aside>

```js
const set = new Set([1, 2, 3])

set.has(3) // true
set.add(4)
set.delete(1)
```

--

<aside class="notes">
Can't use .map like an array

Can cast to Array but gets messy

Can also spread

</aside>

```js [1|3|5]
set.map(item => item * 2) // Error

Array.from(set).map(item => item * 2) // Works

[...set].map(item => item * 2) // Also works
```

--

### for..of

<aside class="notes">
  ES6 added the for..of loop.

Works for arrays, sets & maps; anything Iterable.

</aside>

```js [1|3|5]
for (let item of [1, 2, 3]) console.log(item)

for (let item of new Set([1, 2, 3])) console.log(item)

for (let char of "abcd") console.log(char)
```

---

# Iterables & Iterators

--

## The Iterable Protocol

> The iterable protocol allows JavaScript objects to **define or customize their
> iteration behavior**, such as what values are looped over in a `for...of` construct.
>
> <cite>&mdash; [MDN JavaScript reference][iterable-protocol]</cite>

[iterable-protocol]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Iteration_protocols#the_iterable_protocol

--

### Making an object iterable

<aside class="notes">
As of ES6, any object can implement the iterable protocol by specifying a `Symbol.iterator`
function that returns an iterator
</aside>

```js
const iterableThing = {}

iterableThing[Symbol.iterator] = // some function
```

--

### Iterators

<aside class="notes">
An iterator is an object with a `next` method which returns the next item
</aside>

```js [1-10|2-9|3|4-8|12-14]
const counter = {
  [Symbol.iterator]() {
    let value = 0
    return {
      next() {
        return { done: false, value: value++ }
      },
    }
  },
}

for (let i of counter) {
  console.log(i)
}
```

--

### _Finite_ Iterators

```js [|6-7]
const counter = {
  [Symbol.iterator]() {
    let value = 0
    return {
      next() {
        if (value === 10) return { done: true }
        return { done: false, value: value++ }
      },
    }
  },
}
```

--

<aside class="notes">
  Let's refactor our iterable so that it can accept some arguments
</aside>

```js [1-13|15-17]
function counter(start = 0, end = Infinity, step = 1) {
  return {
    [Symbol.iterator]() {
      let value = start
      return {
        next: () =>
          value === end
            ? { done: true }
            : { done: false, value: (value += step) },
      }
    },
  }
}

for (let i of counter(1, 10)) {
  console.log(i)
}
```

--

<aside class="notes">
  Getting pretty messy, so refactor as a class
</aside>

```js [1-9|11-25|27-28]
class CounterIterable {
  constructor(options) {
    this.options = { start: 0, end: Infinity, step: 1, ...options }
  }

  [Symbol.iterator]() {
    return new CounterIterator(this.options)
  }
}

class CounterIterator {
  constructor(options) {
    this.options = options
    this.value = options.start
  }

  next() {
    if (this.value < this.options.end) {
      this.value += this.options.step
      return { done: false, value: this.value }
    }

    return { done: true }
  }
}

const counter = new CounterIterable({ start: 0, end: 10 })
for (let i of counter) console.log(i)
```

--

<aside class="notes">
While iterables give a common protocol for iterating over arbitrary things,
in the simplest case, this is a lot of code to replicate what could just be a
for loop.
</aside>

```js
for (let i = 0; i < 10; i++) console.log(i)
```

---

## Generators

--

### Basic generator

```js [1-5|7-9]
function* counter(start = 0, end = Infinity, step = 1) {
  for (let i = start; i < end; i += step) {
    yield i
  }
}

for (let i of counter(5, 105, 10)) {
  console.log(i)
}
```

--

```js [1-4|6-8|10-14|16-19|21-23]
function* sequence(initialValue, getNext) {
  let value = initialValue
  while (true) yield (value = getNext(value))
}

function numberSequence(start = 0, step = 1) {
  return sequence(start, n => n + step)
}

function* takeUntil(iterable, shouldStop) {
  for (let item of iterable)
    if (shouldStop(item)) return
    else yield item
}

function counter(start = 0, end = Infinity, step = 1) {
  const numbers = numberSequence(start, step)
  return takeUntil(numbers, i => i >= end)
}

for (let i of counter(0, 10, 2)) {
  console.log(i)
}
```

--

```js [1-5|7|8-10]
function* repeat(generator, times) {
  for (let i = 0; i < times; i++) {
    yield* generator(i)
  }
}

const repeatingCounter = repeat(i => counter(1, i), 5)
for (let i of repeatingCounter) {
  console.log(i)
}
```

--
