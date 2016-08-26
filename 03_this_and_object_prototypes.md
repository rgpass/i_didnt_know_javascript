# I Didn't Know JavaScript: `this` & Object Prototypes

## Chapter 1: `this` Or That?

Why the flexibility on `this`: it allows us to pass in our own context (as objects) to re-use functions. Instead of re-typing it for several object instances, you can just pass the object in.

`this` is not an author-time binding but a run-time binding.

When a function is executed, an activation record (aka an execution context) is created. This activation record includes the call-stack, how the function was invoked, params passed in, etc. It also includes the `this` reference.

## Chapter 2: `this` All Makes Sense Now!

What matters is the call-site. This can be found by investigating the call-stack right before the function was invoked.

There are four ways in which `this` is set. In order of descending priority:
1. `new` binding
2. Explicit binding
3. Implicit binding
4. Default binding -- global scope

Default binding example:

```javascript
'use strict';
function bar() {
  var a = 2;

  function foo() {
    console.log(this.a);
  }

  foo();
}

var a = 1;

bar(); // 1 -- this doesn't pick up the local scope but rather the global because that's the default binding 

// In strict mode, the default binding is undefined -- not the global object
```

Overall `this` rules are based on the call-site, but strict mode is dependent on the function declaration location. Note: mixing strict and non-strict mode is a recipe for confusion.

```javascript
var a = 2;

function foo() {
  console.log(this.a);
}

(function strictFn() {
  'use strict';
  foo(); // 2
})()
```

Implicit binding example:

```javascript
function foo() {
  console.log(this.a);
}

var obj = {
  a: 2,
  foo: foo
};

obj.foo(); // 2
```

Only the last object matters when doing implicit binding.

```javascript
function foo() {
  console.log(this.a);
}

var obj1 = {
  a: 2,
  foo: foo
};

var obj2 = {
  a: 29,
  obj1: obj1
};

obj2.obj1.foo(); // 2
```

Implicit binding can cause confusion when being passed into callbacks. Best to use explicit binding to ensure the correct `this`.

Explicit binding examples:

```javascript
function foo() {
  console.log(this.a);
}

var obj = {
  a: 2
};

foo.call(obj); // 2
foo.apply(obj); // 2
foo.bind(obj)(); // 2
```

Passing in a primitive into an explicit binding sets `this` to the boxed value.

```javascript
function bar() {
  console.log(this);
}

bar.call(2); // Number {[[PrimitiveValue]]: 2}
```

Hard binding solves the callback issue. It wraps a function in a way such that the context when called is always the same.

```javascript
function foo() {
  console.log(this.a);
}

var obj = {
  a: 2
};

function bar() {
  foo.call(obj);
}

bar(); // 2
setTimeout(bar, 100); // 2
bar.call(window); // 2
```

When using `bind` in ES6, the hard-bound function has a `.name` set to it.

```javascript
function foo() {
  console.log(this.a);
}

var obj = {
  a: 2
};

var bar = foo.bind(obj);
bar.name; // "bound foo"
```

There is no such thing as constructor functions, but rather construction calls of functions.

When using the constructor call (aka `new`), four things are done automatically:
1. A brand new object is created (aka constructed)
2. The newly constructed object is `[[Prototype]]`-linked
3. The newly constructed object is set as the `this` binding for that function call
4. Unless the function returns its own alternate object, the newly created object is returned.

`new` overwrites the value for `this` even if the function was previously `bind`ed. This is useful for currying.

```javascript
function foo(something) {
  this.a = something;
}

var objBind = {};

var bar = foo.bind(objBind);
bar(2);
console.log(objBind.a); // 2

var objNew = new bar(3);
console.log(objBind.a); // 2 -- doesn't change which means that `this` was overwritten
console.log(objNew.a); // 3
```

Currying example:

```javascript
function foo(arg1, arg2) {
  this.val = arg1 + arg2;
}

var bar = foo.bind(null, 'yeah');
var baz = new bar('dude');
console.log(baz.val); // "yeahdude"
```

