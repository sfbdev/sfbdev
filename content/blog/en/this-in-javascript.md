---
title: "this in JavaScript: Why It's Not What You Think"
date: 2026-05-12
description: "this doesn't refer to the function or its scope — it refers to the call site. Here are the four rules that determine it."
---

Every JavaScript developer has been burned by `this` at least once. You write a method, you pass it as a callback, and suddenly `this.something` is undefined. You google, you find `var self = this`, you slap a `.bind(this)` somewhere, the bug goes away, and you move on without ever really understanding what happened.

I did this for years. Then I sat down and forced myself to actually figure it out.

Here's the thing: `this` doesn't refer to the function. It doesn't refer to the function's scope. It doesn't refer to where the function is *defined*. It refers to the **call site** — where and how the function is invoked. That's the entire mechanism. The reason it feels magical is that the rules for determining the call site aren't obvious until you've seen them spelled out.

## The Motivation

Before the rules, the *why*. Consider this:

```javascript
function identify() {
  return this.name.toUpperCase()
}

const me = { name: 'Furkan' }
const you = { name: 'Reader' }

identify.call(me)   // 'FURKAN'
identify.call(you)  // 'READER'
```

One function. Two different contexts. Without `this`, I'd have to either write two functions or pass the context as an explicit parameter — both of which clutter the API. `this` is JavaScript's way of letting you pass context *implicitly*. That's the whole point.

The problem isn't that `this` exists. The problem is that the rules for what it points to are unintuitive at first glance.

## The Four Rules (And Their Order)

Whenever a function is called, JavaScript asks four questions, in order. The first one that answers "yes" determines `this`.

**1. Was it called with `new`?**

```javascript
function Person(name) {
  this.name = name
}
const p = new Person('Furkan')
```

`new` creates a fresh object, binds `this` to it, and (unless you return your own object) returns it. This is why constructors work.

**2. Was it called with `.call()`, `.apply()`, or `.bind()`?**

```javascript
function greet() { console.log(this.name) }
const user = { name: 'Furkan' }

greet.call(user)        // 'Furkan' — explicit binding
greet.apply(user)       // same, args as array
const bound = greet.bind(user)
bound()                  // 'Furkan' — pre-bound
```

Explicit binding. You're literally telling JavaScript what `this` should be.

**3. Was it called as a method on an object?**

```javascript
const obj = {
  name: 'Furkan',
  greet() { console.log(this.name) }
}
obj.greet()  // 'Furkan' — `this` is whatever's to the left of the dot
```

The dot rule. `this` becomes the object that owns the method call at *that moment*.

**4. None of the above?**

Then `this` is the global object (`window` in browsers) — or `undefined` in strict mode. This is the default, and it's where most bugs come from.

## The Bug Everyone Hits

```javascript
const obj = {
  name: 'Furkan',
  greet() { console.log(this.name) }
}

obj.greet()              // 'Furkan' — rule 3
setTimeout(obj.greet, 0) // undefined — rule 4
```

Same function. Different call site. `setTimeout` extracts the function reference and calls it standalone — no dot, no `new`, no `.bind`. Rule 4 kicks in. `this` becomes the global object, which doesn't have a `name` property. You get undefined and you wonder what went wrong.

The fix is whichever rule overrides 4: `setTimeout(obj.greet.bind(obj), 0)`, or wrap it in an arrow function (more on that below).

## Arrow Functions Don't Play This Game

Arrow functions don't have their own `this`. They inherit it from the surrounding lexical scope — same way they'd inherit any other variable. The four rules above? Arrow functions ignore all of them.

```javascript
const obj = {
  name: 'Furkan',
  greet() {
    setTimeout(() => {
      console.log(this.name)  // 'Furkan' — inherited from greet
    }, 0)
  }
}
obj.greet()
```

The old `var self = this` pattern was developers manually emulating what arrow functions now do automatically. ES6 didn't invent the behavior — it just gave it syntax.

This is also why `.bind()` and `.call()` are pointless on arrow functions. They literally cannot have their `this` overridden. It's set when the function is defined and never changes.

## Where This Hits in Vue

Vue's Options API uses `this` heavily — it points to the component instance:

```javascript
export default {
  data() {
    return { count: 0 }
  },
  mounted() {
    setTimeout(function() {
      this.count++  // undefined.count — boom
    }, 1000)

    setTimeout(() => {
      this.count++  // works — arrow inherits component instance
    }, 1000)
  }
}
```

This is exactly the bug I'd hit early on. Use a regular function in a callback inside a Vue method and `this` is lost. Use an arrow function and it's preserved. The Composition API sidesteps the whole problem by killing `this` entirely — you work with refs in lexical scope, no implicit context to lose.

That's not an accident. The Vue team specifically moved away from `this` because of how many real-world bugs it caused.

## What I'd Tell Myself Five Years Ago

Stop guessing about `this`. Look at the call site. Apply the four rules in order. If you can't tell what `this` is by inspecting the code at the point where the function is *called*, the answer is rule 4 — global or undefined.

And if you're writing a callback that needs to access something from an enclosing context, just use an arrow function. That's what it's there for. Don't pretend `var self = this` is a pattern. It's a workaround that got replaced.

The whole mechanism stops being magical once you accept it: `this` is a function's invisible first argument, and the four rules decide what gets passed.

---

*Notes after starting Kyle Simpson's* You Don't Know JS: this & Object Prototypes *(1st edition). The four-rule framing is his; the call-site emphasis and Vue examples are mine.*
