# I Didn't Know JavaScript: ES6 & Beyond

## Chapter 1: ES? Now & Future

Short chapter. No notes.

## Chapter 2: Syntax

```javascript
// Typical block scoping
{
  let a = 2;
  let b = 3;
  // ...
}

// Another valid syntax
{ let a = 2, b = 3;
  // ...
}

// Experimental, not standardized
let (a = 2, b = 3) {
  // ...
}
```

`let` and `const` are not initialized until they appear in the block. Trying to access them will give a `ReferenceError`. This is technically called the Temporal Dead Zone error -- accessing a var that's been declared but not initialized.

`typeof` usually survives undefined vars, but not TDZ vars.

```javascript
{
  typeof a; // undefined
  typeof b; // ReferenceError

  let b;
}
```

Always put `let` and `const` at the top of blocks for this reason.

`let` in a `for` loop re-declares the variable each time, making closures act as you'd expect.

```javascript
var funcs = [];

for (let i = 0; i < 5; i++) {
  funcs.push(function() {
    console.log(i);
  });
}

funcs[3](); // 3 -- not 5 like it would be if we had var i = 0 in the initializer statement
```

`const` sets the variable and sets it to be read-only.

There are two arguments for using `const`. One is to always use it and then change it to `let` if needed. The other is to use `const` only to signal to other devs and your future self that you do not want this value to be changed.

ES6 now has block-scoped functions.

```javascript
{
  foo();

  // Still hoisted. Can only be accessed in this block.
  function foo() {
    // ...
  }
}

foo(); // ReferenceError
```

`...` is either the Spread or the Rest operator, depending on its usage.

```javascript
function foo(x, y, z) {
  console.log(x + y + z);
}

// Used in front of an iterable, it spreads out the values into individual values
foo(...[1, 3, 5]); // 9

// Similar to
foo.apply(null, [1, 3, 5]);


// Spreading out arrays into other arrays
var a = [2, 3, 4];
var b = [1, ...a, 5];
console.log(b); // [1,2,3,4,5]
```

The opposite way is to gather it into an array. It gathers the *rest* of the arguments into an array.

```javascript
// ...z in this case is called a "rest parameter" because you're gathering all the rest of the params into a variable
function foo(x, y, ...z) {
  console.log(x, y, z);
}

foo(1, 2, 3, 4, 5); // 1 2 [3,4,5]
```

Default params were dangerous before ES6's Default Parameter Values.

```javascript
function foo(x, y) {
  x = x || 10;
  y = y || 20;
  return x + y;
}

foo(0, 42); // 52 -- because 0 is falsey

// Better result
function bar(x, y) {
  x = (x !== undefined) ? x : 10;
  y = (y !== undefined) ? y : 20;
  return x + y;
}

bar(0, 42); // 42

// The challenge is passing in undefined if you really want to pass that in. Be careful to test your code if undefined is possible.

// ES6's syntax -- executes much more similarly to bar
function baz(x = 10, y = 20) {
  return x + y;
}

baz(0, 42); // 42
```

The default param value can be any valid expression.

```javascript
function bar(x) {
  return x + 5;
}

function foo(a = 10, b = bar(25)) {
  return a + b;
}

foo(); // 40
foo(7); // 37
foo(3, 8); // 11 -- Note: bar is not called here. It's only called when it's needed
```

The formal params in a function declaration are in their own scope. They are not in the scope of the function body's scope.

```javascript
var w = 3, z = 2;

function foo(x = w + 1, y = x + 1, z = z + 1) {
  console.log(x, y, z);
}

foo(); // ReferenceError: z is not defined
// 1st param: w is found in the outer scope and is used to set x
// 2nd param: the x is found in the formal param scope
// 3rd param: fails because it's not-yet-initialized-at-that-moment

When destructuring objects, you can write it longways to give it a new variable name.

```javascript
var obj = {
  a: 1,
  b: 2,
  c: 3
};

var { a: bar, b: baz, c: bap } = obj;

bar; // 1
```

Can even use computed properties.

```javascript
var obj = {
  a: 1,
  b: 2,
  c: 3
};

var thisVar = 'a';
var myMadeUpName = 'alpha';

// Need to wrap it in () or else it will be taken as a block statement rather than an object
({ [thisVar]: obj[myMadeUpName] } = obj);

obj.alpha; // 1
```

You can even have repeated assignments.

```javascript
var obj = { a: 1 };

var { a: X, a: Y } = obj;

X; // 1
Y; // 1
```

```javascript
var obj = {
  a: {
    b: 'blah'
  }
};

var { a: { b: c } } = obj;

c; // "blah"
```

The assignment expression has a completion value of the righthand object/array.

```javascript
var obj = {
  a: 1,
  b: 2
};
var p;

p = { a, b } = obj;
console.log(a, b); // 1  2
p === obj; // true
```

Don't need to define a var if grabbing from an array.

```javascript
var arr = [1, 3, 5];
var [,b] = arr;
b; // 3
```

Similar to destructuring an array:

```javascript
var a = [2, 3, 4];
var b = [1, ...a, 5];
b; // [1,2,3,4,5]