```javascript
function foo(a, b) {
  console.log('a: ' + a + ', b: ' + b);
}

// ES5 way to apply an array of args to a function
foo.apply(null, [2, 9]); // "a: 2, b: 9"

// ES6 way -- avoids an unnecessary this binding
foo(...[2, 9]); // "a: 2, b: 9"
```

Sometimes passing in `null` as a value for `this` when doing `foo.apply(null, [1, 2])` causes weird side-effects with 3rd party libraries that can alter the global scope. The solution is to create a De-Militarized Zone (DMZ) variable where you don't care what happens to it.

```javascript
// This is better than var DMZ = {}; because this new object is not delegated to Object.prototype
var DMZ = Object.create(null);
```

The issue with hard-binding is that it does not allow us to apply hard-binding or implicit binding later.

```javascript
function foo() {
  console.log(this.a);
}

var obj1 = { a: 1 };
var obj2 = { a: 2 };

var bar = foo.bind(obj1);
bar.call(obj2); // 1
```

It is better to have a soft-binding that has our own default binding while still allowing us to use an implicit binding later.

```javascript
if (!Function.prototype.softBind) {
  Function.prototype.softBind = function(obj) {
    var fn = this,
      curried = [].slice.call( arguments, 1 ),
      bound = function bound() {
        return fn.apply(
          (!this ||
            (typeof window !== "undefined" &&
              this === window) ||
            (typeof global !== "undefined" &&
              this === global)
          ) ? obj : this,
          curried.concat.apply( curried, arguments )
        );
      };
    bound.prototype = Object.create( fn.prototype );
    return bound;
  };
}
```

Now we can use it:

```javascript
function foo() {
  console.log(this.a);
}

var obj1 = { a: 1 };
var obj2 = { a: 2 };

var bar = foo.bind(obj1);
bar.call(obj2); // 1

var baz = foo.softBind(obj1);
baz.call(obj2); // 2
```

In ES6, the fat-arrow lexical `this` takes the highest precedence -- even higher than `new`!

```javascript
function foo() {
  return (a) => {
    console.log(this.a);
  }
}

var obj1 = { a: 1 };
var obj2 = { a: 2 };

var bar = foo.call(obj1);
bar.call(obj2); // 1 -- not 2 because this was already bound to obj1
```

## Chapter 3: Objects

Object sub-types aka built-in objects:

* `String`
* `Number`
* `Boolean`
* `Object`
* `Function`
* `Array`
* `Date`
* `RegExp`
* `Error`

These built-ins look like types (or classes in other languages) however they are just built-in functions that can be used as constructors (aka called with `new`).

When using the constructor, you can use `instanceof`

```javascript
var a = 2;
var b = new Number(2);

a instanceof Number; // false
b instanceof Number; // true
```

String literals are primitive and immutable. To perform operations on it (such as find its length), JavaScript will automatically coerce a `string` primitive into a `String` object when necessary. Because it automatically does this, you almost never need to create it explicitly.

`null` and `undefined` have no object wrapper form. They only have their primitive values.

`Date` values can only be created with their constructor object form.

The property access method (`obj.a`) requires an `Identifier` compatible property name, whereas the key access method (`obj["a"]`) can take any UTF-8/unicode compatible string.

Object properties are always converted into strings.

```javascript
var obj = {};
obj[3] = 'hello';
console.log(obj['3']); // "hello"

var arr = ['a', 'b', 'c'];
arr['2']; // "c"
```

ES6 allows for computed property names when creating object literals. This is most helpful with ES6 symbols (covered later).

```javascript
var prefix = 'foo';

var obj = {
  [prefix + 'bar']: 'hey',
  [prefix + 'baz']: 'dude'
};

obj.foobar; // "hey"
```

For duplicating, ES6 has the new `Object.assign` function which iterates over all enumerable arguments, adding properties solely be using the assignment (`=`) expression -- this does not preserve any property descriptors.

```javascript
var oldObj = { blah: 'yeah' };
var newObj = Object.assign({}, oldObj);
newObj.blah; // "yeah"
```

