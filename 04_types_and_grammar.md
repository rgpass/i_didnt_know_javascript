# I Didn't Know JavaScript: Types & Grammar

## Chapter 1: Types

JS describes types as values that are distinguished from other values from the perspective of the engine and the developer. In other words, both the engine and the developer treat `29` differently than `'29'`.

Types: `null`, `undefined`, string, boolean, number, object (plus symbol in ES6)

You can use `typeof` to determine the value's type, except when testing for `null`. It can also return `"function"`.

```javascript
var num = 2;
typeof num; // "number"
typeof null; // "object"

function foo() {}
typeof foo; // "function"
```

Dynamically typed: values have types, variables don't

Statically typed: variables and values have types, strictly enforced

Undefined: has been declared in the scope but does not have a value assigned to it

Undeclared: has not been declared in the scope

```javascript
var a;

a; // undefined
b; // ReferenceError: b is not defined -- "not declared" or "not found" would be less confusing

typeof a; // "undefined"
typeof b; // "undefined" -- adds to the confusion
```

As a safety check, use `typeof`.

```javascript
// Throws an error
if (UNDECLARED_VAR_NAME) {
  console.log('Executing...');
}

// Safely executes
if (typeof UNDECLARED_VAR_NAME !== 'undefined') {
  console.log('Executing...');
}
```

## Chapter 2: Values

Warning: using `delete` on an array will remove the value but will not update the `length` property.

Arrays can also have properties added to them -- after all, they are just objects. If the string value of the property being added can be coerced into a base-10 number, it will assume you want a number. Best to avoid this functionality, however.

There are array-like values, such as DOM query operations. These need to be coerced into real arrays before utilities can be called. This is typically done via the `slice(..)` utility.

```javascript
function foo() {
  // ES5 way
  var args = Array.prototype.slice.call(arguments);

  // ES6 way
  var args = Array.from(arguments);

  args.push('dude'); // the push function is only available on real arrays
  console.log(args);
}

foo('yeah'); // ["yeah", "dude"]

// Another ES6 strategy
function bar(...args) {
  args.push('dude');
  console.log(args);
}

bar('yeah'); // ["yeah", "dude"]
```

It's a common misconception that strings are just arrays of letters. This is not true as strings are immutable and arrays are mutable.

```javascript
var a = 'foo';
var b = ['f', 'o', 'o'];

a[0] = 'F';
b[0] = 'F';

a; // "foo"
b; // ["F", "o", "o"]
```

It is possible to borrow non-mutation array functions.

```javascript
a.join; // undefined
a.map; // undefined

var c = Array.prototype.join.call(a, '-');
c; // "f-o-o"

var d = Array.prototype.map.call(a, function(v) {
  return v.toUpperCase() + '.';
}).join('');
d; // "F.O.O."
```

Reversing a string: convert to an array, reverse, then join into a string. This doesn't work for complex strings -- anything that contains astral symbols, multibyte characters, etc. Will need sophisticated libraries to solve this.

JS doesn't have true integers. `42.0` is just as much of an integer in JS as `42`.

JS's numbers are based off the floating-point standard. JS specifically uses the double precision format aka 64-bit binary.

Both leading and trailing `0`'s are optional in JS. It's best to include them, however.

```javascript
var a = 0.42;
var b = .42;

var c = 42.0;
var d = 42.;
```

JS has lots of `Number.prototype` helpers, including exponents, determining the number of decimals, etc.

```javascript
var a = 5E10;
a; // 50000000000
a.toExponential(); // "5e+10"

var b = 30.567;
// toFixed sets the number of decimal places
b.toFixed(0); // "31"
b.toFixed(1); // "30.6"
b.toFixed(5); // "30.56700"

// toPrecision sets the number of significant figures
b.toPrecision(1); // "3e+1"
b.toPrecision(5); // "30.567"
b.toPrecision(7); // "30.56700"
```

Since `.` is a valid numerical character AND how to access properties, there can be some weird errors. The `.` is first interpreted as a numerical character.

