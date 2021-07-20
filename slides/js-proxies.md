---
theme: black
verticalSeparator: --
---

<style type="text/css">
@import "https://fonts.googleapis.com/css?family=Vibur:400";
.neon-text {
  color: #fff;
  font-family: Vibur;
  text-transform: none;
  text-shadow:
      0 0 7px #fff,
      0 0 10px #fff,
      0 0 21px #fff,
      0 0 42px #0fa,
      0 0 82px #0fa,
      0 0 92px #0fa,
      0 0 102px #0fa,
      0 0 151px #0fa;

}
</style>
<h1 class="r-fit-text">JavaScript: The Weird Parts</h1>
<h2 class="r-fit-text">The Proxy</h2>
<h3 class="r-fit-text">Part 1 of a series of incoherent ramblings by Jim O'Brien</h3>

---

# Everything is an `object`<sup>\*</sup>

<span style="font-size: 1rem">\*except things that aren't</span>

---

# Defining an object

--

```js [1-5|1-3|2|5]
const user = {
  name: "Jim",
};

console.log(user.name);
```

--

```js [1-8|1-3|5|7]
const user = {
  name: "Jim",
};

user.name = "Not Jim";

console.log(user.name);
```

---

# Protecting variables

--

```js [1-10|1,7|2,6|3,5|4|9|10]
function user(name) {
  return {
    get name() {
      return name;
    },
  };
}

const me = user("Jim");
me.name = "Not Jim";
```

--

---

# Getters & Setters

--

```js
const user = Object.defineProperty({}, "name", {
  value: "Jim",
  writable: false,
});
```

--

```js
const user = Object.defineProperties(
  {},
  {
    name: { value: "Jim", writable: false },
    role: { value: "Engineer", writable: true },
  }
);
```

---

# Getters/Setters

<pre><code data-trim>
class User {
  #name;

  constructor(name) {
    this.#name = name;
  }

  get name() {
    return this.#name;
  }
}
</code></pre>

Note: No need to use a class here

---