As of ES5, object properties have property descriptors. There are data descriptors and accessor descriptors (explained more below). The data descriptors are: `writable` means you can change the value of a property, `enumerable` means it will show up in `for..in` loops, `configurable` means that you can modify its property descriptors (cannot be undone when set to `false`) or use the `delete` action to remove a property. If `configurable` is false, `writable` can be later changed to `false` but not back to `true`.

```javascript
var obj = {
  a: 2
};

Object.getOwnPropertyDescriptor(obj, 'a'); // Object {value: 2, writable: true, enumerable: true, configurable: true}

// Works if the property's configurable is true
Object.defineProperty(obj, 'a', {
  value: 4,
  writable: true,
  configurable: true,
  enumerable: true
});

obj.a; // 4
```

To make a property constant, set `writable` and `configurable` to `false`.

Prevent extensions with `Object.preventExtensions(myObj)`.

To prevent extensions and prevent any additions/removals of properties, use `Object.seal`.

To seal an object and now allow any rewriting of property values, use `Object.freeze`.

When accessing properties, a `[[Get]]` operation is performed, like a function call of `[[Get]]()` on the object. `[[Get]]` first inspects the object to see if that property exists, returning the value if does. If it does not find the property, it traverses the `[[Prototype]]` chain. If that does not work, it returns `undefined`.

Similar to `[[Get]]`, there is a `[[Put]]`.

If a property is already set, the `[[Put]]` checks:
1. Is the property an accessor descriptor (explained later)
2. Does the property have a `writable` data descriptor set to `false`? If yes, silently fail in non-stict mode or throw `TypeError` in strict mode
3. Otherwise, set the value of the existing property as normal

If it is not already set on the object, it goes up the `[[Prototype]]` chain for behavior delegation. This is discussed in Chapter 5.

In ES5, you can alter the getter and setter of an object's property. In future JS versions, it will be possible to change object-level getters/setters.

Getters call a hidden function to retrieve a value. Setters call a hidden function set a value. These are called accessor descriptors.

Accessor descriptors ignore the `value` and `writable` descriptors. `configurable` and `enumerable` are still used.

```javascript
var myObj = {
  get a() {
    return 2;
  }
};

Object.defineProperty(
  myObj, // target
  'b', // property name
  // descriptor
  {
    get: function() {
      return this.a * 2
    },

    // by default, enumerable is false when creating this way, so let's change that
    enumerable: true
  }
);

myObj.a; // 2
myObj.b; // 4

myObj.a = 3;
myObj.a; // 2
myObj.b; // 4
```

Better to have a setter whenever you have a getter.

```javascript
var myObj = {
  get a() {
    return this._a_;
  },

  set a(val) {
    this._a_ = val * 2;
  }
};

myObj.a = 2;
myObj.a; // 4
```

To see if a property key exists on that object or any of its prototypes use `'keyName' in myObject`.

To see if a property key exists on the object and not check the prototypes, use `myObject.hasOwnProperty('keyName')`.

If you create a DMZ via `Object.create(null)`, it is not delegated to `Object.prototype`, so you cannot use `Object.hasOwnProperty`. The safer bet is to use `Object.prototype.hasOwnProperty.call(myObject, 'keyName')`.

Trickery:

```javascript
var arr = [1, 2];
1 in arr; // true
2 in arr; // false

// in checks for keys not values
```

There are a couple of strategies to distinguish enumerables from non-enumerables.

```javascript
var myObj = {};

Object.defineProperty(myObj, 'a', {
  value: 2,
  enumerable: true
});

Object.defineProperty(myObj, 'b', {
  value: 2,
  enumerable: false
});

for (var k in myObj) {
  console.log(k, myObj[k]);
}
// "a" 2

// Only tests if the property exists directly on the object
myObj.propertyIsEnumerable('a'); // true
myObj.propertyIsEnumerable('b'); // false

// Only tests if the property exists directly on the object
Object.keys(myObj); // ["a"]
Object.getOwnPropertyNames(myObj); ["a", "b"];
```

Similar to `forEach`, there's also `every` and `some`.

The order of iteration on an object is not guaranteed.

