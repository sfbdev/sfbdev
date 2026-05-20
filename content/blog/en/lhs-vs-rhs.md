---
title: "LHS vs RHS: What Your JavaScript Errors Are Actually Telling You"
date: 2026-05-11
description: "The mechanism behind ReferenceError and TypeError comes down to two distinct kinds of variable look-ups."
---

You've seen `ReferenceError: x is not defined` and `TypeError: Cannot read properties of null` more times than you can count. You probably know one means "this variable doesn't exist" and the other means "the value isn't what you think it is." That's the surface. The actual mechanism behind these errors comes down to how JavaScript looks up variables — and there are two distinct kinds of look-ups happening every time you reference something.

Kyle Simpson calls them LHS and RHS. Left-Hand-Side and Right-Hand-Side of an assignment. The naming is academic but the distinction is real, and it explains a lot of error messages that otherwise feel arbitrary.

## Two Kinds of Asking

When the Engine encounters a variable name during execution, it asks Scope to find it. But it asks in one of two ways depending on what it wants to do.

**RHS look-up** — "give me the value of this variable." Engine wants to read.

```javascript
console.log(a)  // RHS for `a` (and for `console`, and for the `log` property)
```

Here, Engine doesn't want to *change* `a`. It wants to retrieve `a`'s value and hand it to `console.log`. That's an RHS reference.

**LHS look-up** — "give me the container for this variable, I'm going to write to it." Engine wants to assign.

```javascript
a = 2  // LHS for `a`
```

Engine doesn't care what `a` currently holds. It just needs the slot — the memory location — so it can put 2 in there.

In a single line you often have both:

```javascript
b = a + 1
// LHS for b (assigning to it)
// RHS for a (reading its value)
```

## What Happens When Look-ups Fail

This is where the distinction stops being academic and starts mattering.

**RHS fails** — the variable doesn't exist anywhere in the scope chain. Engine throws `ReferenceError`. Doesn't matter if you're in strict mode or not. There's nothing to read, so there's no question what to do.

```javascript
console.log(x)  // ReferenceError: x is not defined
```

**LHS fails in non-strict mode** — the variable doesn't exist, but Engine wants to assign to it. Scope helpfully creates a global variable with that name and hands it back. No error. Your code keeps running. You just accidentally polluted the global scope.

```javascript
function calc() {
  total = 0  // no var/let/const — oops
  // total is now window.total
}
```

This is the source of countless silent bugs in old codebases. A typo, a forgotten `let`, and suddenly you have a global variable you didn't intend.

**LHS fails in strict mode** — same situation, but strict mode refuses to create the global. Engine throws `ReferenceError` instead.

```javascript
'use strict'
function calc() {
  total = 0  // ReferenceError
}
```

This is one of the main reasons to always work in strict mode. Modules in ES6+ are strict by default, which means every Vue, React, or modern Node project you write is already protected. But it's worth knowing *why* the protection matters.

## TypeError Is a Different Beast

RHS and LHS failures both produce `ReferenceError`. `TypeError` shows up when the look-up *succeeds* but you're trying to do something illegal with the result.

```javascript
let x = null
x.foo  // TypeError — x exists, but you can't read properties of null

let y = 'hi'
y()    // TypeError — y exists, but strings aren't callable
```

So when you see:

- `ReferenceError` → the variable couldn't be found
- `TypeError` → the variable was found, but the operation is illegal

## Why This Mental Model Helps

Most developers debug errors by reading the message and guessing. Once you split look-ups into LHS and RHS, the error messages stop being mysterious. A `ReferenceError` means scope resolution failed. A `TypeError` means scope resolution succeeded but you're holding the wrong kind of value. Two different problems, two different fixes.

It's not a model you'll think about every day. But the next time you see an error, ask yourself which one it is. You'll find yourself fixing the right thing faster.

---

*Notes after reading Kyle Simpson's* You Don't Know JS: Scope & Closures *(1st edition). The mental models here are his; the bugs and the Vue parallels are mine.*