```javascript
30.toFixed(2); // Uncaught SyntaxError: Invalid or unexpected token

(30).toFixed(2); // "30.00"
30..toFixed(2); // "30.00"
30 .toFixed(2); // "30.00" -- valid, but super confusing coding style
```

JS allows for hexadecimal and octal. However, in ES6 the ES5 octal literal syntax will not be allowed in strict mode.

```javascript
0xf3; // 243 -- in hexadecimal
0XF3; // same

0363; // 243 -- ES5 syntax for octal
0o363; // 243 -- ES6 syntax
0O363; // same, but harder to read

0b11110011; // 243 -- binary
0B11110011; // same
```

The downside of using binary floating-point numbers of any language is that there can be a bit of confusion.

```javascript
1 + 2 === 3; // true
0.1 + 0.2 === 0.3; // false
0.1 + 0.2; // 0.30000000000000004 -- this is because the representations of 0.1 and 0.2 are not exact but instead are really close to those values
```

The most common workaround to this is to allow a rounding error as a tolerance for comparison. This is often called "machine epsilon" which typically has a value of `2^-52`. This is defined in ES6 as `Number.EPSILON`. The ES5 polyfill:

```javascript
if (!Number.EPSILON) {
  Number.EPSILON = Math.pow(2, -52);
}

// Now we can compare
function numsCloseEnoughToEqual(n1, n2) {
  return Math.abs(n1 - n2) < Number.EPSILON;
}
```

The max number that can be safely called is `2^53 - 1`. There's also a min safe number. ES6 has `Number.MAX_SAFE_INTEGER` and a min equivalent.

The reason there are safe numbers is that for very large numbers, JS has to store them as strings. Doing operations on these will often require a library for big numbers.

Bit-wise operators need a 32-bit signed integers -- which is much smaller. Can force a number into a 32-bit signed integer (stored in `a`) via `a | 0`.

Developers tend to differentiate between `null` and `undefined` in one of two ways.

* `undefined` is a missing value
* `null` is an empty value

or

* `undefined` hasn't had a value yet
* `null` had a value and doesn't anymore

Since it's possible to override the value of `undefined` (ill-advised!), one way to get it is to use `void 0` (or any other value -- `void` always returns `undefined`).

`NaN` stands for "Not a Number" but this is unclear. A better translation is "invalid number, failed number, or bad number".

```javascript
typeof NaN; // "number"
```

Old ES versions had a flaw in `isNaN(..)` in that it literally took it as "is not a number". This is fixed in ES6.

```javascript
var a = 2 / 'foo';
var b = 'foo';

// Pre-ES6
isNaN(a); // true
isNaN(b); // true

// ES6
Number.isNaN(a); // true
Number.isNaN(b); // false
```

`NaN` is the only value that never equals itself.

If you go over the max number, you'll hit either `Infinity` or below the min number gets you `-Infinity`. Try `1 / 0` or `-1 / 0`.

JS has both a `+0` and a `-0`. `-0` can arise from mathematical operations such as `0 * -3`. If stringified, it will return `"0"`. Doing the reverse works (`Number('-0')`), however.

Comparisons also lie `0 === -0; // true`. Thus, if the sign is important (such as for gaming engines), need to use:

```javascript
function isNegZero(n) {
  n = Number( n );
  return (n === 0) && (1 / n === -Infinity);
}

isNegZero( -0 );    // true
isNegZero( 0 / -3 );  // true
isNegZero( 0 );     // false
```

When checking for these special cases in ES6, use `Object.is(..)`.

To pass a scalar primitive value in way where its value updates can be seen, wrap it in an object.

```javascript
function foo(wrapper) {
  wrapper.a = 30;
}

var myObj = { a: 2 };
foo(myObj);
myObj.a; // 30
```

## Chapter 3: Natives

Most commonly used natives:
* `String()`
* `Number()`
* `Boolean()`
* `Array()`
* `Object()`
* `Function()`
* `RegExp()`
* `Date()`
* `Error()`
* `Symbol()` -- added in ES6!

They're actually built-in functions.

Using a constructor call with these will result in an object wrapper around the primitive. It will be an object subtype.

