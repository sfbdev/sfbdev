---
title: "Lexical Scope: Why JavaScript Reads Variables Where You Wrote Them, Not Where You Called Them"
date: 2026-05-14
description: "The entire reason closures, modules, and composables work is that JavaScript picked lexical scope over dynamic scope."
---

There's a question that sounds trivial but separates developers who *think* they understand scope from those who actually do: when a function looks for a variable, does it use the rules at the place where it was *written*, or the place where it was *called*?

JavaScript answers this one way. Some languages answer it the other way. The two models have names: lexical scope and dynamic scope. And the entire reason closures work — the entire reason modules work, the entire reason composables work — is that JavaScript picked lexical.

## Two Possible Worlds

```javascript
function foo() {
  console.log(a)
}

function bar() {
  var a = 2
  foo()
}

var a = 1
bar()
```

What does this print?

In a **lexical scope** world, `foo` looks for `a` in the scope where `foo` was *written*. It was written next to `var a = 1` at the top level. So it prints `1`.

In a **dynamic scope** world, `foo` looks for `a` in the scope of whoever *called* it. `bar` called it, and `bar` has `a = 2`. So it would print `2`.

JavaScript prints `1`. JavaScript is lexically scoped. Bash, by contrast, is dynamically scoped — a function in bash sees variables from whatever shell function called it. They're different worlds.

## Why Lexical Is Better

Dynamic scope sounds flexible. "Inherit context from your caller" feels natural in some ways. But it's a debugging nightmare. To know what a function does, you'd need to know who called it — and who called *that* — and so on. The function's behavior depends on its dynamic position in the call stack at runtime.

Lexical scope is the opposite. To know what a function can access, you read the code. The structure of the source decides everything. Nothing about the runtime changes which variables a function can see. This is what makes JavaScript code *readable* in the static sense: you can look at a snippet and reason about it.

It's also what makes closures possible. A closure works because the function carries its lexical scope around with it. If JavaScript were dynamically scoped, that wouldn't make sense — the function would just inherit whoever called it. There'd be nothing to "close over."

## What Lexical Actually Means

The word "lexical" refers to the **lexing phase** — the very first step the compiler runs, where it converts source code into tokens. By the time lexing is done, the structure of nested scopes is already fixed. Functions inside functions, blocks inside blocks — the nesting is in the source, and that's that. No matter how the code runs, that nesting doesn't change.

This is why people say things like "scope is determined at author time, not runtime." When you wrote the function, you decided what scope it lives in. The function carries that decision forever.

## Where This Shows Up in Vue

Every composable you write relies on lexical scope:

```javascript
function useCounter() {
  const count = ref(0)

  function increment() {
    count.value++
  }

  return { count, increment }
}
```

`increment` can see `count` because of where `increment` was *written* — inside `useCounter`. After `useCounter` returns and you call `increment` from a component, somewhere completely different in the code, it still finds `count`. That's lexical scope doing its job. The function remembers where it was born.

If JavaScript were dynamically scoped, `increment` would try to find `count` wherever it was called from — in the component scope, where `count` doesn't exist. Composables would be impossible. So would basically every pattern you use day to day.

## The Mental Shift

The shift from "scope is about what's happening when the code runs" to "scope is decided by where the code is written" is small in words but huge in practice. Once you internalize it, a lot of things click — closures, modules, why arrow functions inherit `this` from their definition site rather than their call site.

It all comes back to one decision the language designers made decades ago: scope follows the source, not the caller.

---

*Notes after reading Kyle Simpson's* You Don't Know JS: Scope & Closures *(1st edition). The mental models here are his; the bugs and the Vue parallels are mine.*
