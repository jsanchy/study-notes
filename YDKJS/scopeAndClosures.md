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