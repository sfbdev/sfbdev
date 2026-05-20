---
title: "Hoisting Isn't What You Think: Why Nothing Actually Moves"
date: 2026-05-19
description: "Declarations are processed before code runs. The 'moving to the top' metaphor is just a way to picture the effect — no code actually moves."
---

Hoisting is one of those words that gets thrown around in JavaScript interviews like a checkbox: do you know it, yes or no? The usual answer is some version of "declarations get moved to the top of their scope." That's a useful visualization, but it's also wrong in a way that matters — because nothing is actually moved, and missing that detail is why hoisting feels weird.

The truth is simpler: declarations are processed before code runs. The visualization of "moving to the top" is just a way to picture the effect.

## The Two Phases, Again

Think back to compile vs execute. When JavaScript loads your file, the Compiler walks through every line and registers declarations with Scope. Only then does the Engine start executing the code top to bottom.

So a line like `var a = 2` is actually two operations:

- `var a` — handled at compile time. The Compiler tells Scope: "there will be a variable named `a` here."
- `a = 2` — handled at execution time. The Engine writes the value.

By the time the Engine reaches your code, all the declarations are already known. That's the entire mechanism. The "moving to the top" metaphor exists because it produces the same observable behavior — but no code actually moves.

## What Gets Hoisted, What Doesn't

Declarations hoist. Assignments don't. This is the single most important detail.

```javascript
console.log(x)  // undefined
var x = 2
```

This works (doesn't throw) because `var x` was registered with Scope at compile time. By the time `console.log(x)` runs, `x` exists — it just doesn't have a value yet. The `= 2` part hasn't happened yet because the Engine hasn't reached that line.

So when you imagine the code rewritten:

```javascript
var x         // hoisted — declaration only
console.log(x)  // undefined
x = 2          // assignment stays in place
```

The `2` doesn't go to the top. Only the existence of `x` does.

## Function Declarations Are Different

Function declarations get fully hoisted — both the name and the body:

```javascript
greet()  // works, prints 'hi'

function greet() {
  console.log('hi')
}
```

This is genuinely useful. You can call a function before you've written it, as long as it's a declaration. The Engine knows about the full function by the time it starts running.

Function expressions, on the other hand, only get the variable hoisted:

```javascript
greet()  // TypeError: greet is not a function

var greet = function() {
  console.log('hi')
}
```

Here, `var greet` is hoisted as `undefined`. The function value gets assigned later. So when you call `greet()` before the assignment, you're calling `undefined()` — which is a TypeError. The distinction between declaration and expression actually matters at runtime.

Also worth knowing: if you have both a `var` and a `function` declaration with the same name, the function wins. Function declarations are hoisted first, and a subsequent `var` of the same name is ignored at compile time.

## let and const: Hoisted, But in the TDZ

The common misconception: "let isn't hoisted." It is. It's just unreachable until its declaration line:

```javascript
console.log(x)  // ReferenceError, not undefined
let x = 2
```

This is the Temporal Dead Zone. The variable exists in scope, the Engine knows about it — but accessing it before the declaration throws. The reason this matters: it's not the same as "the variable doesn't exist." It's the variable existing in a deliberately inaccessible state. Same with `const`.

The upshot is good. The TDZ catches bugs that `var` silently swallowed. Code that tries to use a `let` before it's declared crashes loudly and immediately. That's the right behavior.

## A Subtle Case

This one trips most people up:

```javascript
console.log(typeof a)  // 'undefined' — safe
console.log(typeof b)  // ReferenceError — not safe
let b = 1
```

`typeof` is famous for not throwing on undeclared variables. That works for variables that genuinely don't exist. But `b` *does* exist — it's just in the TDZ. So `typeof` throws. This is one of the cases where the modern "safe" `typeof` pattern from `var` days breaks down.

## What This Means in Practice

In modern JavaScript, you write declarations before you use them. Always. This isn't because hoisting doesn't work — it's because relying on hoisting is bad style, and with `let` and `const` it'll throw anyway.

The one place hoisting still pulls weight: function declarations at the top of a module. You can structure your file with the main logic first and helper functions at the bottom, and it still works:

```javascript
go()

function go() {
  helper()
  // ...
}

function helper() {
  // ...
}
```

This only works because of function declaration hoisting. With arrow functions assigned to `const`, you'd have to declare in dependency order.

## The Reframe

Drop the "moves to the top" mental model. Replace it with: "the Compiler scans the code first and registers all declarations. Then the Engine runs it." Once you see hoisting as a two-phase process rather than physical motion, the edge cases stop being edge cases. They're just consequences of the model.

---

*Notes after reading Kyle Simpson's* You Don't Know JS: Scope & Closures *(1st edition). The mental models here are his; the bugs and the Vue parallels are mine.*
