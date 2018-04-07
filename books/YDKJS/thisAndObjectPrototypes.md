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


# Chapter 3

## Syntax

Declarative/literal form (`obj = {...}`).

Constructed form (`obj = new Object()`).

## Type

Objects are one of the 6 primary types in JS (called "language types" in the specification).

* `string`
* `number`
* `boolean`
* `null`
* `undefined`
* `object`

### Built-in Objects

* `String`
* `Number`
* `Boolean`
* `Object`
* `Function`
* `Array`
* `Date`
* `RegExp`
* `Error`

Each can be used as constructor (that is, called with `new`).

## Contents

Property names are always strings. If any other value is used, it will first be converted to a string. This includes numbers.

### Computed Property Names

Added in ES6. Most common usage will probably be for ES6 `Symbol`s.

### Property vs. Method

Functions never "belong" to objects (compared to methods of classes in other languages).

### Arrays

Can add named properties. However, if property name _looks_ like a number, it will become numeric index and modify array contents.

### Duplicating Objects

It's not clear what should be the default algorithm. Ex: shallow vs deep? Circular references?

ES6 has defined `Object.assign(...)` to make shallow copy. The only "property descriptor" that is copied is the `value` property.

### Property Descriptors

As of ES5, all properties described in terms of a property descriptor.

Two main property descriptors: data descriptor and accessor descriptor.

Data descriptor properties: `value`, `writable`, `enumerable`, `configurable`.

#### Writable

Controls ability to change the `value` property. If try to change when `writable: false` then will silently fail, or if in `strict mode` will get a `TypeError`.

#### Configurable

Controls ability to modify descriptor definition. Setting to `false` is a one-way action and cannot be undone.

Exception: If `false`, `writable` can still be changed from `true` to `false`, but not back to `true` if already `false`.

`configurable: false` also prevents use of `delete` operator to remove an existing property. Silent fail.

#### Enumerable

Controls if property will show up in certain object-property enumerations, such as `for...in` loop.

### Immutability

The following affect only the object and direct property characteristics, not references to other objects.

#### Object Constant

Set both `writable` and `configurable` to `false`.

#### Prevent Extensions

To prevent object from having new properties added to it, call `Object.preventExtensions(...)`.

Fails silently, unless in `strict mode`, then `TypeError`.

#### Seal

`Object.seal(...)` takes existing object and essentially calls `Object.preventExtensions(...)` AND marks all its existing properties as `configurable: false`.

#### Freeze

`Object.freeze(...)` takes existing object and essentially calls `Object.seal(...)` AND marks all its existing properties as `writable: false`.

### `[[Get]]`

A variable reference that cannot be resolved results in `ReferenceError`.

However, when accessing a property, code performs a `[[Get]]` operation. If it can't find it, then instead of throwing a `ReferenceError`, it will return the value `undefined`.

### `[[Put]]`

Operation invoked when assigning to a property.

If property is present:

1. Is property an accessor descriptor? Call the setter, if any.

2. Is property a data descriptor with `writable: false`? Faily silently, or if `strict mode` throw `TypeError`.

3. Otherwise, set value to existing property as normal.

If proeprty not yet present, `[[Put]]` is more nuanced and complex.

### Getters & Setters

Way to override part of the default operations `[[Get]]` and `[[Set]]`.

### Existence

With property access like `myObject.a` that results in `undefined`, how to tell if it was explicitly set to `undefined` or if it doesn't exist?

`in` checks the object AND the `[[Prototype]]` chain.

`Object.prototype.hasOwnProperty(...)` checks only the object.

#### Enumeration

**Note:** `for...in` loops on arrays enumerate over any enumerable properties in addition to the numeric indices. Good idea to only use on objects.

`Object.prototype.propertyIsEnumerable(...)` checks only the object.

`Object.keys(...)` returns array of all enumerable properties, checks only the object.

`Object.getOwnPropertyNames(...)` returns array of all properties, checks only the object.

## Iteration

ES6 adds `for...of` loop that allows iteration over _values_ instead of properties.

`for...of` asks for `@@iterator`, which arrays have built-in.

Can iterate over other data structures aside from arrays, like objects, by creating a custom @@iterator.


# Chapter 4

## Mixins

### Explicit Mixins

#### "Polymorphism" Revisited

Explicit pseudo-polymorphism should be avoided whenever possible because complex and harder to maintain/read.

#### Mixing Copies

Even manually copying functions (mixins) from one object to another does emulate the real duplication from class to instance in class-oriented languages. It's just a duplicated reference.

## Review (TL;DR)

Classes mean copies.

JS does not automatically create copies as classes imply.

Mixin pattern _sort of_ emulates classes, but usually leads to brittle syntax like expilcit pseudo-polymorphism (`OtherObj.methodName.call(this, ...)`), often leading to harder to understand/maintain code.


# Chapter 5

## `[[Prototype]]`

### `Object.prototype`

Is the top-end of every _normal_ `[[Prototype]]` chain.

### Setting & Shadowing Properties

Assignment of a property does NOT always result in shadowing if the property already exists higher on the `[[Prototype]]` chain.

Shadowing usually more complicated and nuanced than it's worth.

Look at example of subtle implicit shadowing.

## "Class"

### Mechanics

#### "Constructor" Redux

`.constructor` is extremely unreiable, and an unsafe reference to rely upon in your code.

## "(Prototypal) Inerhitance"

ES6 adds `Object.setPrototypeOf(...)` helper utility to modify the linkage of an existing object.

### Inspecting "Class" Relationships

`instanceof`, `Object.prototype.isPrototypeOf()`, `Object.getPrototypeOf()`, `Object.prototype.__proto__`

Generally, should not change the `[[Prototype]]` of an existing object.

## Object Links

### `Create()`ing Links

`Object.create(...)` give power of the `[[Prototype]]` mechanism without unnecessary complication of `new` functions acting as classes and constructor calls, confusing `.prototype` and `.constructor` references, etc.

**Note:** `Object.create(null)` used for storing data in a purely flat storage with no possible surprise effects from delegated properties/functions on `[[Prototype]]` chain, because it has none. Often called "dictionaries".

### Links As Fallbacks?

Less surprising/confusing if it's an internal implementation detail than if it's plainly exposed in your API design. Use delegation design pattern?