var [,,...c] = b;
c; // [3,4,5]
```

Can use a default value assignment in destructuring.

```javascript
var obj = {
  a: 1,
  b: 2
};

var { a = 3, b, c: d = 8 } = obj;
a; // 1
b; // 2
d; // 8
```

There's a difference between destructuring default values vs function param default value.

```javascript
function myFn({ x = 10 } = {}, { y } = { y: 20 }) {
  console.log(x, y);
}

myFn(); // 10 20
myFn({}, {}); // 10 undefined
myFn({ x: 30 }, undefined); // 30 20
```

Concide method form:

```javascript
var x = 2;
var y = 3;

var obj = {
  x,
  y,
  foo() {

  },
  *barGenerator() {

  }
};
```

Concide method naming (like `foo` above) is good if you don't plan on doing any recursion or event binding because of how `this` works.

ES6 standardizes `__proto__` but that should rarely be used.

Objects also have `super` access to their prototype objects.

```javascript
var obj1 = {
  a: 1,
  foo() {
    console.log(this.a);
    console.log('obj1: foo');
  }
};

var obj2 = {
  a: 2,
  foo() {
    // cannot just call super()
    // Note: super is locked to obj2 here
    super.foo();
    console.log(this.a);
    console.log('obj2: foo');
  }
};

Object.setPrototypeOf(obj2, obj1);

obj2.foo();
// 2
// obj1: foo
// 2
// obj2: foo
```

String templating is a misleading term. It's more string interpolation.

```javascript
var name = 'Gerry';
var greeting = `Hello ${name}`;

greeting; // "Hello Gerry"

// Line breaks are preserved
var multiLineStrInterpolation = `Why hello
my good sir.
How are you?`
console.log(multiLineStrInterpolation);
// Why hello
// my good sir.
// How are you?
```

Tagged Template Literals -- better term: Tagged String Literals. Pretty cool feature for ES6.

```javascript
function foo(strings, ...values) {
  console.log(strings);
  console.log(values);
}

var name = 'Gerry';

foo`Call me ${name}!`;
// ["Call me", "!"]
// ["Gerry"]
```

Use fat-arrow (`=>`) functions when:
* It's a short, single-statement inline expression
* The function doesn't already make a `this` reference inside of it
* Not doing recursion
* You're doing the hack of `var self = this` or `.bind(this)`
* You're doing `var args = Array.prototype.slice.call(arguments)`

`=>` is about lexical bind of `this`, `arguments`, and `super`. Not about saving keystrokes.

`for..of` loops require an *iterable*. An iterable is simply an object that is able to produce an iterator.

```javascript
var arr = ['a', 'b', 'c', 'd'];

for (var val of arr) {
  console.log(val);
}
// a  b  c  d

// ES6 non-for..of equivalent via manual iteration
for (var val, ret, it = arr[Symbol.iterator](); (ret = it.next()) && !ret.done;) {
  val = ret.value;
  console.log(val);
}
// a  b  c  d
```

Standard iterables:
* arrays
* strings
* generators
* collections / TypedArrays

`for (XYZ of ABC)` the `XYZ` can be an assignment expression.

```javascript
var obj = {};

for (obj.a of [1, 2, 3]) {
  console.log(obj.a);
}
// 1  2  3
obj; // { a: 3 }

for ({ x: obj.b } of [{ x: 1 }, { x: 2 }, { x: 3 }]) {
  console.log(obj.b);
}
// 1  2  3
obj; // { a: 3, b: 3 }
```

JS strings are typically interpreted as 16-bit characters which correspond to the characters in the Basic Multilingual Plane (BMP). Strings could have other UTF-16 characters in them but they could only be matched with RegExps when treated as two characters in a row -- until ES6 added support.

The `u` flag tells the RegExp to process a string with the interpretation of Unicode (UTF-16) chars.

```javascript
/𝄞/.test( "𝄞-clef" ); // true

// The ^ says to match only a single character
/^.-clef/ .test( "𝄞-clef" );    // false

// This is true because ES6's u flag makes Unicode match the single character
/^.-clef/u.test( "𝄞-clef" );    // true
```

The `y` flag puts it into sticky mode. Essentially you're testing it from the RegExp's `.lastIndex` index given. Note: if you cannot predict the structure of the string, `y` is probably not a suitable solution.

```javascript
var reg1 = /foo/;
var str = '++foo++';

reg1.test(str); // true

var reg2 = /foo/y;

reg2.test(str); // false

reg2.lastIndex = 2;
reg2.test(str); // true
reg2.lastIndex; // 5 -- automatically set. This is where the power comes in if you know the structure of a string.

reg2.lastIndex = 1;
reg2.test(str); // false
reg2.lastIndex; // 0
```

Getting the flags off of a RegExp required parsing in ES5. Now it's easily available.

```javascript
var regex = /foo/ig;

