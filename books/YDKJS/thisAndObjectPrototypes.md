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


# Chapter 2

## Call-site

Location where function is called (not where it's declared).

Only thing that matters for `this` binding.

## Nothing But Rules

### Default Binding

When plain, un-decorated function call.

`this` defaults to global object.

In `strict mode`, if `this` is not defined then it defaults to `undefined`.

`strict mode` state of contents of function determine whether or not `this` defaults to global object.

`strict mode` state of call-site of function is irrelevant.

### Implicit Binding

Ex: `obj.foo()`. `obj` is the context object and `this` will be set to it.

Only top/last level of object property refernce chain matters to call-site.

Ex: `obj1.obj2.foo()`. `this` is set to `obj2`.

#### Implicitly Lost

Ex from chapter: `var bar = obj.foo; bar();` `this` is NOT `obj`.

Same is the case for: `doFoo(obj.foo);` because passing argument is just an implicit assignment.

Additionally, higher order functions can intentionally change `this` for the call of the callback. Ex: Event handlers in JS libraries may force `this` to point to DOM element that triggered the event.

### Explicit Binding

Use `call(...)` and `apply(...)`. If passed a primitive value (of type `string`, `boolean`, or `number`) as `this` binding, the primitive will be "boxed"/wrapped in its object-form (`new String(...)`, etc).

Explicit binding itself doesn't prevent losing `this` or having it changed by a framework.

#### Hard Binding

Variation pattern around explicit binding works. See example.

Hard binding common pattern. Provided as built-in utility as of ES5.

`Function.prototype.bind`, which returns new function that is hard-coded to call original function with specified `this` context.

**Note:** In ES6 `bind(...)` has a `.name` property. Ex: `bar = foo.bind(...)` should have `bar.name` value of `"bound foo"` and is the function call name that should show up in a stack trace.

#### API Call "Contexts"

Many libraries' functions & JS built-in functions provide optional "context" parameter to use instead of `bind(...)`.

### `new` Binding

Not really "contstructor functions", but rather construction calls of functions.

When function invoked with `new`:

1. A new object created/constructed.

2. The new object is `[[Prototype]]`-linked.

3. The new object is set as the `this` binding for that function call.

4. The `new`-invoked function call will automatically return the new object, UNLESS function returns its own alternate object.

## Everything In Order

### Determining `this`

Precedence: new > explicit > implicit > default.

## Binding Exceptions

### Ignored `this`

If `call`, `apply`, or `bind` are passed values of `null` or `undefined`, those values are effectively ignored.

Can be dangerous if used against a function call that does make a `this` reference. Ex: third-party library function. Very difficult to diagnose bugs.

#### Safer `this`

Set up object for `this` that won't cause problematic side effects.

DMZ (de-militarized zone) object -- completely empty, non-delegated object (see Chs 5 & 6).

Easiest way to set up totally empty object is `Object.create(null)` (Ch 5).

Similar to `{}` but without delegation to `Object.prototype`.

### Indirection

Ex: `(p.foo = o.foo)();` default binding.

## Review (TL;DR)

Apply rules (in this order of precedence) to call-site to determine `this`:

1. Called with `new`? Use the newly constructed object.

2. Called with `call` or `apply` (or `bind`)? Use the specified object.

3. Called with a context object owning the call? Use that context object.

4. Default: `undefined` in `strict mode`, global object otherwise.