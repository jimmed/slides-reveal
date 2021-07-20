---
title: "JS: The Weird Parts ─ The Proxy"
theme: black
highlightTheme: atom-one-dark
verticalSeparator: --
transition: fade
backgroundTransition: fade
showNotes: true
slideNumber: true
---

<h1 class="r-fit-text">JavaScript: The Weird Parts</h1>
<h2 class="r-fit-text">PROXIES</h2>
<h3 class="r-fit-text">Part 1 of a series of incoherent ramblings by <a href="https://github.com/jimmed" target="_blank">Jim</a></h3>

---

# Everything is an `object`<sup>\*</sup>

<span style="font-size: 1rem">\*except things that aren't</span>

---

# Defining an object

--

```js [1-5|1-3|2|5]
const user = {
  name: "Jim",
}

console.log(user.name)
```

--

```js [1-8|1-3|5|7]
const user = {
  name: "Jim",
}

user.name = "Not Jim"

console.log(user.name)
```

---

# Protecting variables

--

## Using a closure

```js [1-11|2|3-7|4-6|5|10|11]
function user(name) {
  const secretName = name
  return {
    get name() {
      return secretName
    },
  }
}

const me = user("Jim")
me.name = "Not Jim"
```

Note: getter/setter accessors added in ES5

--

## Defining a property

```js [1-8|2|3|4-7|10]
const user = Object.defineProperty(
  {},
  "name",
  {
    value: "Jim",
    writable: false,
  }
});

user.name = 'Not Jim'
```

Note: since ES5

--

## Defining multiple properties

```js [1-7|4|5|9]
const user = Object.defineProperties(
  {},
  {
    name: { value: "Jim", writable: false },
    role: { value: "Engineer", writable: true },
  }
)

user.name = "Not Jim"
```

Note: since ES5

--

## Private properties

```js
class User {
  #name

  constructor(name) {
    this.#name = name
  }

  get name() {
    return this.#name
  }
}

const me = new User("Jim")
me.name = "Not Jim"
```

Note: Classes added in ES6, but private fields/members proposal is ES2022 (stage 3)

--

## Proxying a getter

```js [1-9|2|3-8|4-7|5|6|11]
const user = new Proxy(
  {},
  {
    get(obj, key, receiver) {
      if (key === "name") return "Jim"
      return Reflect.get(obj, key, receiver)
    },
  }
)

console.log(user.name)
```

Note: Proxies added in ES6

---

# Proxies

--

> The `Proxy` object enables you to create a proxy for another object, which can **intercept and redefine fundamental operations for that object**.
>
> ─ <cite>[MDN Web Docs](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy)</cite>

--

```js []
const user = new Proxy(
  {},
  {
    get(obj, key, receiver) {
      if (key === "name") return "Jim"
      return Reflect.get(obj, key, receiver)
    },
  }
)
```

--

```js
const proxy = new Proxy(target, handler)
```

--

```js [1-6|2|3|4|8|9]
const proxy = new Proxy(target, {
  has(target, key, receiver) {
    if (key === "name") return true
    return Reflect.has(target, key, receiver)
  },
})

"name" in target
"name" in proxy
```

--

```js [1-6|2|3|4|8|9]
const proxy = new Proxy(target, {
  set(target, key, value, receiver) {
    if (key === "name") return false
    return Reflect.set(target, key, value, receiver)
  },
})

target.name = "Jim"
proxy.name = "Jim"
```