regex.toString(); // "/foo/gi"

var flags = regex.toString().match(/\/([gim]*)$/)[1];

flags; // "gi"

// ES6 .flags
regex.flags; // "gi"

// Note: the spec requires the order to be "gimuy" so that's why they switched
```

Unicode characters range from `0x0000` to `0xFFFF` and contain all the standard printed characters in various languages. This range of characters is called the Basic Multilingual Plane (BMP). It includes the snowman (U+2603).

Extended Unicode characters extend up to `0x10FFFF` and are called astral symbols. These include include 𝄞 (U+1D11E) and 💩 (U+1F4A9).

Prior to ES6:

```javascript
// Issue: Unicode escaping only supports 4 characters
var snowman = "\u2603";
console.log(snowman); // ☃
```

To interpret an astral character prior to ES6, you had to use a *surrogate pair*, i.e. two specially calculated Unicode-escaped characters side by side which JS interprets as a single astral character.

```javascript
var clef = "\uD834\uDD1E";
console.log(clef); // 𝄞
```

ES6 has Unicode escaping in strings and RegExps called Unicode code point escaping.

```javascript
var clef = "\u{1D11E}";
console.log(clef); // 𝄞
```

By default, JS string operations are not sensitive to astral symbols.

```javascript
var snowman = '☃';
var clef = '𝄞';

snowman.length; // 1
clef.length; // 2

// Costly workarounds
[...clef].length; // 1
Array.from(clef).length; // 1
```

Sometimes you need to do a Unicode normalization to combine adjacent marks.

```javascript
var str1 = '\xE9'; // "é"
var str2 = 'e\u0301'; // "é" -- weird that it pasted like that, it looks just like \xE9 in console.log

str1 === str2; // false
[...str1].length; // 1
[...str2].length; // 2

str1.normalize().length; // 1
str2.normalize().length; // 1
```

Testing the length for these special characters is a rabbit hole.

`String#charAt` doesn't work for these astral symbols unless you do `[...str.normalize()][2]` (for getting the char at index `2`). This is expensive.

Best bet is to combine `String.fromCodePoint` and `String#normalize String#codePointAt`.

```javascript
var s1 = "abc\u0301d",
  s2 = "ab\u0107d",
  s3 = "ab\u{1d49e}d";

String.fromCodePoint( s1.normalize().codePointAt( 2 ) );
// "ć"

String.fromCodePoint( s2.normalize().codePointAt( 2 ) );
// "ć"

String.fromCodePoint( s3.normalize().codePointAt( 2 ) );
// "𝒞"
```

Can create Unicode-escaped chars as var names.

```javascript
var \u03A9 = 42;
// Same as: var Ω = 42;
Ω; // 42

// As of ES6, can use new escape syntax
var \u{2B400} = 42;

// same as: var 𫐀 = 42;
```

Symbols do not have a literal form.

```javascript
var sym = Symbol('some optional description');

typeof sym; // "symbol"
```

Note:
* Do not use `new` as it's not a constructor and you're not producing an object
* The param is optional -- but use it as a friendly descriptor

The internal value of a symbol itself (referred to as its `name`) is hidden from code and cannot be obtained. Think of this symbol value as an auto generated, unique (within your app) string value.

Main point: creating a string-like value that can't collide with any other value. Good example is with a constant that is across the app.

```javascript
const EVT_LOGIN = Symbol('event.login');

// Replace a generic string literal of "event.login" with EVT_LOGIN
evthub.listen(EVT_LOGIN, function(data) {
  // ...
});
```

You can use Symbols as a special property on an object that you want to treat as hidden or meta. Note: although this is your intent, it's not actually hidden or untouchable.

Example of a Singleton pattern.

```javascript
const INSTANCE = Symbol('instance');

function Racecar() {
  // Storing the symbol on the Racecar function object
  if (Racecar[INSTANCE]) {
    return Racecar[INSTANCE];
  }

  function startEngine() {
    // ...
  }

  return Racecar[INSTANCE] = {
    startEngine: startEngine
  };
}

var myCar = Racecar();
var yourCar = Racecar();

myCar === yourCar; // true
```

Often need to use Symbols in the global scope. Use the *global symbol registry*. This checks to see if there is already a symbol stored with that descriptor. If there is, it returns it. If it doesn't, it creates one then returns it.

```javascript
var EVT_LOGIN = Symbol.for('event.login');
```

Symbols are used to eliminate magic strings, however their matching is dependent on the string descriptor.

Real benefit of Symbols over magic strings is with metaprogramming.

Can get a symbol's descript by using `Symbol.keyFor(sym)`.

See an object's symbols via `Object.getOwnPropertySymbols(obj)`.

There are built-in Symbols that are accessed directly on the Symbol function. The spec writes these with a prefix of `@@`. Here's accessing `@@iterator`.

```javascript
var arr = ['a', 'b', 'c'];
arr[Symbol.iterator]; // native function
```

## Chapter 3: Organization