```javascript
var a = new String('yeah');
typeof a; // "object"
a instanceof String; // true
Object.prototype.toString.call(a); // "[object String]" -- this is the way to find an object's classification, accesses the object's [[Class]] property
console.log(a); // results depend on the browser
```

Primitive values do not have properties or methods. To access these, we need an object wrapper around these primitives. Thankfully, JS automatically boxes primitives.

To see an example of boxed wrappers:

```javascript
Object.prototype.toString.call(30); // "[object Number]"
```

JS engines are optimized to box, so pre-boxing values yourself (aka using object forms of primitives) it will actually execute slower.

Can manually box using `Object(..)`.

```javascript
var a = 'abc';
var b = new String(a);
var c = Object(a);

var a = "abc";
var b = new String( a );
var c = Object( a );

typeof a; // "string"
typeof b; // "object"
typeof c; // "object"

b instanceof String; // true
c instanceof String; // true

Object.prototype.toString.call( b ); // "[object String]"
Object.prototype.toString.call( c ); // "[object String]"
```

Use `.valueOf()` to get the value of a boxed primitive. This can also happen implicitly during coercion.

It's typically ill-advised to use the constructor form over the literal form for creating arrays, objects, functions, and RegExs.

Using the `Array` constructor, having one argument will set its length. Having more than one will set its contents. This weird way of distinguishing is why it's ill-advised.

```javascript
var arr1 = new Array(5);
var arr2 = new Array(5, 2);

arr1; // [undefined × 5] -- this is called a sparsed array since it's missing a value in at least one spot
arr2; // [5, 2]
```

Sparsed arrays are parsed differently depending on the browser. Also, it's not really a bunch of `undefined` values, the slots actually don't even exist. This makes it a nightmare to troubleshoot. Also, calling functions like `.map` will fail because the slots doesn't actually exist.

The best way to create an empty array: `var arr = Array.apply(null, { length: 3 });`

Sometimes `new RegExp` makes sense, such as dynamic pattern matching.

```javascript
var name = "Kyle";
var namePattern = new RegExp( "\\b(?:" + name + ")+\\b", "ig" );

var matches = someText.match( namePattern );
```

`Date` and `Error` require the constructor call.

Symbols are typically used to symbolize that a value is private. It will replace the `_` format.

## Chapter 4: Coercion

Changing types: when explicit it's called "type casting" and implicit is "coercion"

Coercion always results in a scalar primitive (string, number, boolean).

When calling `.toString()`, large numbers will show their exponent form and objects will show their `[[Class]]` (such as `"[object Number]"`).

If an object has a `.toString` method on it, that method will be called.

```javascript
var obj = {
  toString: function() {
    return 'oh yeah';
  }
};

String(obj); // "oh yeah"
```

When using `JSON.stringify(..)`, if a non-JSON-safe value is found (such as a symbol or function), its value is replaced by `null`.

When calling `JSON.stringify(obj)`, the object's `.toJSON()` function will be called first. If you have an unsafe object, make sure to have a `.toJSON` function defined for it.

`JSON.stringify` has a second argument called a replacer. If the replacer is an array, it will only return those values. If it is a function, it will loop through the properties and return those values.

```javascript
var obj = {
  a: 30,
  b: 'dude',
  c: [1, 2, 3]
};

JSON.stringify(obj, ['b']); // "{"b":"dude"}"
JSON.stringify(obj, function(key, value) {
  if (key !== 'b') {
    return value;
  }
}); // "{"a":30,"c":[1,2,3]}"
```

`JSON.stringify` takes a 3rd argument which is either a number of spaces to be used for indentation or a string to use for each indentation.

```javascript
JSON.stringify(obj, null, '---');
// "{
// ---"a": 30
// ...
// }"
```

When using `ToNumber` (typically via `Number(..)` or implicit coercion):
* `true` -> `1`
* `false` -> `0`
* `undefined` -> `NaN`
* `null` -> `0` -- weird!
* `[1]` -> first becomes the primitive value of `"1"` -> `1`

