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

<style type="text/css">
.reveal code {
  font-family: "Iosevka Term", monospace;
}
</style>

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

# Protecting Properties

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

## Property Descriptors

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

## Property Descriptors

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

```js [|1-12|2|5|9|13|13-14]
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

---

# Proxies

--

> The `Proxy` object enables you to create a proxy for another object, which can **intercept and redefine fundamental operations for that object**.
>
> ─ <cite>[MDN Web Docs](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy)</cite>

--

```js
const proxy = new Proxy(target, handler)
```

--

```js [1-6|2-5|3|4|8|1-8]
const user = new Proxy(target, {
  get(obj, key, receiver) {
    if (key === "name") return "Jim"
    return Reflect.get(obj, key, receiver)
  },
})

console.log(user.name)
```

--

```js [1-6|2|3|4|8|9]
const proxy = new Proxy(target, {
  has(target, key, receiver) {
    if (key === "name") return true
    return Reflect.has(target, key, receiver)
  },
})

console.log("name" in target)
console.log("name" in proxy)
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

--

<style type="text/css">
table { font-size: 1.5rem; }
table code { color: powderblue; font-size: 1.3rem;}
</style>
<table>
<thead><tr><th>Handler Method</th><th>Traps</th></tr></thead>
<tbody>
<tr><td><code>has(<em>t, key</em>)</code></td><td>querying a property (<code>key in proxy</code>)</td></tr>
<tr><td><code>get(<em>t, key, receiver</em>)</code></td><td>getting a property (<code>proxy[key]</code>)</td></tr>
<tr><td><code>set(<em>t, key, value, receiver</em>)</code></td><td>setting a property (<code>proxy[key] = value</code>)</td></tr>
<tr><td><code>deleteProperty(<em>t, key, value, receiver</em>)</code></td><td>deleting a property (<code>delete proxy[key]</code>)</td></tr>
</table>

--

<table>
<thead><tr><th>Handler Method</th><th>Traps</th></tr></thead>
<tbody>
<tr><td><code>ownKeys(<em>t</em>)</code></td><td>enumerating an object (<code>Object.keys(proxy)</code>)</td></tr>
<tr><td><code>getPrototypeOf(<em>t</em>)</code></td><td>querying an object (<code>Object.getPrototypeOf(proxy)</code>)</td></tr>
<tr><td><code>setPrototypeOf(<em>t, proto</em>)</code></td><td><code>Object.setPrototypeOf(proxy, proto)</code><sup>*</sup></td></tr>
</table>

<small style="font-size: 1rem">\*don't do this</small>

--

<table>
<thead><tr><th>Handler Method</th><th>Traps</th></tr></thead>
<tbody>
<tr><td><code>defineProperty(<em>t, key, attr</em>)</code></td><td>defining a property descriptor (<code>Object.defineProperty(proxy, key, attr)</code>)</td></tr>
<tr><td><code>getOwnPropertyDescriptor(<em>t, key</em>)</code></td><td>querying a property descriptor (<code>Object.getOwnPropertyDescriptor(proxy, key)</code>)</td></tr>
</table>

--

<table>
<thead><tr><th>Handler Method</th><th>Traps</th></tr></thead>
<tbody>
<tr><td><code>preventExtensions(<em>t</em>)</code></td><td>freezing an object (<code>Object.freeze(proxy)</code>)</td></tr>
<tr><td><code>isExtensible(<em>t</em>)</code></td><td>querying an object (<code>Object.isExtensible(proxy)</code>)</td></tr>
</table>

--

<table>
<thead><tr><th>Handler Method</th><th>Traps</th></tr></thead>
<tbody>
<tr><td><code>apply(<em>t, thisArg, args</em>)</code></td><td>function application (<code>proxy.apply(thisArg, args)</code>)</td></tr>
<tr><td><code>construct(<em>t, args, newTarget</em>)</code></td><td>constructor calls (<code>new proxy(args)</code>)</td></tr>
</table>

