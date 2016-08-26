# I Didn't Know JavaScript: Scope & Closures

## Chapter 1: What is Scope?

In traditional compiled-languages, your code will undergo three steps prior to execution called "compilation".

1. Tokenizing/Lexing: breaking up a string of characters into meaningful chunks called tokens. `var a = 2;` gets broken up into the following tokens: `var`, `a`, `=`, `2`, and `;`.
2. Parsing: takes this stream (array) of tokens and converts it into a tree of nested elements which represent the grammatical structure of the program. This tree is called Abstract Syntax Tree (AST). For the `var a = 2;` example, this would look like: `VariableDeclaration > Identifier (a), AssignmentExpression > NumericLiteral (2)`.
3. Code Generation: turns AST into executable code and handles reserving memory

In JS, compilation happens just microseconds before execution.

### Understanding Scope

There are three major actors pertaining to scope. Think of scope as a conversation.

1. Engine: responsible for start-to-finish execution of the JS code
2. Compiler: friend of the Engine, handles all the work of the parsing and code-generation (as discussed above)
3. Scope: friend of the Engine, collects and updates a list of all declared identifiers (variables) and enforces the rules of how these identifiers can be accessed to the code currently being executed

`var a = 2;` is two parts from the Engine's perspective.

1. `var a` Compiler asks Scope to see if a variable `a` exists in the scope collection. If it does exist, Compiler ignores it and moves on. If it doesn't, Compiler asks Scope to declare a new variable called `a` for that scope collection.
2. Compiler produces code for Engine to later execute to handle the `a = 2` assignment. When Engine runs the code, it asks Scope if an `a` variable exists in the current scope collection. If it does, Engine uses that variable. If it does not, Engine looks in the nested scope for the variable. If it doesn't exist anywhere, Engine will raise a `ReferenceError`.

There are two types of look ups. Left-Hand Side (LHS) and Right-Hand Side (RHS). The difference typically happens in assignment expressions. LHS -- "who's being assigned a value", RHS -- "what is the value"

```javascript
var a; // LHS for a
var b = a + 2; // LHS for b, RHS for a
```

RHS is simply a look up of the value of a variable. LHS actually looks up variable container itself.

When calling a function, variables defined in the function's parameters are implicitly assigned and thus execute a LHS look-up.

```javascript
function logIt(a) {
  // Implicit a = 29 LHS look-up is done when executed
  console.log(a); // RHS look-up for console, then a property-resolution is done to see if log exists, finally a RHS is done for a
}

logIt(29);
```

In `logIt` above, the Compiler does not assign the function to the value of `logIt`, rather it handles both the declaration and the value definition during code generation.

ReferenceError: scope related issue
TypeError: data type related issue (such as executing a non-function or retrieving a property of `undefined`)

## Chapter 2: Lexical Scope

Lexical scope: is named this because it is defined at lexing time (see notes above). Lexical scope is based on where you wrote variables and blocks.

It is possible to bypass lexical scoping, however should rarely (if ever) be used. It leads to significantly poorer performance. This is because the Engine optimizes look-ups when compiling. When it is cheated, the Engine has to assume all of its look-ups are incorrect.

Scope look-up stops once it finds the first match.

Shadowing: Having a variable named the same as one in a parent scope (where the inner scope shadows the outer)

The ways to cheat lexical scope are: `eval(..)` (including creating dynamically-generated functions) and `with`

```javascript
// eval is used under the hood when using new Function
var dynamicFunc = new Function('console.log("Hello");');
dynamicFunc(); // "Hello"
```

In strict mode, `eval` does not modify the enclosing scope.

`with` is a deprecated feature that induces global variable side-effects with the minor benefit of saving keystrokes. It essentially creates its own scope based off an input object.

## Chapter 3: Function vs. Block Scope

Similar to how creating a function creates a new nested scope, it's equally as powerful to think about how new functions can hide variables and functions. This aligns with the Principle of Least Privilege (aka Least Exposure aka Least Authority) -- only expose what is minimally necessary and hide everything else.

Hiding increases readability, lowers user-created side-effects, and lowers variable collisions.

Module management is a tool (or set of tools) that requires identifiers to be explicitly imported into separate scopes. You can achieve the same results as these dependency managers by hiding code via the module pattern.

The `()` around IIFEs turn it from a function declaration into a function-expression.

Block scope is in fact possible in ES3. It is possible in `with` blocks and `try/catch`.

```javascript
// This is similar to how Traceur does block scoping via let
try {
  throw undefined;
}
catch (a) {
  a = 2;
  console.log(a); // 2
}

console.log(a); // ReferenceError
```

`let` and `const` statements are not hoisted.

Another benefit of block scoping is to enable garbage collection.

In the example below, because both the `hugeDataSet` and the event listener are in the same scope, `hugeDataSet` will not be GC'ed because the Engine thinks that it may be used again.

```javascript
function processData() {
  // ...
}

var hugeDataSet = {};

var btn = document.getElementById('my-btn');
btn.addEventListener('click', function onBtnClick(evt) {
  console.log('clicked');
}, false);
```

Applying block scope, we can tell the GC'er to clean it up after use.