When coercing to a primitive first, it will first run `valueOf()` then try `toString()` and take the result from either of those and coerce it into a number.

```javascript
var obj1 = {
  valueOf: function() {
    return '30';
  }
};

var obj2 = {
  toString: function() {
    return '30';
  }
};

var arr = [2, 9];
arr.toString = function() {
  return this.join('');
};

Number(obj1); // 30
Number(obj2); // 30
Number(arr); // 29
```

In some languages `true` is equal to `1` and `false` is equal to `0`. This is not true in JS under-the-hood.

When coercing to a boolean, it works like this:

1. The value is one that can be coerced to false
2. Everything else

The JS spec has a narrow list of falsey values. If it's not in the list, it is coerced to a boolean.

All falsey values:
* `false`
* `undefined`
* `null`
* `''`
* `+0`, `-0`
* `NaN`

When using the native constructors, it creates an object, so it will be truthy. `var myBool = new Boolean(false)` will be truthy. In this case, `myBool` is called a "falsey object". A falsey object is an object that acts like a normal object, but when coerced to a boolean is `false`.

Browsers have created their own scenarios of what to do with falsey objects. These are different from the ECMAScript spec. In Chrome, the only way to get `false` from a falsey object is to run `myFalseyObj.valueOf()`.

Coercing between strings and numbers is done through `String(..)` and `Number(..)`. This does not use the `new` keyword, thus does not create object wrappers.

```javascript
var age = 30;
String(age);
age.toString();

var luckyNumber = '7';
Number(luckyNumber);
+luckyNumber; // unary operator, considered to be an acceptable form of explicit coercion in the JS open-source community except when next to adjacent to another operator
```

Do not put use the unary conversion when near operators. Confusion:

```javascript
var c, d;
c = '2';
d = 0;
d += c; // "02"
d =+ c; // 2
```

The unary operator is common with dates. To get the current date and time as a number, use `var timestamp = +new Date();`.

It's better to use `new Date(..).getTime()` for not-now timestamps and `Date.now()` for current timestamps.

`~` aka the tilde operator aka the bitwise NOT is often avoided.

Bitwise operators first convert to a 32-bit value by first doing a `ToNumber` then a `ToInt32` process. This is not technically coercion as the data type doesn't change, it has a different value.

The following are examples of explicit coercion into `ToInt32` operations.

```javascript
0 | -0;
0 | NaN;
0 | Infinity;
0 | -Infinity;
```

The `~` operator often returns a `-1` -- which can be found in things like `arr.indexOf(..)` if that value doesn't exist in the array. `-1` is called a sentinel value. This means it's given an arbitrary semantic meaning. In JS, this often means "not found".

`~` is similar to `!` but `!` also coerces into a boolean.

We often write code that checks for this sentinel value and it's kind of ugly.

```javascript
var msg = 'Hello World';

if (msg.indexOf('lo') >= 0) {
  console.log('yay');
}

if (msg.indexOf('lo') === -1) {
  console.log('try again'); // won't print
}

// Much cleaner to use the ~
if (~msg.indexOf('lo')) {
  console.log('yay2');
}

if (!~msg.indexOf('lo')) {
  console.log('try again 2'); // won't print
}
```

Some developers will use a double `~` to remove the decimals off of a number. This is sometimes said to be the same as `Math.floor(..)` but it is not. `~~` only works correctly with 32-bit values. Also, it doesn't work the same with negative numbers.

```javascript
Math.floor(-49.6); // -50
~~-49.6; // -49

// Little bit cleaner if looking to round down if positive and round up if negative
-49.6 | 0; // -49
```

Sometimes you need to use `~~` when considering operator precedence -- discussed more later.

```javascript
~~1E20 / 10;    // 166199296

1E20 | 0 / 10;    // 1661992960
(1E20 | 0) / 10;  // 166199296
```

When working with strings that start with numbers, you can either coerce or you can parse. Parsing is tolerant of non-numeric characters and coercion is not tolerant.

```javascript
var a = '42';
var b = '42px';

Number(a); // 42
parseInt(a); // 42

Number(b); // NaN
parseInt(b); // 42 -- stops when it hits a non-number
```