Iterators must have `.next()`. Some iterators extend with `.return()` which stops the iterator and returns `IteratorResult` and `.throw()` which signals an error and returns `IteratorResult`.

`IteratorResult` returns an object with `.value` (optionally if `undefined`) and `.done` boolean.

`Iterable` must be able to produce iterators.

```
Iterable
  @@iterator() {method}: produces an Iterator
```

An array is an iterable. Can create an iterator to produce its values.

```javascript
var arr = [1, 2, 3];
var iterator = arr[Symbol.iterator]();

iterator.next(); // { value: 1, done: false }
iterator.next(); // { value: 2, done: false }
iterator.next(); // { value: 3, done: false }

// Need to call it one more time for it be finished
iterator.next(); // { value: undefined, done: true }
```

Strings are iterables. Technically, string primitives are not but they are boxed to their String object wrapper which *is* an iterable.

ES6 adds several new data structures called Collections. Collections are iterables themselves and have API methods to generator iterators.

```javascript
var m = new Map();
m.set('blah', 30);
m.set({ a: 1 }, 'hey');

var iterator1 = m[Symbol.iterator]();
var iterator2 = m.entries();

iterator1.next(); // { value: ["blah", 30], done: false }
iterator2.next(); // { value: ["blah", 30], done: false }
```

`.return()` will automatically be called in an early termination and can be manually called as well. In some iterators, sending a value in to `.return(..)` will make the `IteratorResult` value equal the passed in value.

If an iterator is also an iterable, it can be used directly with `for..of`. You make an iterator an iterable by giving it a `Symbol.iterator` method that returns the iterator itself.

```javascript
var it = {
  [Symbol.iterator]() {
    return this;
  },
  next() {
    // ..
  }
};

// Can now use for..of
for (var v of it) {
  console.log(v);
}
```

Custom iterator using Fibonnaci sequence:

```javascript
var Fib = {
  [Symbol.iterator]() {
    var n1 = 1;
    var n2 = 1;

    return {
      // Makes the iterator an iterable
      [Symbol.iterator]() {
        return this;
      },

      next() {
        var current = n2;
        n2 = n1;
        n1 = n1 + current;
        return {
          value: current,
          done: false
        }
      },

      // Use this to clean up any connections
      return(v) {
        console.log('Fib sequence abandoned');

        return {
          value: v,
          done: true
        }
      }  
    };
  }
};

for (var v of Fib) {
  console.log(v);

  // break will be called an abnormal return and will thus call .return()
  if (v > 50) break;
}
// 1 1 2 3 5 8 13 21 34 55
// Fib sequence abandoned.
```

Array destructuring consumes an iterator.

```javascript
var arr = [1, 2, 3, 4, 5];

var it = arr[Symbol.iterator]();

var [x, y] = it; // Takes the two elements from `it`
var [z, ...w] = it; // Take the third then take the rest

it.next(); // { value: undefined, done: true }

x; // 1
z; // 3
w; // [4,5]
```

Executing a `*foo` generator like `foo(5, 10)` doesn't run the function but it rather returns an iterator to control the code execution.

```javascript
function *foo() {
  var x;

  console.log('Starting...');
  x = yield 'blah';
  console.log(x);
}

var it = foo();

it.next().value; // "blah"
it.next(5); // 5
// implicit return: { value: undefined, done: true }
```

Only the `...` spread operator and the `,` comma operator have lower precedence. Often need to wrap the yield with `()` to make it valid and execute first.

`yield` is right-associative. `yield yield 4` is the same as `yield (yield 4)`.

`yield *bar` will invoke `bar`'s iterator and delegate its own host generator's control to `bar`'s iterator until `bar`'s iterator is exhausted, then it resumes with the original.

Generators, when execute, create both an iterator and iterable. Thus can be used in `for..of` loops.

The first `.next()` doesn't need a value because all it's doing is starting the generator.

A generator's iterator does have `.return` and `.throw` methods. Both abort a paused generator immediately.

`return` is always run when completed. The main way to accomplish your own cleanup pattern:

```javascript
function *foo() {
  try {
    yield 1;
    yield 2;
    yield 3;
  }
  finally {
    console.log('Clean up time');
  }
}

var it = foo();
it.next(); // { value: 1, done: false }

it.return(30);
// from log: "Clean up time"
// implicit return: { value: 30, done: true }
```

Calling `.throw(x)` is essentially like injecting a `throw x` at the pause point.

```javascript
function *foo() {
  try {
    yield 1;
  }
  catch(err) {
    console.log("Caught error in foo's catch" + err);
  }
}

function *bar() {
  yield 1;
}

var fooIt1 = foo();
fooIt1.next();
fooIt1.throw('foo1');
// printed: "Caught error in foo's catch: foo1"
// implict return: { value: undefined, done: true }

var fooIt2 = foo();
fooIt2.next();

try {
  fooIt2.throw('foo2');
}
catch(err) {
  console.log('Caught in main catch: ' + err);
}
// printed: "Caught error in foo's catch: foo2"
// Notice how it doesn't propagate out

var barIt = bar();
barIt.next();

try {
  barIt.throw('barItErr');
}
catch(err) {
  console.log('Caught in main catch: ' + err);
}
// "Caught in main catch: barItError"
// Notice how it propagates out if not caught inside
```

