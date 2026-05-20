---
title: "var, let, const: The Differences That Actually Matter"
date: 2026-05-17
description: "Three differences matter between var, let, and const: scope, reassignment, and pre-declaration behavior."
---

The usual explanation goes: `var` is old, `let` and `const` are new, just use the new ones. That's true, but it's also the kind of advice that gets passed around without anyone really understanding *why*. The actual differences between these three are more interesting than "use the new one," and they explain a class of bugs that I used to write without knowing why.

Three differences matter: how they handle scope, how they handle reassignment, and how they behave before they're declared.

## Scope: Function vs Block

`var` is function-scoped. It doesn't care about blocks. If you declare a `var` inside an `if` or a `for` loop, it leaks out to the enclosing function.

```javascript
function example() {
  if (true) {
    var x = 1
  }
  console.log(x)  // 1 — x escaped the if block
}
```

This surprises everyone the first time. The `if` block looks like a container, but for `var`, it isn't one. The only scope `var` respects is the function it's in.

`let` and `const` are block-scoped. They live and die inside whatever set of braces they're declared in:

```javascript
function example() {
  if (true) {
    let x = 1
  }
  console.log(x)  // ReferenceError — x doesn't exist here
}
```

This is the behavior most developers expect from variables. Other languages have worked this way for decades. JavaScript got around to it with ES6.

## Reassignment: const Is Not What You Think

`var` and `let` can both be reassigned. `const` cannot. But the word "constant" is misleading — it only protects the binding, not the value.

```javascript
const arr = [1, 2, 3]
arr.push(4)        // fine — modifying the array's contents
console.log(arr)   // [1, 2, 3, 4]

arr = []           // TypeError — can't rebind
```

`const` means "this name will always point to this exact value." If the value is an object or array, the object's internals can change freely. Only the *reference* is locked.

This catches people. Developers see `const obj = { ... }` and assume the object is frozen. It isn't. If you need actual immutability, you reach for `Object.freeze`, `readonly` in TypeScript, or a library.

## Pre-Declaration Access: undefined vs ReferenceError

This is the difference that actually changes how you write code.

`var` declarations are hoisted to the top of their function, initialized to `undefined`. Access before the declaration line returns `undefined`. No error.

```javascript
console.log(x)  // undefined
var x = 1
```

`let` and `const` are also hoisted, but they sit in the **Temporal Dead Zone** until their declaration line runs. Access during the TDZ throws a `ReferenceError`.

```javascript
console.log(x)  // ReferenceError
let x = 1
```

This is good. Really good. The `var` version silently returns `undefined`, which means a typo or a misplaced declaration becomes a subtle bug that surfaces three function calls later when you try to use the value. The `let` version blows up immediately, exactly where the problem is.

The common mistake here is to say "let isn't hoisted." It is hoisted. It just isn't accessible until its declaration line. That distinction matters because it explains why `let` and `const` have a TDZ at all — the variable exists, it's just deliberately unreachable until the engine reaches the point where you said it should be initialized.

## A Bonus: typeof on TDZ Variables

```javascript
console.log(typeof undeclaredVar)  // 'undefined' — safe
console.log(typeof x)               // ReferenceError
let x = 1
```

`typeof` is famously safe — it doesn't throw on undeclared variables, it returns `'undefined'`. But it does throw on TDZ variables, because they *are* declared, just unreachable. This breaks the old pattern of using `typeof` as a safety check. It's a small wart in the language, worth knowing about.

## What I Actually Use

Default to `const`. Switch to `let` when you genuinely need to reassign. Never use `var` in new code. The reasoning isn't "new is better" — it's that `const` and `let` give you stricter guarantees, and stricter guarantees mean fewer bugs.

In Vue 3 Composition API specifically:

```javascript
const count = ref(0)        // const — the ref binding never changes
const { x, y } = useStuff() // const — destructured bindings
let timer                    // let — will be assigned later
let intervalId               // let — same
```

The pattern of using `const` for refs feels weird at first because the value inside `.value` clearly changes. But the *ref itself* doesn't — it's the same object the whole time. The reactivity happens inside it. `const` is the right tool.

## The One Line Summary

`var` is function-scoped, hoisted with `undefined`, and reassignable. `let` is block-scoped, in TDZ until declaration, and reassignable. `const` is block-scoped, in TDZ until declaration, and the binding can't change — though the value can if it's an object or array.

That's the entire mechanism. The rest is style.

---

*Notes after reading Kyle Simpson's* You Don't Know JS: Scope & Closures *(1st edition). The mental models here are his; the bugs and the Vue parallels are mine.*