If you run your code in pre-ES5 envs, watch out for `parseInt`.

For boolean coercion, use `Boolean(a)` or `!!a`.

The `+` operator is used for addition with numbers and concatentation with strings. How does JS know which one to do when it's mixed? If either of them are a string, the non-strings are coerced into strings.

Although technically implicit coercion, many devs use the `a + ''` to coerce to a string.

There is a difference between `a + ''` and `String(a)`. The implicit version calls `ToPrimitive` which calls `valueOf()` which is then coerced to a string via the internal `ToString` operation. `String(a)` just calls `toString()` directly.

```javascript
var obj = {
  valueOf: function() {
    return 30;
  },
  toString: function() {
    return '29';
  }
};

obj + ''; // "30"
String(obj); // "29"
```

Can coerce a string to a number via `a - 0`, `a / 1`, or `a * 1`.

The logical operators in JS (`&&` and `||`) aren't actually operators because they don't result in a boolean (like they do in other languages). They're rather selector operators.

```javascript
var a = 42;
var b = "abc";
var c = null;

a || b;   // 42
a && b;   // "abc"

c || b;   // "abc"
c && b;   // null
```

JS minifers convert `if (a) { foo(); }` to `a && foo()` -- utilizing short circuiting.

Symbols can be explicitly coerced into strings but not implicitly. They cannot be coerced to numbers. They are always coerced to `true` for boolean coercion.

Many people believe `==` vs `===` is that `==` checks for equality and `===` is equality and type. This is not true. In reality, `==` allows for coercion and `===` does not.

Although coercion has an insignificant performance hit, `===` is slower by the first definition and faster in the second (read: real) one.

Both `==` and `===` check the type. The difference is how they respond when they don't match.

For `!=` and `!==` the full process of `==` and `===` (respectively) are run through, then returns the negation.

Implicit coercion will coerce one or both values.

For comparisons, when one is a string and the other is a number, the string is coerced to a number.

```javascript
var a = '30';
var b = 30;
a == b; // true -- a is coerced to a number
```

There are many gotchas when comparing to a boolean.

```javascript
var a = '30'; // truthy value
var b = true; // truthy value
a == b; // false -- '30' is not being considered a truthy value like you may be thinking. See below for the explanation.
```

The spec says that if it is a boolean, run `ToNumber` on it.

```javascript
var a = '30';
var b = true;

// first step -- coerce boolean to a number
'30' == 1;

// second step -- one is a string and one is a number, coerce the string to a number
30 == 1; // false
```

Never use `== true` or `== false` -- it's opening you up to confusion. `===` is OK because it doesn't allow for coercion.

If the values are `null` and `undefined`, return true for `==`.

```javascript
var a = null;
var b;

a == b; // true -- kind of weird
a == null; // true
b == null; // true
a == undefined; // true
b == undefined; // true

a == false; // false

// Thus, these two are the same
if (a == null) {}
if (a === undefined || a === null) {}
```

If an object is being compared to a primitive, `ToPrimitive` is called on it. See notes above on giving an object a `valueOf` or `toString` function to manipulate its output.

```javascript
var a = 30;
var b = [30];

30 == [30]; // first test
30 == '30'; // converts ToPrimitive which is a string
30 == 30; // true -- coerces the string to a number
```

When boxing a primitive, it is unboxed when coerced. Note: because `null` and `undefined` to do not have object wrapper equivalents and thus can't be boxed, this does not work for them.

```javascript
var a = 'abc';
var b = Object(a); // same as new String(a)

a === b; // false
a == b; // true -- because it goes through the ToPrimitive which unboxes to get the scalar primitive

var c = null;
var d = Object(c); // same thing as Object()
c === d; // false
c == d; // false

var e = undefined;
var f = Object(e); // same thing as Object()
e === f; // false
e == f; // false
```

Some scenarios that can trap you:

* Modified `Number#valueOf`

```javascript
Number.prototype.valueOf = function() {
  return 3;
};

var a = new Number(2);
a == 3; // true -- because boxed values go through valueOf
```