Bonus: adding a `try..finally` -- even with an error, the `finally` would be called before the error propagated out of the `barIt.throw` example above.

Manually recreating generators:

```javascript
// Target:
function *foo() {
  var x = yield 42;
  console.log(x);
}

// Our version
function foo() {
  function nextState(v) {
    switch (state) {
      // 0: initial state
      case 0:
        state++;

        // Handle yield expression
        return 42;
      case 1:
        state++;

        // yield fulfilled
        x = v;
        console.log(x);

        // implicit return that generators have
        return undefined;
    }
  }

  var state = 0;
  var x;

  return {
    next: function(v) {
      var ret = nextState(v);

      return {
        value: ret,
        done: (state == 2)
      };
    }
    // Skipping return and throw for simplicity sake
  }
}

var it = foo();
it.next(); // { value: 42, done: false }
it.next(10);
// print: 10
// return: { value: undefined, done: true }
```

Two main uses of generators: to produce a series of values and to run through a queue of tasks serially.

ES6 modules are done via one per file. They have a static API. They are all singletons. If you want to be able to produce multiple instances, you will need to provide a factory. Importing a module is the same thing as statically requesting it to load -- thus a blocking load over the network for browsers and a blocking load from the filesystem for Node.

AMD, UMD, CommonJS, and ES6's native module system are different means of module loading. For now, CommonJS is ahead of ES6 in terms of API and fine-tune controlling for Node.js and UMD or AMD is ahead for the browser.

ES6 uses `import` and `export`. Both of which must appear at the top-level of the module's scope, outside of all blocks and functions.

There is no global scope in modules. The only top-level scope is the module itself.

```javascript
export function foo() {
  // ...
}

export var awesome = 30;

var bar = [1, 2, 3];
export { bar };

// Same thing as:
function foo() {
  // ...
}

var awesome = 30;
var bar = [1, 2, 3];

// These are called named exports
export { foo, awesome, bar };

// These can be renamed aka aliased
export { foo as blah, awesome, bar };
```

When referencing a module's value, you are referencing a binding (similar to a pointer). In the example below, the binding is a pointer to the `awesome` variable itself, not a copy of its value (like normal JS). Calling this module will point to the `awesome` variable and thus will return `100`.

```javascript
var awesome = 30;
export { awesome };

awesome = 100;
```

Each module has one `default` exported value. It is encouraged to only export one value thus the default is clear.

There is a subtle syntax difference when exporting that has different effects.

```javascript
function foo() {
  // ...
}

// (1) If foo is changed, the exported value will STILL reference the original foo. This way points to the value, not the identifier.
export default foo;

// (2) If foo is changed, the exported value will update. This syntax implies that you want a pointer to that identifier.
export { foo as default }; 
```

Importing:

```javascript
// If a module is using named exports
import { foo } from './foo';
import { foo as theFooFunc } from './foo';

// If the module has just a default export. These two are the same. The first one is implicitly getting the default.
import foo from './foo';
import { default as foo } from './foo';
```

If you export multiple references to a module's API, you can import all of them into a wildcard.

```javascript
export function foo() {};
export var bar = 30;
export function baz() {};

// In another file
import * as myModule from './my-module';
myModule.bar; // 30
```

All imported variables are immutable and/or read-only.

```javascript
// continuing above
myModule.bar = 100; // (runtime) TypeError
```

It is possible for modules to change their API from the inside, however ES6 modules are intended to be static. Any deviations from this should be well documented. Note: Can export a plain object and this object can be mutated.

The module system solves circular dependencies.

The host environment provides the Module Loader mechanism. The specs for this are not set by ES6 but rather a separate, parallel committee.

Can interact with the module loader directly. This is useful if a non-module needs to load a module.

```javascript
// This script is loaded in the browser via `<script>`
// import is illegal here

// Is evaluated like the wildcard described above
// Returns a promise
Reflect.Loader.import('foo')
  .then(function(foo) {
    foo.bar();
  });
```

Can use `Promise.all` with `Reflect.Loader.import(..)` to load many.

Can use `Reflect.Loader.import(..)` in a real module if you want to dynamically or conditionally load a module. This has lower performance though.

`class` methods are non-enumerable. They must be made with `new` (as opposed to `Foo.call(obj)`). They are not hoisted.

Can also do `var x = class Y { .. };`. This is beneficial if you want to pass a class constructor as a function argument or assigning it to an object.

`super` is not limited to `class` declarations. It can be used in object literals.

`super` is statically bound to the `extends XYZ`.

Providing a `constructor` method is not required. Default subclass constructor: 

```javascript
// Default subclass constructor
constructor(...args) {
  super(...args);
}
```