If you want to iterate over the values rather than the indices of an array or the properties of an object, ES6 adds the `for..of` loop. For this to work with objects, the object needs to define its own customer iterator.

```javascript
var arr = [1, 2, 3];
for (var v of arr) {
  console.log(v);
}
// 1  2  3

var it = arr[Symbol.iterator]();
it.next(); // Object {value: 1, done: false}
it.next(); // Object {value: 2, done: false}
it.next(); // Object {value: 3, done: false}
it.next(); // Object {value: undefined, done: true}
```

The `@@iterator` was left out intentionally for objects. It is possible to define your own, though.

```javascript
var myObject = {
  a: 2,
  b: 3
};

Object.defineProperty( myObject, Symbol.iterator, {
  enumerable: false,
  writable: false,
  configurable: true,
  value: function() {
    var o = this;
    var idx = 0;
    var ks = Object.keys( o );
    return {
      next: function() {
        return {
          value: o[ks[idx++]],
          done: (idx > ks.length)
        };
      }
    };
  }
} );

// iterate `myObject` manually
var it = myObject[Symbol.iterator]();
it.next(); // { value:2, done:false }
it.next(); // { value:3, done:false }
it.next(); // { value:undefined, done:true }

// iterate `myObject` with `for..of` -- now working
for (var v of myObject) {
  console.log( v );
}
// 2
// 3
```

## Chapter 4: Mixing (Up) "Class" Objects

Polymorphism: general behavior from a parent class can be overriden in a child class.

Relative polymorphism lets us reference the base behavior from the overriden behavior. Typically it means "look one level up". This is like the `super` method in Ruby.

Class theory strongly suggests that both parent and child classes share the same method name so that it overrides. This is a bad idea in JS.

Classes are just a design pattern. They are just an optional abstraction on top of code.

Java does not give you a choice if you want to use classes or not. C/C++ and PHP give both functional and class-oriented syntaxes.

