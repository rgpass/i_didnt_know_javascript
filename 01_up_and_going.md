# I Didn't Know JavaScript: Up & Going

## Chapter 1: Into Programming

Before code can be executed by the computer, it needs to be translated into commands the computer understands. This is done via interpreting or compiling.

Interpreted: the code is translated from top to bottom, once every time the program is run

Compiled: the translation is done ahead of time, so when it's run later it's already in commands the computer understands

JS compiles the code on the fly then runs it immediately.

When evaluating `'99.99' == 99.99`, the left-hand side is implicitly coerced to a number

`for` loops have three clauses: the initialization clause (`var i = 0`), the conditional test clause (`i <= friends.length`), and the update clause (`i++`).

*Scope* is technically called *lexical scope*. Scope: a collection of variables plus rules for how those variables are accessed by name. Lexical scope: code in one scope can access variables of either itself or any scope outside of it.

## Chapter 2: Into JavaScript

```javascript
var a = null;
typeof a; // "object"

// Thus, to test if null:
function isNull(a) {
  return a === null;
}

isNull(a); // true
var b; // equivalent to var b = undefined;
isNull(b); // false
```

It is possible to add properties to a function, however this is not commonly used.

```javascript
function myFunc() {}

myFunc.bar = 'example property value';
myFunc.bar; // "example property value"
```

When calling a built-in method on a primitive value, JS "boxes" the value to its object wrapper counterpart.

```javascript
var str = 'my string'; // primitive
str.toUpperCase(); // JS "boxes" the primitive to its String object counterpart (done under the hood) aka its "native" counterpart
```

Primitives: just a value, does not have properties. In JS, they are `undefined`, `null`, strings, numbers, and booleans (plus symbols in ES6). Everything else is an object.

Natives: standard built-in objects, often "boxed" around primitives to be able to call properties (such as `myStr.length`)

All falsey values:
* `""` (empty string)
* `0`, `-0`, `NaN`
* `null` and `undefined`
* `false`

`===` is called "strict equality"
`==` is called "loose equality"

Coercion is OK as long as neither value will ever be `true`, `false`, `0`, `""`, or `[]`. If you can be certain about the values, use implicit coercion.

Arrays are coerced into comma separated strings, rather than checked by reference.

```javascript
var arr1 = [1, 2, 3];
var arr2 = "1,2,3";

arr1 == arr2; // true
arr1 === arr2; // false
```

Inequality operators (ex: `<`, `>=`) apply similar coercion rules, but don't have an equivalent to strict equality. Thus, coercion can cause some weird results.

```javascript
var a = 29;
var b = "30";
var c = "31";
var d = "blah";

b < c; // true, both are strings, so it is compared lexographically aka alphabetically
a < b; // true, if one or both are not strings, they are both coerced to numbers, then compared to numerically

a < d; // false
a > d; // false
a == d; // false
// These are all false because Number('my string') is NaN and all comparisons with NaN are false, even NaN == NaN;
```

Strict mode: safer code, more optimizable, use it all the time, if it breaks things then fix those things and keep strict mode on

Polyfill: using the definition of a new feature and making an equivalent piece of code to work in older JS envs

```javascript
// Polyfilling ES6's Number.isNan(..)

if (!Number.isNaN) {
  Number.isNaN = function isNaN(x) {
    return x !== x; // since NaN is the only value not equal to itself
  }
}
```

Do not make your own polyfills as messing up slightly will create brittle code that will be hard to update later. Instead, use ES5 and ES6 shims.

Transpiling: transforming + compiling. Used when there is a new syntax that cannot be polyfilled.

When accessing JS in a browser, the `document` object is available. This is called a "host object".

Other host objects are I/O, such as `alert(..)` which is provided by the browser, not JS.

Objects Linked to Other Objects (OLOO) is also called "behavior delegation"

## Chapter 3: Into YDKJS

Promises: time-independent wrapper of a future value

Generators: can be paused at `yield` points and are resumed asynchronously later. This allows code to be written similar to synchronous code. Removes the issue of non-linear and non-local-jump code that make it hard to reason about.