You cannot access `this` in a subclass before calling `super` in the constructor.

Huge benefit of `class`: extending natives. This was troublesome in pre-ES6.

```javascript
class MyCoolArray extends Array {
  first() { return this[0]; }
  last() { return this[this.length - 1]; }
}

var arr = new MyCoolArray(1, 2, 3);
arr.length; // 3
arr.first(); // 1
arr.last(); // 3


class Oops extends Error {
  constructor(reason) {
    super(reason);
    this.oops = reason;
  }
}

var ouch = new Oops('I messed up');
ouch; // Oops: I messed up
ouch.oops; // "I messed up"
throw ouch; // Will behave like any other error object, including capturing the stack
```

New ES6 concept: meta property via `new.target`. When used with `class`, it tells you what function had `new` called on it. Benefit: if you want to throw an error if `new` was not used.

```javascript
class Foo {
  constructor() {
    console.log('new.target: ' + new.target.name);
  }
}

class Bar extends Foo {
  constructor() {
    super();
  }
}

function Baz() {
  // Note: not using .name here
  console.log(new.target);
}

var a = new Foo();
// new.target.name: Foo

var b = new Bar();
// new.target.name: Boo

// Note: not using new here
var c = Baz();
// undefined
```

Can add properties to the function itself with `static`.

```javascript
class Foo {
  static cool() { console.log('Essentially a class method'); }

  wow() { console.log('A normal instance method') }
}

Foo.cool(); // Essentially a class method
Foo.wow; // undefined

var f = new Foo();
f.cool; // undefined
f.wow(); // A normal instance method
```

`@@species` let's you define what kind of class a child should return.

```javascript
class MyCoolArray extends Array {
  static get [Symbol.species]() { return Array; }
}

var arr = new MyCoolArray(1, 2, 3);
arr instanceof MyCoolArray; // true
arr instanceof Array; // true

var doubledArr = arr.map(function(v) { return v * 2; });

doubledArr instanceof MyCoolArray; // false
doubledArr instanceof Array; // true
```

## Chapter 4: Async Flow Control

Converting a callback to a promise:

```javascript
// Callback
function ajax(url, cb) {
  // Make a request
  // Eventually call cb(..)
}

ajax('url.com/1', function handler(err, contents) {
  if (err) {
    // handle ajax error
  }
  else {
    // handle contents
  }
});


// Promise
function ajax(url) {
  return new Promise(function pr(resolve, reject) {
    // Make a request
    // Call either resolve(..) or reject(..)
  });
}

ajax('url.com/1')
  .then(function fulfilled(contents) {
    // handle contents
  }, function rejected(reason) {
    // handle ajax error
  });
```

Similar shortcuts:

```javascript
// Same behaviors
var p1 = Promise.resolve(30);
var p2 = new Promise(function(resolve, reject) {
  resolve(30);
});

// Same behaviors
// Note: ajax returns a Promise
var thePromise = ajax('url.com');

var p3 = Promise.resolve(thePromise);
var p4 = new Promise(function(resolve, reject) {
  resolve(thePromise);
});
```

It is possible to express a series of promises in a chain to represent async flow control. Consider this long promise chain:

```javascript
step1()
  .then(step2, step2Failed)
  .then(function step3(msg) {
    return Promise.all([
      step3a(msg),
      step3b(msg),
      step3c(msg)
    ]);
  })
  .then(step4);
```

Much better solution: generators that yield promises.

```javascript
function *main() {
  try {
    var ret = yield step1();
  }
  catch (err) {
    ret = yield step1Failed(err);
  }

  ret = yield step2(ret);

  ret = yield Promise.all([
    step3a(ret),
    step3b(ret),
    step3c(ret)
  ]);

  yield step4(ret);
}
```

To get this to run, we need a "runner" that will start the generator, receive `yield`ed promises and resume the generator with the fulfillment value or throw an error into the generator with the rejection reason.

Anywhere that you have more than two async steps of flow control logic you should use a promise-yielding generator.

## Chapter 5: Collections

Maps: similar to objects, but you can use any value for the keys -- event objects or other maps.

Sets: similar to arrays, but the values are unique

TypedArray: provides structured access to binary data

```javascript
var buffer = new ArrayBuffer(32);

buffer.byteLength; // 32
// buffer is now a binary buffer that is 32-bytes long and initialized to all 0's

var arr = new Uint16Array(buffer);
arr.length; // 16
// arr is a typed array of 16-bit unsigned integers mapped over the 256-bit `buffer` buffer -- thus 16 elements
arr; // [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
```

```javascript
var m = new Map();

var x = { id: 1 };
var y = { id: 2 };

// Use an object as a key
// Downside: can't use [] access to set or get values
m.set(x, 'foo');
m.set(y, 'bar');

m.get(x); // "foo"
m.get(y); // "bar"

m.size; // 2
m.clear();
m.size; // 0

// Another way to create a map -- 2d arrays where the first is the key and the second is the value
var m2 = new Map([
  [x, 'foo'],
  [y, 'bar']
]);

m2.entries(); // MapIterator {[Object, "foo"], [Object, "bar"]}
m2.keys(); // MapIterator {Object {id: 1}, Object {id: 2}}
m2.values(); // MapIterator {"foo", "bar"}

m2.has(x); // true
m2.delete(y);
m2.has(y); // false
```