* Modified `Number#valueOf` that changes

```javascript
var counter = 2;
Number.prototype.valueOf = function() {
  return counter++;
}

var a = new Number(29);
var b = new Number(30);

if (a == 2 && a == 3) {
  console.log('Wow, this is really happening.'); // prints
}
```

Major issue: when falsey values are compared to each other.

```javascript
'0' == null; // false
'0' == undefined; // false
'0' == NaN; // false
'0' == ''; // false

// GOTCHA #1
'0' == false; // true -- false is coerced to 0, then '0' == 0 is coerced to 0 == 0, which is true
'0' == 0; // true


false == null; // false
false == undefined; // false
false == NaN; // false

// GOTCHA #2
false == 0; // true -- false -> 0, 0 == 0
// GOTCHA #3
false == ''; // true -- false -> 0, '' -> 0, 0 == 0
// GOTCHA #4
false == []; // true -- false -> 0, [] -> '', '' -> 0, 0 == 0
false == {};

'' == null; // false
'' == undefined; // false
'' == NaN; // false
// GOTCHA #5
'' == 0; // true -- '' -> 0, 0 == 0
// GOTCHA #6
'' == []; // true -- [] -> '', '' == ''

0 == null; // false
0 == undefined; // false
0 == NaN; // false
// GOTCHA #7
0 == []; // true -- [] -> '', '' -> 0, 0 == 0
0 == {};

[] == ![]; // true -- ! takes precedence: flips and converts to boolean so ![] -> false, false -> 0, [] -> '', '' -> 0, 0 == 0

2 == [2]; // true -- [2] -> '2', '2' -> 2, 2 == 2
'' == [null]; // true -- [null] -> '', '' == ''

0 == "\n"; // true -- whitespace is coerced via ToNumber to 0 so "\n" -> 0, 0 == 0
```

There are literally 7 gotchas that can confuse you and make you not want to use coercion. If you understand those then you're good.

If you always, always avoid `== false`, then you really only have three gotchas.

```javascript
'' == 0; // true
'' == []; // true -- but this is pretty rare
0 == []; // true
```

Kyle's rules for avoiding `==` comparison.

1. If either side can have `true` or `false`.
2. If either side can have `[]`, `''`, or `0`.

Table of equalities: https://github.com/dorey/JavaScript-Equality-Table

For comparisons (`<`), both values are called via `ToPrimitive`. If neither are strings, `ToNumber` is run on both. If either of them are strings, it compares the location lexically.

```javascript
var a = [29];
var b = ['30'];

a < b; // true
b < a; // false

var c = [2900];
var d = [291];

c < d; // true -- thus the primitive converts to a string
d < c; // false
```

Trickery with objects and comparisons:

```javascript
var a = { num: 29 };
var b = { num: 30, valueOf: function() { return a; } };

a < b; // false -- objects become "[object Object]"
a > b; // false
a == b; // false -- they are not the same object

a <= b; // true -- what?
a >= b; // true -- what?
```

The spec says that for `a <= b` to just do `a > b` and take the opposite. `a > b` is false, so reverse it.

There is no strict equality equivalent for comparisons. Use explicit coercion if necessary.

My Summary:
* When `+` operator is used, if either are strings it's a concatentation, otherwise it's numerical.
* Implicit coercion calls `ToPrimitive` which calls `valueOf` (or `toString` if no `valueOf`) then coerces that. Explicit bypasses `valueOf` and calls `toString` then coerces that.
* When one is a string and the other a number, `ToNumber` is called on the string
* `!` calls `Boolean(..)` and reverses it. This has highest priority.
* Next booleans get called `ToNumber`
* If object compared to a primitive, `ToPrimitive` is called on the object
* If two objects are being compared, `==` and `===` are identical
* Whitespace is coerced via `ToNumber` to `0`
* Avoid coercion in comparisons if either side can have `true`, `false`, `[]`, `''`, or `0`
* `a <= b` is actually evaluated as `!(a > b)` -- which can cause weird results when comparing objects

