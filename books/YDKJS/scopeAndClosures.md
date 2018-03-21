# Chapter 1

## Compiler Theory

1. Tokenizing/Lexing: break characters into chunks, called tokens.

2. Parsing: turn tokens into Abstract Syntax Tree (AST).

3. Code-Generation: turn AST into executable code.

## Understanding Scope

LHS lookup: find the address of some memory location to assign a value to that memory locaiton.

RHS lookup: find the value held in some memory location.

### Quiz

count number of LHS and RHS lookups in the following code:

    function foo(a) {
	  var b = a;
      return a + b;
    }

	var c = foo( 2 );

`var c = foo( 2 );` LHS lookup for `c`, RHS lookup for `foo( 2 )`.

`function foo(a) {` above line calls `foo` with argument `2`, which is assigned to parameter `a` using a LHS lookup.

`var b = a;` LHS lookup for `b`, RHS lookup for `a`.

`return a + b;` RHS lookup for `a` and `b`.

In total, 3 LHS lookups, 4 RHS lookups.

## Errors

    function foo(a) {
      console.log( a + b );
      b = a;
    }

    foo( 2 );

This code will result in `ReferenceError: b is not defined`.

What if the order of the two lines within foo was switched?

    function foo(a) {
      b = a;
      console.log( a + b );
    }

    foo( 2 );

There would be no error, `4` would be printed. Even though `b` is never explicitly declared, `b` would automatically be declared in the global scope, not the scope of `foo`. However, if using strict mode, it would again result in `ReferenceError: b is not defined`.

Now back to the original, but with an explicit declaration of `b`.

    function foo(a) {
      console.log( a + b );
      var b = a;
    }

    foo( 2 );

`NaN` would be printed. No error because `b` is declared. `b` is not assigned a value before the RHS reference, so in the line `console.log( a + b );`, the value of `a` is `2` and the value of `b` is `undefined`, and their sum is `NaN`.


# Chapter 2

Lexical scope is defined at write time.

## Cheating Lexical

### `eval`

Takes string argument and treats it as code.

Usually used to execute dynamically created code, which can modify author-time lexical scope.

If used in strict mode, `eval` operates in its own lexical scope, so declarations inside it do not modify enclosing scope.

Avoid dynamically generating code, rarely needed.

### `with`

Deprecated.

### Performance

Using `eval` and `with` means JS engine doesn't use optimizations, so code runs slower.

Don't use them.


# Chapter 3

## Hiding In Plain Scope

"Principle of Least Privilege" or "Least Authority" or "Least Exposure".
- only expose what is necessary.

### Collision Avoidance

Is a benefit of "hiding" variables and functions inside a scope.

#### Global "Namespaces"

Libraries can collide if they don't hide their private variables and functions.

To avoid this, libraries usually create single variable (say object) with unique name in global scope. This object is used as a "namespace" where exposures of functionality made through properties, instead of lexically scoped identifiers.

#### Module Management

Using a dependency manager, no libraries add identifiers to global scope, but have their identifiers imported into another specific scope.

## Functions As Scopes

If "function" is very first thing in a statement, it's a function declaration. Otherwise, it's a function expression.

Function declarations are bound in enclosing scope.
Function expressions are not.

Function expressions, not declarations, can be anonymous.

Drawbacks of anonymous function expressions:

1. No useful name to display in stack traces, which can make debugging more difficult.

2. Can't refer to itself (say for recursion). Need `arguments.callee` which is deprecated. Event handler function would need to reference itself to unbind itself.

3. Omits name that can make code more understandable.

Best practice: always name function expressions.

### Invoking Function Expressions Immediately

(function(){..})() or (function(){..}()), stylistic choice.

In UMD (Universal Module Definition), function to execute given as argument to IIFE.

## Blocks As Scopes

### `try/catch`

The variable declaration in the `catch` clause is block-scoped to the `catch` block. Linters may complain about re-defintion if using two or more `catch` clauses in same scope with same error variable name, even though they are safely block-scoped.

### `let`

Introduced in ES6.

`let` keyword attaches variable declaration to scope of whatever block (commonly a {..} pair) it's contained in.

Can cause confusion if move blocks around as develop code.

Creating explicit blocks can make it more obvious.

Declarations with `let` will not hoist to the entire scope, do not "exist" until declaration statement.

#### Garbage Collection

Block-scoping can allow garbage collection that may otherwise not happen due to a closure.

#### `let` loops

`for (let i .. )` re-binds `i` to each iteration of loop.

### `const`

Creates a block-scoped variable with fixed value. Attempting to change causes error.

## Review

Block scope shouldn't be used as outright replacement of `var` function scope.

Use function- and block-scope techniques to improve readability/maintainability.


# Chapter 4

## The Compiler Strikes Again

JS program not interpreted line-by-line, top-down.

Declarations (variables and functions) processed first.

Hoisting is per-scope.

Function expressions not hoisted.

## Functions First

Functions hoisted first, then variables.

Be careful about duplicate declarations, especially mixed between normal variable and function declarations.


# Chapter 5

Closure is when a function is able to remember and access its lexical scope even when that function is executing outside its lexical scope.

## Loops + Closure

Linters often complain about functions inside loops because the mistakes of not understanding closure are very common among developers.

## Modules

Function that returns public API, whether it be an object with properties/methods or just a function.

Two "requirements" for module pattern to be exercised:

1. There must be an outer enclosing function which must be invoked at least once (each time creates new module instance).

2. Enclosing function must return back at least one inner function, so that this inner function has closure over the private scope and can access and/or modify that private state.

## Modern Modules

Module managers wrap modules using this same module definition pattern to create a friendly API.

## Future Modules

ES6 adds support for modules. When loaded via the module system, ES6 treats a file as a separate module. Each module can both import other modules and export their own public API members.

**Note:** Function-based modules aren't statically recognized patter, which means a module's API can be modified during runtime.

ES6 Module APIs are static (don't change at run-time). So compiler checks during (file loading and) compilation that a reference to a member of an imported module's API actually exists, and if it doesn't, the compiler thors an error, rather than waiting for dynamic run-time resolution (and errors, if any).

`import` imports one or more members for a module's API into the current scope, each bound to a variable.

`module` imports an entire module API to a bound variable.

`export` exports an identifier (variable, function) to the public API for the current module.

These operators can be used as many times in a module's definition as necessary.


# Appendix A

Lexical scope determined by where functions and scopes are declared.

Dynamic scope determined by where they are called from.

So, scope chained based on call-stack, not nesting of scopes in code.

Lexical scope is write-time, dynamic scope is run-time.

`this` cares how a function was called, so it is closely related to the idea of dynamic scoping.


# Appendix C

Arrow functions do not behave like normal functions when it comes to their `this` binding. They discard normal rules and take on the `this` value of immediate lexical enclosing scope.

Author (Kyle Simpson) thinks arrow function codes into the language syntax a common mistake of confusing "this binding" rules with "lexical scope" rules.

Additional downside of arrow functions is that they are anonymous.

Perhaps better to use `Function.prototype.bind()` to properly use the `this` mechanism.