Use normal objects as maps unless you need a key to be an object.

WeakMaps: maps that *only* have objects as keys. The objects are held weakly -- if the object is GC'd, the entry in the WeakMap is GC'd. Create using `var wm = new WeakMap();`.

```javascript
var s = new Set();
var x = { id: 1 };
var y = { id: 2 };

s.add(x);
s.add(y);
s.size; // 2

s.add(x);
s.size; // 2

s.delete(y);
s.size; // 1

s.clear();
s.size; // 0

var arrWithDups = [1, 2, 1, 2, 3]
var s2 = new Set(arrWithDups);
s2; // Set {1, 2, 3}

var arrUniques = [...s2];
arrUniques; // [1, 2, 3]
```

## Chapter 6: API Additions

Weird behavior in JS when creating an array via the Array constructor. If you pass in one number, it creates an array of that length. If you pass in multiple, it creates a real array with those as entries.

If you want to create an array of just one element, use the new `Array.of(3);`.

`Array.from(arrOrArrLikeObj)` will convert the argument into an array and duplicate it. The new best way to make an array with many empty slots is now `Array.from({ length: 4 });`.

```javascript
Array(3).fill('blah');
// ["blah", "blah", "blah", "blah"]

var arr = [1, 2, 3, 4];
arr.find(function matcher(v) {
  return v % 2 === 0;
}); // 2
// For a boolean version, use .some

// indexOf just applies strict equality
// findIndex can have a function applied
arr.findIndex(function matcher(v) {
  return v % 2 === 0 && v / 2 >= 2;
}); // 3 (index 3 has a value of 4)
```

Although `Object.is(..)` is now in ES6 to test for equality, it's not a replacement for `===`. Only use when strictly identifying a `NaN` or a `-0`. Consider:

```javascript
var x = NaN;
var y = 0;
var z = -0;

x === x; // false
y === z; // true

Object.is(x, x); // true
Object.is(y, z); // false

// Better test for NaN
Number.isNaN(x);

// Better test for -0
Object.is(z, -0);
```

Lots of `Math` functions have been added for trig.

Helpful `Number` properties
* `Number.EPSILON` -- the numbe rused as tolerance for imprecision in floating-point arithmetic
* `Number.MAX_SAFE_INTEGER` -- The highest integer that can be safely represented in JS -- `2^53 - 1`
* `Number.MIN_SAFE_INTEGER` -- Similar, but the smallest -- `-(2^53 - 1)`
* `Number.isSafeInteger`
* `Number.isNaN`
* `Number.isFinite` -- similar to the global `isFinite`, but doesn't coerce -- better to use this and explicitly coerce via `Number.isFinite(+x)`
* `Number.isInteger` -- checks if any non-zero value in the decimal place

`String#repeat` is similar to Ruby's `'foo' * 3`.

Helpful string methods
* `String#repeat` -- multiply a string by a number to repeat it
* `String#startsWith`
* `String#endsWith`
* `String#includes` -- optional second param of what index to start at

Note: The above methods will not accept a regular expression by default. Regular Expression Symbols are discussed later.

## Chapter 7: Meta Programming

Meta programming: programming which targets the behavior of the program itself. Focuses on: code inspecting itself, code modifying itself, or code modifying default language behavior so other code is affected.

Goal of meta programming: leverage the language's own intrinsic capabilities to make the rest of the code more descriptive, expressive, and/or flexible.

Introspection: a form of meta programming; inspecting the program using the program. Example: `obj1.isPrototype(obj2)`

Macros: code that modifies itself at compile time; currently not in JS.

Can better inspect a function's name now.

```javascript
var foo = function() {};

function bar() {}

var baz = bar;

foo.name; // "foo"
bar.name; // "bar"
baz.name; // "bar" -- references the real function name
```

Can use `Object.defineProperty(..)` to modify the `name` property.

`new.target` inside a constructor is an introspection operation.

Well Known Symbols (WKS) are defined primarily to expose special meta properties to give you more control.

WKS:
* `Symbol.iterator`
* `Symbol.toStringTag` -- overriding the `.toString()` of an object so it shows `"[object MyMadeUpObj]"`
* `Symbol.hasInstance` -- overriding the `a instanceof X`expression
* `Symbol.species` -- when dealing with 
* `Symbol.toPrimitive` -- override the `ToPrimitive` that objects do in coercion
* `Symbol.match` -- applied to RegExps, used to match all or part of a string value with a given RegExp. Used by `String.prototype.match` and `isRegExp`.
* `Symbol.replace`
* `Symbol.search`
* `Symbol.split`
* `Symbol.isConcatSpreadable` -- should an object be spread out when using `Array#concat`
* `Symbol.unscopables` -- configuring scope in a `with` statement