My experimentation:

```javascript
[] + []; // "" -- ToPrimitive is called on both, so it becomes "" + ""
"" + []; // ""
[] + ""; // ""
0 + []; // "0"
[] + 0; // "0"

{} + {}; // NaN -- this is because the first { is interpreted as the opening to a code block, so this is really +{} which is the same as Number({}) which is NaN
"" + {}; // "[object Object]" -- empty string concatentates ({}).toString()
{} + ""; // 0 -- same as {} + {}, this is really +"" which is Number("") which is 0

{} + []; // 0 -- same as {} + {}, this is really +[] which is Number([]) which is 0
[] + {}; // "[object Object]" -- [] -> "", {} -> ({}).toString(), "" + "[object Object]"
```

Further explanation on http://www.2ality.com/2012/01/object-plus-object.html

## Chapter 5: Grammar

Statements are like sentences. Expressions are like phrases. Operators are like conjunctions/punctuation.

```javascript
var a = 3 * 6; // This is a declaration statement. 3 * 6 is an expression.
var b = a; // This is a declaration statement. a here is an expression.
b; // This is an expression statement. b here is an expression.
```

All expressions have a completion value (even if it's just `undefined`). The completion statement of `var b = a;` above is not `18` because all `var` statements have a completion value of `undefined` as per the spec.

The completion value of a block (`{}`) is its own last completion value.

```javascript
var a;

if (true) {
  a = 29 + 1;
} // will show 30 in a REPL

// The only way to do something like var b = if (true) { ... } is to use eval which is evil!
```

There's a proposal in ES7 to have a `do { .. }` expression which would allow variable assingment.

The `++` increment operator can be either prefixed `++a` or postfixed `++a`, depending on if you want the operator to incur after or before the value is returned from that statement.

```javascript
var a = 2;
var b = a++ + a; // 5 -- the first a is returned as 2, then incremented, so the second call to a is 3

// Can also use the statement-series comma operator to string together multiple standlone expressions
var c = 2, d;
d = (c++, c); // Parantheses are required here due to operator precedence, discussed later
c; // 3
d; // 3
```

Other side effect operators are `delete obj.propName` and `=` (interestingly).

Because of implicit return values, we are able to string creations along.

```javascript
var a, b, c; // This needs to be set prior when in strict mode.
a = b = c = 30; // now all are set to 30
```

Using the assignment side effect of implicitly returning a value, can shrink code.

```javascript
function vowelsLong(str) {
  var matches;

  if (str) {
    matches = str.match(/[aeiou]/g);

    if (matches) {
      return matches;
    }
  }
}
```

```javascript
function vowelsShort(str) {
  var matches;

  // Extra () is required for operator precedence
  if (str && (matches = str.match(/[aeiou]/g))) {
    return matches;
  }
}
```

For braces (`{}`), if you have an assignment (and thus a RHS lookup), it's an object. If it's just a LHS, it's a block.

```javascript
var a = {}; // object
{} // code block
```

The following is valid code and utilizes JavaScript's labeing functionality which allows use to use a `goto` statement. This causes spaghetti code, however.

```javascript
{
  foo: bar()
}
```

It can be useful when you have nested loops and want to stop or continue the outer. This is actually easier to understand than the non-label version.

```javascript
// foo is the name of our labeled-loop
foo: for (var i = 0; i < 4; i++) {
  for (var j = 0; j < 4; j++) {
    if ((i * j) >= 3) {
      console.log('stopping', i, j);
      break foo;
    }
    console.log(i, j);
  }
}
// 0 0
// 0 1
// 0 2
// 0 3
// 1 0
// 1 1
// 1 2
// stopping 1 3
```

Another example of breaking out of a block (rather than a loop). Uncommon and should be documented heavily.

```javascript
function foo() {
  bar: {
    console.log('hello');
    break bar;
    console.log('never runs');
  }
  console.log('world');
}

foo();
// hello
// world
```

JSON-P makes JSON into valid JS grammar. JSON's valid `{ "a": 42 }` is not valid JS. Passing it into a function makes it an object literal which makes it valid JS grammar.

Another place for `{}` is destructuring in ES6.

```javascript
var myObj = {
  a: 'hey',
  b: 'dude'
};

// This
var { a, b } = myObj;

// is the same as
var a = myObj.a;
var b = myObj.b;

// And this
({ result }) => {
  
}

// Is the same as (ignore the lexical this with fat arrow)
function(data) {
  var result = data.result;
}
```

JS doesn't have an `else if` clause. It's actually parsed as:

```javascript
if (a) {
  // ...
}
else {
  if (b) {
    // ...
  }
}
```

Operator precedence: `,` has the lowest precedence. For example, it's lower than `=`.

```javascript
var a = 30;
var b = (a++, a);
a; // 31
b; // 31 -- (a++, a) is interpreted as do the left a++ then result in the right hand

var c = 30;
var d = c++, c;
c; // 31
d; // 30 -- the comma is less than the equal, so it's more like (var d = c++), c;
```

`&&` has higher precedence than `||`. These are both more precedent than the ternary operator (`? :`).

```javascript
true || false && false; // true -- reading left to right would make it false, but with && first it's true || false which is true.
```

`&&` and `||` are evaluated left-to-right. When these are grouped, they are either grouped to the left or to the right. This is called either less-associativity or right-associativity. Normal conditionals are left-associative.

```javascript
a && b && c;
// Left-associative, so this is really
(a && b) && c;

// If it was right-associative, but left-to-right processing, it'd be like this
a && (b && c); // where c won't run if b is false
```

Ternaries are right-associative.

```javascript
a ? b : c ? d : e;

// Is actually
a ? b : (c ? d : e);

// It's still left-to-right processing, so if a is true, only a and b will be processed.
```

Another right-associative operator is `=`.

```javascript
var a, b, c;
a = b = c = 30;

// Same thing as
a = (b = (c = 30));
```

ASI: Automatic Semicolon Insertion. Lowers errors so that the program still runs even if missing a semicolon. It only takes presence with a newline (aka line break).

In the spec, ASI is supposed to be an error correction rather than a tool to be utilized. Even the creator tells you not to rely on it.

Python has significant whitespace. Does JS really have significant line breaks? Questionable.

ES6 has a "Temporal Dead Zone" where `let` and `const` variables haven't been defined yet. It strangely isn't safe from `typeof` like all other variables.

```javascript
foo: {
  typeof a; // undefined
  typeof b; // ReferenceError
  let b;
}
```

`try..catch..finally` exists. `finally` is always run. It will be run before a `return` or a `throw` in a `try`.

```javascript
function foo() {
  try {
    return 30;
  }
  finally {
    console.log('Hello from finally');

    // If we threw an error here, it would override the return 30 from the try block

    // If we returned something, it would override it -- there is no implicit "return undefined" in finally blocks
  }

  console.log('Does not run');
}

console.log(foo());
// Hello from finally
// 30
```

`switch` statements can clean up code, but require `true` and `false` only -- no truthy values!

## Appendix A: Mixed Environment JavaScript

If JS was always run in the same engine, it'd be a simple following of the spec. But because it's always hosted and run alongside another source (such as browsers), there can be some differences in behavior.

ECMAScript is the real language and spec. JS is the browser implementation of the spec.

The majority of differences are not used.

Many host objects do not conform to JS object behavior. It's important to keep this in mind. Some are not able to be overwritten, some have predefined read-only properties, some don't have `toString`.

Creating a global variable will always create a property on the global object as well.

Creating DOM elements with `id` attributes create global variables of those same names. This can seriously cause issues if you're checking for a variable's existence.

```html
<div id="blahblah"></div>
```

```javascript
if (!blahblah) {
  console.log('Will not run.');
}

console.log(blahblah);
// <div id="blahblah"></div>
```

Do not extend natives. Shims/Polyfills are OK. If something will probably be in the spec later, it's called a prollyfill (hah).

The size of function arguments, num chars in a string, bytes to a function, max depth of call stack, how long a JS program can block a browser, var name length -- all of these are different by browser.
