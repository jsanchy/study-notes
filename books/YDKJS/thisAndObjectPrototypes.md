# Chapter 1

## Why `this`?

Provides more elegant way to implicitly pass along an object reference, leading to cleaner API design and easier re-use.

## Confusions

### Itself

Misconception: `this` refers to the function itself.

### Its Scope

Misconception: `this` refers to the function's scope.

Cannot create bridge between lexical scopes of functions.

## What's `this`?

- A runtime binding, NOT author-time binding.
- Nothing to do with where function is declared, but everything to do with manner in which function is called.
- When a function is invoked, an activation record (or execution context) is created, which contains info about where function was called from (call-stack), how it was invoked, what parameters were passed, etc.
- this record also contains the `this` reference which will be used for the duration of that function's execution.