```javascript
function Foo(givenName) {
  this.givenName = givenName;
}

Foo.prototype[Symbol.toStringTag] = 'FooObj';

var a = new Foo();

a.toString(); // "[object FooObj]"

// Have to use this syntax to define it as by default it's writable: false
Object.defineProperty(
  Foo,
  Symbol.hasInstance,
  {
    value: function(instance) {
      return instance.givenName === 'myObj';
    }
  }
);

var b = new Foo('myObj'); // true
var c = new Foo('otherObj');

b instanceof Foo; // true
c instanceof Foo; // true

var obj = {
  [Symbol.toStringTag]: 'madeUpObj'
};

obj.toString(); // "[object madeUpObj]"
```

```javascript
var arr = [1, 2, 3, 4, 5];

// Default ToPrimitive is to convert to a string first
arr + 10; // "1,2,3,4,510"

arr[Symbol.toPrimitive] = function(hint) {
  if (hint == 'default' || hint == 'number') {
    // sum the numbers
    return this.reduce(function(acc, current) {
      return acc + current;
    }, 0);
  }
};

// Note: `+` operator has no hint, so `"default"` is passed in
arr + 10; // 25
```

Tread lightly when overriding the RegExp algorithms (via symbols).

```javascript
var a = [1, 2, 3];
var b = [4, 5, 6];
var c = [].concat(a, b);

b[Symbol.isConcatSpreadable] = false;

var d = [].concat(a, b);

c; // [1, 2, 3, 4, 5, 6]
d; // [1, 2, 3, [4, 5, 6]]
```

Proxy: special kind of object that "wraps" / "sits in front of" another normal object.

You register special handlers (called "traps") on the proxy object which are called when various operations are performed against the proxy. These handle additional logic before forwarding the operations to the original target (aka wrapped object).

```javascript
var obj = { a: 1 };
var handlers = {
  get(target, key, context) {
    // target === obj
    // context === proxyObj
    console.log('Accessing:', key);

    // .get is provided because this function is called get
    return Reflect.get(
      target, key, context
    );
  }
};
var proxyObj = new Proxy(obj, handlers);

obj.a; // 1
proxyObj.a;
// log: "Accessing: a"
// returned: 1
```

There are a lot of available proxies. They cover pretty much anything you want to do meta-level on an object (including arrays and functions). See the `Reflect` API to learn more.

It's possible to create a revocable proxy that can be expired. This is useful if you want to invalidate a user's info after a set amount of time. Any references to a revoked proxy will result in a `TypeError`.

Proxies are used when you want to pass an object around but can't fully trust that they won't modify the object.

There are proxy-first patterns and proxy-last patterns. Proxy-first is for data manipulation protection. Proxy-last is good for doing catchall behaviors.

With Proxies, it's possible to workaround the `[[Prototype]]` call and create circular references (`obj1` delegating to `obj2` and viceversa).

Use the `Reflect` API to extend features and APIs to create Domain Specific Languages (DSLs).

`Reflect.ownKeys`, `Object.getOwnPropertyNames`, and `Object.getOwnPropertySymbols` now have a defined order whereas before ES6 it was not to be trusted.

1. Enumerate owned properties that are integer indexes, ascending
2. Enumerate rest of owned string property names in creation order
3. Enumerate owned symbol properties in creation order

Feature test: determine what features are available to use in your code (such as `Number.isNaN` or `() => {}`).

For polyfills, it's easy to do `if (!Number.isNaN) {}`.Because syntax errors will choke, it's harder.

With arrow functions, a feature test is:

```javascript
try {
  new Function( "( () => {} )" );
  ARROW_FUNCS_ENABLED = true;
}
catch (err) {
  ARROW_FUNCS_ENABLED = false;
}
```

Split delivery: using a bootstraping script to do feature tests and then load either ES6 code *or* transpiled code.

Use "https://featuretests.io" which is a feature-tests-as-a-service solution to creating your own. It also caches results so won't be run constantly.

Tail Call Optimization can only run in `strict mode`.

Two pre-TCO ways to enable large recursion are *trampolining* or *recursion unrolling*.

Best way to utilize TCO in ES6+ browsers while still working in lower browsers is to use scoped variables.

```javascript
'use strict';

function foo(x) {
  function _foo() {
    if (x > 1) {
      acc = acc + (x / 2);
      x = x - 1;
      return _foo();
    }
  }

  var acc = 1;

  // if we get a callstack error, this loop will just pick up where we left off
  // utilizes TCO
  while (x > 1) {
    try {
      _foo();
    }
    catch (err) {}
  }

  return acc;
}

// Would normally fail in a ES5 browser
foo(123456); // 3810376848.5
```

## Chapter 8: Beyond ES6

WebAssembly (WASM) will enable other languages to not require compilation into JS to run in a browser. Ex: could run C/C+ code in the browser without first converting it to JS.