---

# Proxies in practice

--

## "Private" properties

```js [|1|3-7|4|5|6|9-12|14|16|17]
const isPublic = (key) => !key.startsWith("_")

const handler = {
  ownKeys: (t) => Reflect.ownKeys(t).filter(isPublic),
  has: (t, k) => isPublic(k) && Reflect.has(t, k),
  get: (t, k, r) => (isPublic(k) ? Reflect.get(t, k, r) : undefined),
}

const target = {
  name: "Jim",
  _secret: "foo",
}

const proxy = new Proxy(target, handler)

console.log(target._secret)
console.log(proxy._secret)
```

--

## Transforming object keys

```js [|3-7|4|5|6|9|11|13]
import { snakeCase, camelCase } from "lodash"

const handler = {
  ownKeys: (t) => Reflect.ownKeys(t).map(camelCase),
  has: (t, k) => Reflect.has(t, snakeCase(k)),
  get: (t, k, r) => Reflect.get(t, snakeCase(k), r),
}

const target = { some_kinda_property: "foo" }

const proxy = new Proxy(target, handler)

console.log(proxy.someKindaProperty)
```

--

## Callable constructors

```js [1-5|7|8]
class User {
  constructor(name) {
    this.name = name
  }
}

const jim = new User("Jim")
const billy = User("Billy")
```

--

## Callable constructors

```js [1-3|5|7|8]
const callableHandler = {
  apply: (t, thisArg, args) => new t(...args),
}

const user = new Proxy(User, callableHandler)

const jim = user("Jim")
const allUsers = ["Alice", "Bob"].map(user)
```

--

## Weird Fluent Interfaces

```js [|1-5|4|3|7|9]
const logger = (...items) =>
  new Proxy(() => {}, {
    apply: () => console.log(...items),
    get: (_, k) => logger(...items, k),
  })

const log = logger()

log.something.about.javascript.witchcraft()
```

---

# Proxies with TypeScript

--

# Type Gymnastics

```ts [|1-4|6-8|10|12|14-16]
const target = {
  name: "Jim",
  _secret: "foo",
}

function makePrivate<T extends {}>(obj: T): Private<T> {
  return new Proxy(obj as Private<T>, privateHandler)
}

const proxy = makePrivate(target)

type Private<T extends {}> = Pick<T, PublicProps<T>>

type PublicProps<T extends {}> = {
  [K in keyof T]: K extends `_${string}` ? never : K
}[keyof T]
```

---

# Revocable Proxies

--

# Revocable Proxies

```js [1|3|5|5,6]
const revocable = Proxy.revocable(target, handler)

revocable.proxy.doSomething()

revocable.revoke()
revocable.proxy.doSomething()
```

---

# Proxies In The Wild

--

# immer

```js [1-5|7]
import { produce } from "immer"

const producer = produce((state, action) => {
  state.some.deeply.nested.value += action.count
}, initialState)

const nextState = producer(prevState, 1)
```

--

```js [1-16|18]
const reducer = (state, action) => (
  {
    ...state,
    some: {
      ...state.some,
      deeply: {
        ...state.some.deeply,
        nested: {
          ...state.some.deeply.nested,
          value: state.some.deeply.nested.value + action.count,
        },
      },
    },
  },
  initialState
)

const nextState = reducer(prevState, 1)
```

--

# React DOM events

```jsx []
const Input = ({ value, onChange }) => (
  <input type="text" value={value} onChange={onChange} />
)
```

--

```js [|2|4-6]
const onChange = (event) => {
  doSomething(event.target.value)

  setTimeout(() => {
    doSomething(event.target.value)
  }, 100)
}
```

---

<h3 class="r-fit-text">Questions?</h3>
<h3 class="r-fit-text">Part 1 of a series of incoherent ramblings by <a href="https://github.com/jimmed" target="_blank">Jim</a></h3>
