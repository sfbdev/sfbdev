---
title: "Scope Chain and Shadowing: How JavaScript Actually Finds Your Variables"
date: 2026-05-15
description: "When a variable isn't found locally, the search walks outward — one level at a time, stopping at the first match. That's the scope chain."
---

When a function asks for a variable that isn't in its own scope, where does it look next? And what happens when the same name exists at multiple levels? These two questions have one answer in JavaScript, and that answer is responsible for some of the most subtle bugs you'll write.

The answer is the **scope chain**. Functions are nested inside each other, scopes are nested with them, and when a variable isn't found locally, the search walks outward — one level at a time, stopping at the first match.

## The Walk Outward

Think of nested scopes as nested rooms. You're in a room. Around it is a hallway. Around the hallway is the lobby. Around the lobby is the street.

```javascript
var a = 'street'

function lobby() {
  var a = 'lobby'

  function hallway() {
    var a = 'hallway'

    function room() {
      console.log(a)  // 'hallway'
    }

    room()
  }

  hallway()
}

lobby()
```

When `room` looks for `a`, it doesn't find one locally. So it walks out to `hallway`. Found one. Stops. Prints `'hallway'`. It never reaches `lobby` or the global scope, because the search ended at the first match.

The rule is simple: **the search only goes outward, and it stops the moment it finds something**. There's no scanning all levels, no "closest variable wins" logic. The first match — starting from the innermost scope — wins. Period.

## Shadowing

That "first match wins" rule has a name when you exploit it on purpose: **shadowing**. An inner scope can declare a variable with the same name as an outer one, and the inner version *shadows* the outer.

```javascript
var message = 'outer'

function speak() {
  var message = 'inner'
  console.log(message)  // 'inner'
}

speak()
console.log(message)    // 'outer'
```

The outer `message` still exists. It's not overwritten. It's just hidden from anything inside `speak` because `speak` has its own `message`. Outside `speak`, the outer one comes back.

Shadowing is sometimes useful (you want a clean local name) and sometimes a source of bugs (you accidentally shadow something you meant to read). Modern linters warn about it for that reason. The rule of thumb: shadow on purpose, not by accident.

## Property Look-up Is a Different Thing

There's a trap worth pointing out. The scope chain only handles plain identifiers — names like `a`, `foo`, `count`. Property access is a separate mechanism:

```javascript
foo.bar.baz
```

Here, only `foo` goes through the scope chain. JavaScript walks outward until it finds something called `foo`. Once it has `foo`, the `.bar` and `.baz` parts use a completely different system — property look-up on objects, which follows the prototype chain, not the scope chain.

This distinction matters when you're debugging. If `foo.bar` throws, the problem could be that `foo` doesn't exist (scope chain failure → `ReferenceError`) or that `foo` exists but doesn't have a `bar` property (property failure → usually `undefined`, no error). Two completely different mechanisms, two completely different fixes.

## Where This Hits Vue

In Composition API, the scope chain is doing real work inside `<script setup>`:

```javascript
<script setup>
const count = ref(0)

function increment() {
  count.value++  // scope chain finds `count` one level up
}
</script>
```

`increment` doesn't have its own `count`. It walks out to the `<script setup>` scope and finds it there. That's exactly the same mechanism as `room()` finding `a` in `hallway()`.

A more subtle case: nested composables that shadow.

```javascript
function useOuter() {
  const state = ref('outer')

  function inner() {
    const state = ref('inner')
    // any function here sees the inner state, not the outer
  }
}
```

If you weren't being careful, you might think `inner` could access *both* states. It can't. The inner `state` shadows the outer one for everything written inside `inner`.

## The Mental Model

When you're tracking down a "why does this variable have that value" bug, the scope chain is the answer. Walk outward from the function in question, level by level. The first match is the value being used. There's no other rule.

That's the whole mechanism. Most JavaScript developers never explicitly think about it because it usually does the obvious thing. But the moment something unexpected happens — a variable holding a value you didn't put there, or a shadowed name you forgot about — the scope chain is where you look.

---

*Notes after reading Kyle Simpson's* You Don't Know JS: Scope & Closures *(1st edition). The mental models here are his; the bugs and the Vue parallels are mine.*