Class-like syntax (such as `new`, `instanceof`, and ES6's `class`) are JS's attempts to satisfy developers' desire to have classes. This does not work out because JS's mechanics do not align well with this pattern.

When instantiating a class, the new object is a *copy* of the all the characteristics described by the class. A class is instantiated into object form by a copy operation. It makes more sense to think about the relationship of a class to an object instance -- rather than the indirect relationship between an object instance and the class it came from.

The constructor of a class belongs to the class.

In class-oriented languages, using `super` gives you direct access to the constructor of the parent class. The constructor belongs to the class. In JS, it's the reverse -- the "class" (`Foo.prototype`) belongs to the constructor (constructor function). In JS, the only relationship between a child and a parent is between the two `.prototype` objects. The constructors themselves are not directly related. This is somewhat "solved" in ES6 via `super`.

Not having multiple inheritance in JS lowers functionality but significantly lowers complexity.

Developers try to fake the missing copy behavior of classes in JS via mixins. They are either explicit or implicit.

Explicit pseudo-polymorphism is when you call the parent directly by name in the child.

```javascript
// vastly simplified `mixin(..)` example:
function mixin( sourceObj, targetObj ) {
  for (var key in sourceObj) {
    // only copy if not already present
    if (!(key in targetObj)) {
      targetObj[key] = sourceObj[key];
    }
  }

  return targetObj;
}

var Vehicle = {
  engines: 1,

  ignition: function() {
    console.log( "Turning on my engine." );
  },

  drive: function() {
    this.ignition();
    console.log( "Steering and moving forward!" );
  }
};

var Car = mixin( Vehicle, {
  wheels: 4,

  drive: function() {
    // Explicit pseudo-polymorphism here
    Vehicle.drive.call( this );
    console.log( "Rolling on all " + this.wheels + " wheels!" );
  }
} );
```

In class-oriented languages, the linkage between `Car` and `Vehicle` are established once at the top with the class definition. In JS, because of shadowing, we need to have explicit pseudo-polymorphism. This requires the code to call it explicitly and link it explicitly every time -- which is super brittle.

Even with mixins, the parent and child objects share references to functions.

If you need a complex library/utility to use mixins, it's a sign that you should go with OLOO.

Implicit mixins are where you don't use a `mixin` function, however just call it from the child.

```javascript
var Something = {
  cool: function() { /* .. */ }
};

var Another = {
  cool: function() {
    // Implicit mixin
    Something.cool.call(this);
  }
};
```

Summary: attempting to apply classes to JS is no good because...
Chapter 4:
* Classes are copies as functions whereas JS uses references -- overwriting the reference will change it for all instances
* Making a copy of a function is sometimes functional at best
* Before ES6's `super`, there was no easy was to apply relative polymorphism
* Mixins do not allow for polymorphism in JS
* Pseudo-polymorphism is brittle
Chapter 5:
* Adds confusion that functions are constructors
* Adds confusion on what `new` is actually doing
* Adds confusion that `instance.constructor` is a property on the instance, not its delegated object's property

## Chapter 5: Prototypes

Everything discussed in this chapter assumes normal behavior for `[[Get]]` and `[[Put]]`. ES6's `Proxy` overrides these and are covered later.

When a property doesn't exist on an object, the `[[Get]]` looks up the `[[Prototype]]` link.

```javascript
var obj1 = { a: 2 };
var obj2 = Object.create(obj1);
obj2.a; // 2 -- doesn't exist on obj2, so JS looks up the prototype link
```

The top-end of every normal prototype chain is `Object.prototype`.

When setting a new property on an object can have a few different scenarios. Since it is not already present on `myObject`, the `[[Prototype]]` chain is traversed, similar to `[[Get]]`. If it doesn't exist anywhere in the chain, it is created and set as expected.

If the property does exist somewhere higher in the chain, different behavior can occur. For the following scenarios, consider:

```javascript
myObject.foo = 'bar';
```

1. If a data accessor for that property has `writable: true` (aka can be written over), the new property `foo` is added directly to `myObject` -- resulting in shadowing
2. If a data accessor for that property is higher in the chain and marked as read-only, then the property creation/editing is not allowed. The only way to add the property would be to use `Object.defineProperty(..)`.
3. If a `foo` setter is found then the setter will be called. This will set it on the original object.

```javascript
var myObj = {
  get a() {
    return this._a_;
  },

  set a(val) {
    this._a_ = val * 2;
  }
};

myObj.a = 2;
myObj.a; // 4

var newObj = Object.create(myObj);
newObj.a = 5;
newObj.a; // 10
myObj.a; // 4
```

Implicit shadowing can lead to weird results. In general, be careful with delegated properties that you want to modify.

```javascript
var delegatedObj = { a: 2 };
var newObj = Object.create(delegatedObj);
newObj.hasOwnProperty('a'); // false
newObj.a++;

delegatedObj.a; // 2
newObj.a; // 3
newObj.hasOwnProperty('a'); // true -- it was added to the new object rather than modifying the linked object
```

Every function by default gets a public, non-enumerable property called `prototype` which points to an otherwise arbitrary object. Every time an object is created with `new`, that object will end up prototype linked to this "Foo dot prototype" object.

```javascript
function Foo() {
  // ...
}

var myFoo = new Foo();
Object.getPrototypeOf(myFoo) === Foo.prototype; // true
myFoo.__proto__ === Foo.prototype; // true
```

This linking can be done more simply with `Object.create(..)`.

Opposite of how classes make copies, in JS we make links between objects.

"Inheritance" with classes makes sense. "Prototypical inheritance", which is the term to describe OLOO, is the exact opposite behavior.

Functions aren't constructors. Function calls are constructor calls if and only if `new` is used. The following is JS's attempt to remedy classical inheritance.

```javascript
function Foo() {}

Foo.prototype.constructor === Foo; // true -- which causes confusion because functions aren't constructors

var myFoo = new Foo();
myFoo.constructor === Foo; // true -- this is a huge source of confusion because myFoo doesn't have a constructor property. This is actually delegated up to Foo.prototype which happens to by default have a .constructor property that points to Foo.
// In other words, it's like this:
Foo.prototype.constructor === Foo; // true -- well, yeah.

// constructor does not mean "was constructed by" -- here's an example
Foo.prototype = {}; // new object
var myNewFoo = new Foo();
myNewFoo.constructor === Foo; // false
myNewFoo.constructor === Object; // true -- because it goes up the prototype link again and the object literal above has a constructor property with a value of the built-in Object function
```

If you insist on linking two functions to fake classes, make sure to do the following:

```javascript
function Foo() {}
function Bar() {}

// Pre-ES6 -- throws away default Bar.prototype
Bar.prototype = Object.create(Foo.prototype);
// If you rely on .constructor, will need to be manually added

// ES6+ -- modifies existing Bar.prototype
Object.setPrototypeOf(Bar.prototype, Foo.prototype);

// Does not work as expected:
Bar.prototype = Foo.prototype;
Bar.prototype = new Foo(); // induces unwanted side effects
```

`myFoo instanceof Foo` only works if `myFoo`'s chain points to `Foo.prototype` at some point. This can break down if using a `.bind(..)` as the hard-bound function will not have a `.prototype` property.

Determing relationship is much easier with OLOO.

```javascript
var delegatedObj = {};
var newObj = Object.create(delegatedObj);
delegatedObj.isPrototypeOf(newObj); // true -- in the entire chain of newObj, delegatedObj does appear

Object.getPrototypeOf(newObj) === delegatedObj; // true -- works in ES5
newObj.__proto__ === delegatedObj; // true -- doesn't work in all browsers
```

`.__proto__` wasn't standardized until ES6. It exists on `Object.prototype` as a special `[[Get]]`. It is also a settable property, however you should not change the prototype of an existing object.

Using `Object.create(null)` creates an empty object that has no prototype chain. These special, empty-prototype objects are typically used for dictionaries so there are no surprises via delegation. Dictionaries are solely used to store data as objects.

`Object.create` in ES5+ can take an additional argument to create properties with property data descriptors.

```javascript
var anotherObject = {
  a: 2
};

var myObject = Object.create( anotherObject, {
  b: {
    enumerable: false,
    writable: true,
    configurable: false,
    value: 3
  },
  c: {
    enumerable: true,
    writable: false,
    configurable: false,
    value: 4
  }
});
```

Do not think about the prototype chain as a set of fallbacks "in case something is missing". If that is how your code is written, it will seem more magical and harder to understand/maintain.

When future developers see the new API, they will have some trouble maintaining it as delegated properties aren't visible on the object. To get around this, use an internal delegation wrapper.

```javascript
var delegatedObj = { a: 2 };
var newObj = Object.create(delegatedObj);
console.log(newObj); // Object {}

// Solution: wrapper
newObj.getA = function() {
  return this.a;
}
newObj.getA(); // 2
```

## Chapter 6: Behavior Delegation

Avoid naming functions the same at different levels. Name collisions create awkward/brittle code. Push yourself to come up with more specific names to increase clarity.

OLOO's object creation (`var myBtn = Object.create(widget)`) and initialization (`myBtn.config({ name: 'yeah' })`) are two different steps rather than one when using a constructor. This is a benefit because you can do it in two different steps rather than all at once.

## Appendix A: ES6 `class`

Benefits:
1. It removes `.prototype` cluttering
2. It removes the need to link prototypes
3. Gives relative polymorphism
4. Properties can't be set on parent classes, which before was a cause of confusion on how to do this (`childClass.counter++` would change it on the child and not the parent). No more confusion with this since it's not possible
5. It's much easier now to `extend` built-in object sub-types such as `Array` and `RegExp`

Cons:
1. JS doesn't make copies of functions like classes expect. Changing the parent affects all children
2. If you want to add properties to the parent, you still need to use `.prototype`
3. `super` is not bound dynamically but rather set at declaration time. `this` is set at call time but `super` is not. This means it's not possible to re-use and pass around functions with `super` unless you expect `super` to always to get the `[[HomeObject]]` defined at initialization (`class C extends P`)
4. Requires the use of `.toMethod` to change the relative position of `super`
5. `class` implies that it's not possible to change an object after creation -- it's static. But JS is dynamic and that's where it's powerful
