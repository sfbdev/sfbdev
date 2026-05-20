---
title: "Engine, Compiler, Scope: The Three Actors Behind Every JavaScript Program"
date: 2026-05-09
description: "JavaScript code goes through three actors before it does anything. Once you see them, a lot of weird behavior stops being weird."
---

When I started writing JavaScript, I thought the engine just ran code top to bottom. Read a line, execute it, move on. Most developers think this way for years and never get bitten by it. But once you start asking *why* hoisting works, *why* `var a = 2` does what it does, you realize there's a whole process happening before your code ever runs.

JavaScript code goes through three actors before it does anything. Once you see them, a lot of weird behavior stops being weird.

## The Three Actors

**Engine** is the boss. It's responsible for the entire lifecycle of your program — compilation, execution, and everything in between. V8 in Chrome, SpiderMonkey in Firefox, JavaScriptCore in Safari. When you load a page, the Engine takes over.

**Compiler** does the dirty work. It takes your raw source code and turns it into something executable. People don't usually associate JavaScript with compilation — it's marketed as an interpreted language. That's a half-truth. The compilation just happens microseconds before execution, in the same process. There's no separate build artifact like in C or Java, but the steps are the same: tokenize, parse, generate code.

**Scope** is the archivist. It maintains a list of every identifier (variables, functions) and enforces the rules about who can access what. When the Compiler finds a declaration, it tells Scope to register it. When the Engine encounters a reference at runtime, it asks Scope to find it.

## The Conversation

The statement `var a = 2` is not one thing. The Engine sees it as two:

First, at compile time, the Compiler hits `var a`. It asks Scope: "do you have an `a` in this scope?" Scope says no. Compiler says: "create one." Scope registers it. The `= 2` part is not handled here — it's left in the code as an instruction for later.

Then, at runtime, the Engine reaches the line and asks Scope: "do you have an `a` I can write to?" Scope says yes, points to the slot it created earlier. Engine writes 2 into it.

This two-phase model is the entire reason hoisting works. Declarations are processed before execution, so by the time the code runs, every `var` and `function` in the scope is already known. The famous `console.log(a); var a = 2;` printing `undefined` instead of throwing isn't magic — it's just the Compiler getting to `var a` before the log line ever executes.

## Why This Matters

Most of the confusing behavior in JavaScript stops being confusing when you separate the two phases in your head. Hoisting, the TDZ for `let` and `const`, the difference between `ReferenceError` and `TypeError` — they all fall out of this model.

The other thing that becomes obvious: scope is not something that exists at runtime, decided as your code runs. It's decided by where you wrote things. By the time the Engine starts executing, Scope already knows its shape. That's lexical scope, and it's the foundation everything else — closures included — sits on.

Understanding the three actors isn't trivia. It's the mental model that turns JavaScript from "a series of rules I memorized" into something you can reason about.

---

*Notes after reading Kyle Simpson's* You Don't Know JS: Scope & Closures *(1st edition). The mental models here are his; the bugs and the Vue parallels are mine.*