```javascript
function processData() {
  // ...
}

// Tells the Engine that it can go away after being processed
{
  let hugeDataSet = {};
  processData(hugeDataSet);
}

var btn = document.getElementById('my-btn');
btn.addEventListener('click', function onBtnClick(evt) {
  console.log('clicked');
}, false);
```

For `for` loops, the `i` actually re-binds it to each iteration. This is the equivalent of:

```javascript
{
  let j;
  for (j = 0; j < 10; j++) {
    let i = j; // this is re-bound every iteration -- talk about powerful for closures!
    console.log(i);
  }
}
```

## Chapter 4: Hoisting

`var a = 2;` is actually `var a;` and `a = 2` in the eyes of the Compiler.

```javascript
'use strict';
a = 2;
var a;
console.log(a); // 2
```

Hoisting: when compiling, all variable declarations and function declarations are processed first.

Function expressions are not hoisted.

```javascript
foo(); // TypeError: foo is not a function
var foo = function bar() {
  console.log('from bar');
};

// Translates to:
var foo;
// Try calling bar on this line and you will get: bar is not defined -- see below
foo();
foo = function bar() {};

// DOES NOT TRANSLATE TO
var foo;
function bar() {};
foo();
foo = bar;
```

Functions are hoisted first, then variables.

Do not create functions in blocks. Current behavior is that the last definition, regardless of the result of a block/conditional, will always override.

```javascript
var isTrue = true;

printStuff(); // "Override" -- or in some Engines will return a TypeError

if (isTrue) {
  function printStuff() {
    console.log('Original');
  }
}
else {
  function printStuff() {
    console.log('Override');
  }
}
```

## Chapter 5: Scope Closure

Closure: when a function is able to remember and access its lexical scope even when that function is executing outside its lexical scope.

Nested scopes are said to close over the scope of its parent.

```javascript
function foo() {
  var a = 2;

  function bar() {
    console.log(a); // 2;
  }

  // Terminology: bar() closes over the scope of foo() because it is nested inside of foo()
  bar();
}

foo();
```

Closure does not let the Garbage Collector clean it up.

The reference to the scope that's closed over is what closure is.

IIFE's are not really closure because they are not executed outside of their lexical scope. Although they do create scope, they're executed in their current scope.

Using IIFE's inside of `for` loops to create scope (and apply closure) is quite powerful.

```javascript
for (var i = 0; i <= 5; i++) {
  // IIFE is needed or else when printing i, it will be the end value of i (which will be 6)
  (function iteratorFn(i) {
    setTimeout(function printStuff() {
      console.log(i);
    }, 500)
  })(i);
}
```

`let` re-binds the iterator in a `for` loop, allowing us to close over the block's scope -- much simpler than the IIFE example above.

```javascript
for (var i = 0; i <= 5; i++) {
  let j = i; // let hijacks a block and turns the block into a scope that can be closed over
  setTimeout(function printStuff() {
    console.log(j);
  }, 500);
}

// Even cleaner -- let has a special behavior: it re-binds every iteration
for (let i = 0; i <= 5; i++) {
  setTimeout(function printStuff() {
    console.log(i);
  }, 500);
}
```

Module pattern has two requirements: (1) it must be outer enclosing function that is envoked at least once, (2) the enclosing function must return back at least one function

Functions that return an object with just data aren't really modules.

ES6 adds first-class syntax for the concept of modules. ES6 treats a file as a separate module. Each file is enclosed in scope closure.

Because the ES6 module API is static (i.e. doesn't change at run time), it can determine if a reference to a module exists or not at compile-time, throwing an error before run-time. English: ES6 module system throws an error if something is missing prior to actually executing it.

## Appendix A: Dynamic Scope

Lexical scope cares about where functions and scopes were declared.

Dynamic scope cares about where the function was called and the current call stack.

In the example below, lexical scope will print `2`, while dynamic scope will print `3`.

```javascript
function foo() {
  console.log(a);
}

function bar() {
  var a = 3;
  foo();
}

var a = 2;
bar();
```

`this` is similar to dynamic scope as it cares about how the function was called rather than where it was declared.

## Appendix B: Polyfilling Block Scope

In pre-ES6, the best way to get block scope is to use `try..catch`.

```javascript
try {
  throw 29;
}
catch(a) {
  console.log(a); // 2
}
// or
try {
  throw undefined;
}
catch(a) {
  a = 2;
  console.log(a); // 2
}
```

## Appendix C: Lexical-this

ES6's fat-arrow do not behave like normal functions and do not follow the normal `this` binding rules. Instead, it uses the `this` value of the immediate lexical enclosing scope.

```javascript
var obj = {
  count: 0,
  printFn() {
    setTimeout(() => {
      console.log(this.count);
    }, 0);
  }
}

obj.printFn(); // 0
```

A much less confusing approach is to use `bind`. This way you don't mix up lexical scoping with `this` rules and don't need `var self = this`.

```javascript
var obj = {
  count: 0,
  printFn: function printFn() {
    setTimeout(function wrapper() {
      console.log(this.count);
    }.bind(this), 0);
  }
}

obj.printFn();
```

Fat-arrow is not just about saving keystrokes. It is an intentional behavioral difference.
