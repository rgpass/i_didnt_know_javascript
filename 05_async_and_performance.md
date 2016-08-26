# I Didn't Know JavaScript: Async & Performance

## Chapter 1: Asynchrony: Now & Later

Only one part of your program will execute *now* and all the rest are done *later. For example, a function is done later.

Syncronous Ajax requests should be avoided always. They block all user interaction (including scrolling!).

Browsers have different host objects. One host object is `console.*`. Some browsers handle these calls sync and others async (since I/O is slow and blocking).

If you notice that `console.log` has different results, either use `debugger` or create a snapshot via `JSON.stringify`.

The event loop is a FIFO queue.

`setTimeout` doesn't put your callback into the event loop queue. What it does it wait until the timer expires then it adds to the queue. This is why timeouts won't execute after one second when `1000` is set -- it just means the time to be added to the queue is one second. If there are other things in the queue, they will be executed first.

Async: now vs later.
Parallel: both happening now simultaneously

Processes and threads execute independently. Processes happen on separate processors or computers. Multiple threads can share the memory of a single process.

Threaded programming is complicated as it requires special steps to make sure that interruption/interleaving doesn't cause weird results.

JS is run-to-completion, which means that a code chunks will finish before another one starts. If JS was multi-threaded, you could weave chunks within each other and have indeterministic behavior.

Race condition: function-ordering nondeterminism where two functions are "racing" to finish first. You cannot predict which one will finish first.

Concurrency: "processes" happening at the same time, regardless of where their operations are happening. This is higher-level than operation-level parallelism (such as threads).

Note: "process" means a general process, not the process in a CS perspective

Individual events run sequentially on the event loop. "Processes" can happen concurrently.

Noninteracting: code written such that concurrent events do not interact with each other -- thus no opportunity for race conditions

Fixing a race condition via a Gate:

```javascript
var a, b;

function foo(x) {
  a = x * 2;

  // Gate:
  if (a && b) {
    baz();
  }
}

function bar(x) {
  b = x * 2;

  // Gate:
  if (a && b) {
    baz();
  }
}

function baz() {
  console.log(a + b);
}

// Note: Ajax is pseudo-code of an Ajax lib
Ajax('url.com/1', foo);
Ajax('url.com/2', bar);
```

If you only want something to fire once, use a Latch (sometimes called a Race, where only the winner gets called):

```javascript
var a;

function foo(x) {
  // Can't do if (!a) because a could be zero
  // Latch:
  if (a == undefined) {
    a = x * 2;
    baz();
  }
}

function bar(x) {
  // Latch:
  if (a == undefined) {
    a = x / 2;
    baz();
  }
}

function baz() {
  console.log(a + b);
}

// Note: Ajax is pseudo-code of an Ajax lib
Ajax('url.com/1', foo);
Ajax('url.com/2', bar);
```

Cooperative concurrency: take long-running "processes" and break them unto steps or batches so other concurrent "processes" have a chance to interleave their operations into the event loop queue

```javascript
// Long running process -- lets assume a huge piece of data such as millions of pieces
// Let's say we just want to double the data

var res = [];

function responseBlocking(data) {
  res = res.concat(data.map(function(val) {
    return val * 2;
  }));
}

// Depending on one which wins the race, the other will be significantly delayed
Ajax('url.com/1', responseBlocking);
Ajax('url.com/2', responseBlocking);


function responseCooperative(data) {
  // 1000 pieces at a time
  var chunk = data.splice(0, 1000);

  // Just double the chunk
  res = res.concat(chunk.map(function(val) {
    return val * 2;
  }));

  // Check if anything left to process. If there is come back to it.
  if (data.length > 0) {
    setTimeout(function() {
      responseCooperative(data);  
    }, 0);
  }
}
```

Cooperative concurrency has the issue of being hard to manage the ordering. Use a Gate or a Latch for these.

Note: Two `setTimeout(.., 0)` statements will not necessarily fire in the order expected. This is because this statement doesn't mean "Add to the queue now" but rather "Add to the queue at your next opportunity".

As of ES6, there's a new concept layered on top of the event loop queue called the job queue. The job queue is a queue hanging off the end of every tick in the event loop queue. It's like adding tasks to the end of the current tick. Essentially you're saying, "Later, but as soon as possible and before other stuff happens."

The API for jobs is still being worked on. Here's an example of it using a made up `schedule` to add to the job queue.

```javascript
console.log(1);

setTimeout(function() {
  console.log(2);
}, 0);

schedule(function() {
  console.log(3);

  schedule(function() {
    console.log(4);
  });
});

// 1, 3, 4, 2
```

The async behavior of Promises is based on Jobs, so it's necessary to understand this.

## Chapter 2: Callbacks

"Events": async function invocations.

```javascript
// A
setTimeout(function() {
  // C
}, 1000);
// B
```

This is best described as: "Do A, set a timer for 1000 milliseconds, do B, then after timer fires do C."

Callback hell has nothing to do with indentation. You can write the code without nesting (just making it flat via functions and scoping). It's still challenging to navigate because we have to reason about when it will be executed.

If the function is executed syncronously, it will often behave in a different order than async.

Callback hell has to do with pre-planning the code for recovery/retrying/forking flow. If something fails, it's a lot of code duplication to retry.

Callbacks articulate code in a way that our brains can't easily follow. We have to jump back and forth.

Passing a callback to a 3rd-party library is "inversion of control" -- the library now has control of the function.

The issue: what if it's never called? Called too early? Too late? Too few or too many times? What if it doesn't pass along the correct params? Or it swallows errors?

All these issues will have to be addressed with *every* callback.

With our own code, with a function like `addTwoNumbers`, we can add checks to make sure that the two values put in are numbers and to throw errors otherwise. Callbacks require us to implement these trust mitigations everywhere -- and that's just for the part we control.

ES6 Promises use a success/failure API, such as `ajax('url.com', successFn, failureFn);`. Node style is to do `function response(err, data) {` where `err` is empty or falsey if no errors. Neither of these mitigate the multi-fire issues nor the sync-or-async.

All callbacks should be fired asyncronously. If you design a library, make sure this happens.

## Chapter 3: Promises

Callbacks: lack sequentiality and lack trustability.

Note: "immediately" refers to Job queue behavior rather than right-this-moment.

Adding two async values via callbacks example below. This makes it async, however doesn't deal well with a requirements change if you need to add more inputs. Also requires a gate.

```javascript
function add(getX, getY, cb) {
  var x;
  var y;

  getX(function(xVal) {
    x = xVal;

    if (y != undefined) {
      cb(x + y);
    }
  });

  getY(function(yVal) {
    y = yVal;

    if (x != undefined) {
      cb(x + y);
    }
  });
}

add(fetchX, fetchY, function(sum) {
  console.log(sum);
});
```

Adding two values using Promises. Note how Promises behave as future values. It can be considered a value-promise (i.e. a promise of a value, like a receipt for a hamburger).

```javascript
function add(xPromise, yPromise) {
  // Promise.all takes in an array of Promises
  // It is not resolved until all have been resolved
  // It returns a new promise that waits all to finish
  return Promise
    .all([xPromise, yPromise])

    // When that promise is resolved, we add the received values
    .then(function(values) {
      return values[0] + values[1];
    });
}

// fetchX and fetchY return promises for their respective values
// The returned promises can resolve now or later, the behavior is always like the Job queue
add(fetchX(), fetchY())

  // This .then is operating on the second promise returned, aka the promise that sums the values
  .then(function(sum) {
    console.log(sum);
  });

// The implicit return value of the add function call is a Promise as well
```

The `.then` can take two arguments. The first is called the fulfillment handler and the second is the rejection handler.

```javascript
// Continued from above
add(fetchX(), fetchY())
  .then(
    function(sum) {
      console.log(sum);
    },
    function(err) {
      console.log(err);
    }
  }
);
```

Promises are time-independent. They can be combined in predictable ways.

Promises can only be resolved once. Once resolved, it becomes immutable and stays that way forever. With callbacks, the 3rd-party library may call the callback multiple times, however with Promises it's only once.

Callbacks require you to pass the callback to the function being invoked, only notifying you of completion by invoking the callback. With Promises, it's reversed; you are "subscribing" to be "notified" when it's resolved. You can do what you want and don't need to pass in a function.

```javascript
// Playing around with the Promise API
var fetchX = new Promise(function(resolve, reject) { resolve(3) });

var fetchY = function() {
  return new Promise(function(resolve, reject) {
    resolve(4);
  });
}

fetchX.then(function(val) {
  console.log(val); // 3
});

var valuesPromise = Promise
  .all([fetchX, fetchY()])
  .then(function(values) {
    console.log(values);
    return values;
  });
```

The pattern for `new Promise(function(..) { .. })` is called ["revealing constructor"](http://domenic.me/2014/02/13/the-revealing-constructor-pattern/). Functions passed in are executed immediately.

Promises can be used for control-flow.

```javascript
function foo(x) {
  // ... stuff that takes awhile

  return new Promise(function(resolve, reject) {
    // Eventually call resolve(..) and/or reject(..)
    resolve(x); // for funsies
  });
}

function bar(val) {
  console.log('Bar executing');
  console.log('Foo is ' + val);
}

function oopsBar(err) {
  console.log('Error happened before Bar');
  console.log(err);
}

// .. same two functions for Baz

var p = foo(30);

// This is splitting/forking
// Bar and Baz only get called if foo succeeds
p.then(bar, oopsBar);
p.then(baz, oopsBaz);

// This is NOT the same as
p
  .then(bar, oopsBar)
  .then(baz, oopsBaz);
```

If you want a pattern where `bar` and `baz` get called regardless if `foo` succeeds:


```javascript
// ... same foo as above

function bar(fooPromise) {
  fooPromise.then(function(val) {
    // Foo has finished
    console.log('Bar executing');
    console.log('Foo is ' + val);
  },
  function(err) {
    console.log('Something went wrong with Foo');
    console.log(err);
  });
}

var p = foo(30);
bar(p);
```

These two strategies (passing in a promise or using `.then`) really differs on the error handling. In the first way, `bar` is only called if `foo` succeeds. In the second way, `bar` handles its own error. 

It's not possible to test for promises using `p instanceof Promise` as this would require the library you're using to be using ES6 Promises (which probably isn't happening). The best way to recognize promises is if it has a `.then` method on it. Duck-typing yay!

The Promises system assumes that anything with a `.then` property is a "thenable". This messes with a bunch of libraries.

Trust Issues from callbacks:
* call the callback too early
* call the callback too late
* call the callback too few (or never) or too many times
* fail to pass along any necessary environment/params
* swallow any errors/exceptions that may happen

Too early aka the Zalgo effect aka when code that looks async runs sync or vice-versa. Promises solve this because Promises cannot be *observed* synchronously. If a Promise was already resolve and you call `.then` on it, the callback you put in for the fulfillment handler will *always* be called async.

Too late. When a Promise is resolved, all registered `.then` callbacks will be called in order and will be called at the next async opportunity (i.e. Job queue). This order cannot be changed. For example:

```javascript
p.then(function() {
  p.then(function() {
    console.log('C');
  });

  console.log('A');
});

p.then(function() {
  console.log('B');
});
// A B C
```

Promise scheduling has its quirks. When resolving a promise, you are putting adding to the Job queue to unwrap it i.e. you are unwrapping it async so it's added to the end of the queue. Example:

```javascript
var p3 = new Promise(function(resolve, reject) {
  resolve('B');
});

var p1 = new Promise(function(resolve, reject) {
  resolve(p3);
});

var p2 = new Promise(function(resolve, reject) {
  resolve('A');
});

function printVal(v) {
  console.log(v);
}

p1.then(printVal);
p2.then(printVal);
// A B, not B A
```

DO NOT RELY ON ORDERING/SCHEDULING OF PROMISES.

Never calling the callback. You can use a utility for timing out a Promise.

```javascript
function timeoutPromise(delay) {
  return new Promise(function(resolve, reject) {
    setTimeout(function() {
      reject('Timeout!');
    }, delay);
  });
}

// Promise.race will resolve the first one
Promise
  .race([
    foo(), // attempt foo()
    timeoutPromise(3000) // fail after 3 seconds
  ])
  .then(function() {
    // foo was fulfilled in time
  }, function(err) {
    // either
    // 1. foo was reject
    // 2. it timed out
    // Inspect err to know which
  });
```

Too many times. Promises can only be resolved once. You can however have a `.then` registered on it that does the same thing, such as `p.then(f); p.then(f);`.

Failing to pass on any params/environment. Promises can only be passed one param. This param is passed to all registered `.then`ables. Multiple params are silently ignored. Not putting anything in `resolve()` will make it `undefined`.

Swallowing errors. If you follow the API, you'll pass in `reject(yourErrObj)`. If at any point in the creation of a Promise or observation of a resolution of a Promise there is a JS exception error (`TypeError`, `ReferenceError`), the exception will be caught and will force the Promise to be rejected.

```javascript
var p = new Promise(function(resolve, reject) {
  foo.bar(); // foo is not defined
  resolve(30); // never gets here
});

p.then(function fulfilled() {
  // never gets here
}, function rejected(err) {
  // err is an TypeError exception object
  console.log(err);
});
```

Promises even turn errors into async events -- avoiding Zalgo.

Promises have one issue: they rely on other things being Promises. This is an issue because all your code could fail once one library doesn't have Promises. Solution: ES6 has `Promise.resolve` which turns any value into a Promise.

Passing in a value to `Promise.resolve` makes it a Promise. Passing a Promise to a Promise just returns the same Promise.

```javascript
var p1 = Promise
  .resolve(30)
  .then(function(v) {
    console.log(30);
  });

// same as (minus .then above)
var p2 = new Promise(function(resolve, reject) {
  resolve(30);
});

var p3 = Promise.resolve(p1);

p1 === p3; // true
```

`Promise.resolve(..)` also unwraps `.then`ables and resolves it into a Promise.

```javascript
var myObj = {
  then: function(cb) {
    cb(30);
  }
};

Promise
  .resolve(myObj)
  .then(function(val) {
    console.log(val); // 30
  });
```

Always wrap potentially non-Promises in `Promise.resolve` to avoid Zalgo.

```javascript
// What if foo is sync? Eek!
foo(30)
  .then(function(val) {
    console.log(val);
  });

// async yay
Promise
  .resolve(foo(30))
  .then(function(val) {
    console.log(val);
  });
```

Returning inside of a Promise chain (aka a `.then`) creates a new Promise and sets the value for the next chain.

```javascript
var p1 = new Promise(function(resolve, reject) {
  resolve(30);
});

p1
  .then(function(val) {
    // Return a value right away, but knowing that it will be re-wrapped into a Promise
    // return val * 2;

    // If we wanted to do something async, we could either return a new Promise or do the Promise.resolve
    // Much safter to just always do Promise.resolve
    return Promise.resolve(val * 2);
    // Note: remember that Promise.resolve will either keep the same Promise if it's already one or make it one
  })
  .then(function(val) {
    console.log(val); // 60
  });
```

An example of passing data through:

```javascript
function request(url) {
  return new Promise(function(resolve, reject) {
    // Making resolve the callback will make sure it's resolved
    ajax(url, resolve)
  });
}

request('url.com/1')
  .then(function(response1) {
    return request('url.com/2?' + response1);
  })
  .then(function(response2) {
    console.log(response2);
  });
```

You can correct errors yourself or there is an assumed error handling.

```javascript
// Correcting errors myself
p1
  .then(function() {
    foo.bar();
  })
  .then(function() {
    // never gets here
  }, function(err) {
    // can do what you want here
    // let's just return a val
    return 30;
  })
  .then(function(val) {
    console.log(val);
  });

p1
  .then(function() {
    foo.bar();
  })
  .then(function() {
    // never gets here
  }
  // there is an assumed rejection handler here that looks like
  // function(err) {
  //   throw err;
  // })
  );

// There's always an assumed fulfillment handler, looks like:
function(val) {
  return val;
}
```

Key notes that enable control flow:
* A `.then` call on a Promise produces a new Promise
* Returning a value or throwing an error causes the Promise to resolve accordingly
* If a Promise is returned, the fulfillment and rejection handlers unwrap it so that whatever its resolution is will become the resolution of the chained Promise

Arguments for `new Promise` make sense as `resolve` and `rejected`. Arguments for `.then` make sense to be `fulfilled` and `rejected`.

Can use `Promise.reject('Oops')` to create a simulated rejection. Note: this does not unwrap a Promise like `resolve` will. Need to pass in a real value to `.reject` only.

Promises assume by default that you want errors to be swallowed up. If you forget to observe that behavior, you're out of luck.

One solution is to always end Promise chains with `.catch(..)`. The only issue with this: what if you have an error inside of your `.catch`? Sticking another `.catch` doesn't work nor does it solve the problem.

```javascript
var p = Promise.resolve(30);

p
  .then(function fulfilled(val) {
    console.log(val.toLowerCase()); // will throw an error
  }
  // Don't provide your own error handler so the default handler gets substituted
  )
  .catch(function(err) {
    console.log(err); // TypeError
  });
```

If any of `Promise.all([..])` promises fail, it is immediately rejected and all results are discarded from any other promise. Always attach a rejection/error handler to every promise, especially on `Promise.all`.

`Promise.race([..])` is similar to the Latch pattern. Note: nothing to do with race condition, but rather the first to finish wins. The fulfillment value is a single message, not an array like `.all`. Rejections are silently ignored.

Sending an empty array to `.race` will cause it to never resolve.

Most Promise libraries have their own implementations of `.first`, `.last`, `.any`, `.none`, and `.map`. It is possible to make your own utilities for these as well.

If you need a polyfill for native promises, check out "Native Promise Only" (http://github.com/getify/native-promise-only).

`p.catch(rejected)` is the same as `p.then(null, rejected)`.

One downside of Promises is that you have to bring it all into one value to be returned. All extra params are silently ignored. The solution: destructuring.

```javascript
Promise
  .all([getX, getY])
  .then(function([x, y]) {
    console.log(x, y);
  });
```

When working with events that need to happen multiple times (such as a button being clicked), you can set it so that a whole new Promise sequence is executed. This is necessary because each Promise are only resolved once. The downside to this approach is that you do not have separations of concerns. You must define the entire chain inside the event handler.

"A code base in motion (with callbacks) will remain in motion (with callbacks) unless acted upon by a smart, Promises-aware developer."

If you want to start converting callbacks to Promises, check the `Promise.wrap` utility in this chapter. It essentially makes a function that returns a function that returns a Promise. This conversion of callbacks to Promises is called "lifting" or "promisifying".

Performance issues with Promises are small and considering the value they bring they are almost always worth using.

## Chapter 4: Generators

Generators make it possible to express async flow control in a sequential, sync-looking fashion.

Generators are cooperative because they "yield" control back to the event loop. They allow for concurrency as well since they do not follow run-to-completion like all other JS functions.

Consider the simple example below.

```javascript
var x = 1;

function foo() {
  x++;
  bar();
  console.log(x);
}

bar() {
  x++;
}

foo(); // 3
```

What if we wanted to pause it on the line with `bar()` to execute some async behavior? Generators to the rescue.

```javascript
var x = 1;

function *foo() {
  x++;
  yield; // aka pause!
  console.log('x: ' + x);
}

function bar() {
  x++;
}

// Construct an iterator to control the generator
// This does not execute *foo(), it merely constructs the iterator that will control its execution
var iterator = foo();

// Start foo()
iterator.next();
// At this point, foo is still running and is in a paused state
x; // 2
bar();
x; // 3
it.next(); // x: 3 -- this line resumes the generator
```

Generators, just like functions, can take inputs and can return a value. The syntax for returning a value is slightly different.

```javascript
function *foo(x, y) {
  return x * y;
}

var iterator = foo(3, 10);
var result = iterator.next(); // needs to be started
result.value; // 30
```

`.next` takes in args!!

```javascript
function *foo(x) {
  var y = x * (yield);
  return y;
}

var iterator = foo(3);

// Start foo()
iterator.next();

var result = iterator.next(10);
result.value; // 30
```

Note that `yield` pauses in the middle of an assignment statement -- powerful.

For all generators, you will have one more `.next` call than `yield`s. This is because the first `.next` always starts the generator.

Looking at the previous example, you would assume that `next` is a one-way communication to `yield`. But it's not.

```javascript
function *foo(x) {
  var y = x * (yield 'Hello!!');
  return y;
}

// Construct
var iterator = foo(3);

// Start
var res1 = iterator.next();
// Get value passed to yield
res1.value; // "Hello!!"
// Note: res1.next() doesn't work

var res2 = iterator.next(10);
res2.value; // 30
```

Multiple instances of one generator can be created. They are their own (child) scopes, so they can interact.

Iterator: a well-defined interface for stepping through a series of values from a producer. The interface is to call `.next()` each time you want the next value from the producer. See `something` below.

Iterable: an `object` that contains an iterator. The iterable must have a function on with the name being the special ES6 symbol value `Symbol.iterator`.

```javascript
var gimmeSomething = (function() {
  var nextVal;

  return function() {
    if (nextVal === undefined) {
      nextVal = 1;
    }
    else {
      nextVal = (3 * nextVal) + 6;
    }

    return nextVal;
  }
})();

gimmeSomething(); // 1
gimmeSomething(); // 9
gimmeSomething(); // 33


// Implementation of the standard interator.
var something = (function() {
  var nextVal;

  return {
    // [] -- ES6's new "computed property name"
    // Here we use a ES6 predefined special Symbol
    [Symbol.iterator]: function() {
      // By returning this, we are now making something that is not just an iterable but also an iterator.
      return this;
    },
    next: function() {
      if (nextVal === undefined) {
        nextVal = 1;
      }
      else {
        nextVal = (3 * nextVal) + 6;
      }

      return {
        done: false,
        value: nextVal
      }
    }
  }
})();

something.next().value; // 1
something.next().value; // 9
something.next().value; // 33
```

Because we have defined an iterator and always have `done:false`, we can now use `for..of`.

```javascript
// for..of automatically calls .next()
// It expects something to be an iterable, so it looks for and calls something's Symbol.iterator function
for (var v of something) {
  console.log(v);

  // Prevent an infinite loop
  if (v > 500) {
    break;
  }
}
// 1 9 33 105 321 969
```

Arrays have default iterators in `for..of` loops.

```javascript
// Iterable
var arr = [1, 3, 5, 7];

// for..of automatically calls the Symbol.iterator function to construct an iterator
for (v of arr) {
  console.log(v); // 1 3 5 7
}

// Manual way
var iterator = arr[Symbol.iterator]();

iterator.next().value; // 1
iterator.next().value; // 3
iterator.next().value; // 5
iterator.next().value; // 7
iterator.next().value; // undefined
```

From the context of iterators, generators are producers of values that we can get one at a time by calling `.next()`.

A generator function creates an iterator (not an iterable).

```javascript
function *genSomething() {
  var nextVal;

  // Can do this infinite loop with generators no-problem as you are in control of when this runs
  while (true) {
    if (nextVal === undefined) {
      nextVal = 1;
    }
    else {
      nextVal = (3 * nextVal) + 6;
    }

    yield nextVal;
  }
}

var genIt = genSomething();
genIt.next().value; // 1
genIt.next().value; // 9
genIt.next().value; // 33
```

A generator's iterator also has a `Symbol.iterator` function which is basically `return this;`. A generator's iterator is also an iterable.

Specifying a `try..finally` inside a generator will always run when externally completed. This is good for cleaning up resources, such as database connections.

```javascript
function *cleanUpGen() {
  try {
    var nextVal;

    while (true) {
      if (nextVal === undefined) {
        nextVal = 1;
      }
      else {
        nextVal = (3 * nextVal) + 6;
      }

      yield nextVal;
    }
  }
  finally {
    console.log('Cleaning up...');
  }
}

for (var v of cleanUpGen()) {
  console.log(v);

  if (v > 100) {
    break;
  }
}
// 1 9 33 105 Cleaning up...

// Can also cancel it yourself
var it = cleanUpGen();
for (var v of it) {
  console.log(v);

  if (v > 100) {
    // Manually complete the iterator via .return on the iterator
    // It also sets the returned value to whatever we pass in
    console.log(it.return('Hello Mate').value);

    // No break needed here
  }
}
// 1 9 33 105 Cleaning up... Hello Mate
```

Generators can be iterated async and are a better solution than callbacks.

```javascript
// Callback version
function cbFoo(x, y, cb) {
  ajax(
    'url.com/1?x=' + x + '&y=' + y,
    cb
  );
}

cbFoo(11, 31, function(err, text) {
  if (err) {
    console.log(err);
  }
  else {
    console.log(text);
  }
});


// Same thing but with a generator
function *foo(x, y) {
  ajax('url.com/1?x=' + x + '&y=' + y, function(err, data) {
    if (err) {
      // Throw it into *main()
      // We can catch errors sync now because Generators pause in execution
      it.throw(err);
    }
    else {
      // Keep going with *main() with data
      it.next(data);
    }
  });
}

function *main() {
  try {
    // Looks sync but is async!
    var text = yield foo(10, 20);
    console.log(text);
  }
  catch (err) {
    console.log(err);
  }
}

// Create iterator
var it = main();
it.next();
```

Combining generators with promises === Big Win! Generators: sync looking async code. Promises: trustable and composable. Best way to use them is to `yield` a Promise and wire that Promise to control the generator's iterator.

```javascript
function foo(x, y) {
  // return request('url.com/1?x=' + x + '&y=' + y);
  return Promise.resolve(x + y);
}

function *main() {
  try {
    var text = yield foo(10, 30);
    console.log(text);
  }
  catch (err) {
    console.error(err);
  }
}

// Create iterator
var it = main();

// Go to the first yield, pause, grab its value
var p = it.next().value;

p
  .then(function(text) {
    // Continue along, pass in text to be used in yield foo line
    it.next(text);
  }, function(err) {
    it.throw(err);
  });
```

What if we want to Promise-drive a generator such that if a Promise comes out, it should we wait on its resolution before continuing? What if an error occurs? Find a utility that covers these scenarios. Here's an example of our `run`:

```javascript
function run(gen) {
  var args = [].slice.call(arguments, 1), it;

  // Initialize the generator using the current context
  it = gen.apply(this, args);

  return Promise
    .resolve()
    .then(function handleNext(value) {
      // Go to next value
      var next = it.next(value);

      return (function handleResult(next) {
        // Is the generator completed running?
        if (next.done) {
          return next.value;
        }
        else {
          return Promise
            .resolve(next.value)
            .then(
              // Resume the async loop on success
              // Sends the resolved value back into the generator
              handleNext,

              // If `value` is a rejected promise, pass it back to the generator for it to handle the error
              function handleErr(err) {
                return Promise
                  .resolve(it.throw(err))
                  .then(handleResult)
              }
            );
        }
      })(next);
    });
}
```

If you need to get two async results, combine them into a third async, and print a result, do something like this:

```javascript
function *foo() {
  var results = yield Promise.all([
    request('url.com/1'),
    request('url.com/2')
  ]);

  var r1 = results[0];
  var r2 = results[1];

  var r3 = yield request('url.com/3?r1' + r1 + '&r2=' + r2);

  console.log(r3);
}

function *gpFoo() {
  var results = yield Promise.all([
    Promise.resolve(10),
    Promise.resolve(30)
  ]);

  var [r1, r2] = results;

  var r3 = yield Promise.resolve((r1 + r2) * 2);

  console.log(r3);
}

// previously defined
run(foo);

run(gpFoo); // 80
```

Be cautious about how many Promises you put in Generators. If you put in too much, you're missing the value of sync-looking code that Generators offer. The best solution is to put the Promises into another function then `yield` it in.

```javascript
function bar(url1, url2) {
  return Promise.all([
    request(url1),
    request(url2)
  ]);
}

function *foo() {
  var results = yield bar('url.com/1', 'url.com/2');

  // ... same as above
}
```

Can have one iterator take control of another.

```javascript
function *foo() {
  console.log('*foo() starting');
  yield 3;
  yield 4;
  console.log('*foo() finished)');
}

function *bar() {
  yield 1;
  yield 2;

  // This transfers iterator control over to the other generator -- notice the * syntax
  yield *foo(); // yield delegation to another generator
  yield 5;
}

var it = bar();
it.next().value; // 1
it.next().value; // 2
it.next().value; // "*foo() starting"  3
it.next().value; // 4
it.next().value; // "*foo() finished"  5
```

Can even do something like `yield *[1, 2, 3]` as the array is an iterable.

Thunk: a function with no params that wraps around a another function that has params

```javascript
function foo(x, y) {
  return x + y;
}

function fooThunk() {
  return foo(3, 4);
}

fooThunk(); // 7
```

## Chapter 5: Program Performance

Web Workers allow for another thread. This way you can have the main UI thread another thread. This is a browser (or other host environment) feature.

To instantiate a Worker:

```javascript
var w1 = new Worker('url.com/my-worker.js'); // Has to be a JS file
```

The `w1` Worker object is an event listener and a trigger. You can subscribe to events sent by this Worker. You can also send events to this Worker. Here is an example of the `"message"` event.

```javascript
// In the main thread
w1.addEventListener('message', function(evt) {
  // evt.data
});

w1.postMessage('something cool to say');


// In the worker file
addEventListener('message', function(evt) {
  // evt.data
});

postMessage('a cool reply here');
```

Call `w1.terminate()` to stop it immediately. It's similar to closing a browser tab.

Workers do not have access to the global vars on the UI thread.

Common uses for Workers:
* processing intensive math calculations
* sorting large data sets
* data operations (compression, audio analysis, image pixel manipulations, etc)
* high-traffic network communications

Structured Cloning Algorithm: the process use to copy/duplicate an object passed to/from a Worker.

Transferable Objects: a better solution where the ownership of an object is transferred, but the data itself is not moved. The original thread loses the ability to manipulate the data, removing the hazard of threaded programming shared scope.

Browsers that do not have Transferable Objects degraded nicely (at the cost of performance) to a Structured Cloning Algorithm.

Shared Worker: with a normal worker, loading a new tab will create a new Worker instance. With Shared Workers, all tabs shared the same Worker. This is useful for web socket connections.

Shared Workers survive termination of a port connection if there are any other connections still alive. On the other hand, Workers terminate once their initiating program is terminated.

SIMD: Single Instruction, Multiple Data. Processing multiple bits of data in parallel. Will probably be around in ES7.

Task parallelism : Web Workers :: Data parallelism : SIMD

asm.js -- a subset of JS that can be aggressively optimized. It avoids hard to optimize code (garbage collection, coercion, etc). There is a spec (http://asmjs.org/spec/latest/).

asm.js is a style of coding, however it's best to use a build step with it.

```javascript
// normal code -- requires the engine keep track of what data type is being used
var a = 30;

// ...

var b = a;


// asm.js way
var a = 30;

// ...

var b = a | 0; // binary OR with 0 has no effect other than make sure it's a 32-bit integer. This tells the engine that it's going to be a 32-bit integer.

(a + b) | 0; // Faster integer addition since it does not use a floating point
```

Heap: reserved spot in memory where variables can already be used without asking for more memory or releasing used memory. This way there is no memory churn. 

```javascript
// "heap" is likely a typed ArrayBuffer
var heap = new ArrayBuffer(0x10000); // 64k heap
```

# Chapter 6: Benchmarking & Tuning

Classic benchmarking test:

```javascript
var start = Date.now();

// Some process that takes awhile

var end = Date.now();

console.log('Duration: ', end - start);
```

This doesn't always work because if it's less than 1ms, it won't show up. Also, some browsers only update the clock every 15ms.

Also, if the system or engine had interference then the data is not true.

You need to have a high quantity of tests run. The better approach is to run the tests for a preset amount of time and see how many times it can do that operation.

Benchmark.js is a tool made by some statisticians to help. It's considered the best benchmarking tool.

Because of engine optimizations, most tests for performance are irrelevant -- the engine may discard a variable that you think you're testing or it may convert `Number('12')` to `12` prior to testing. It may not choose to optimize at 10,000 tests but will at 10 million.

To get real info, get real data -- aka test in the real world. Brand new laptop using Chrome >>> Mobile using Chrome. Full battery Mobile >>> 2% battery Mobile. Chrome may be faster than Firefox.

Use jsPerf (http://jsperf.com) to test your real site. It creates a link that you can share with anyone.

Good testing requires the author to analyze the differences between test cases. Unintentional differences will skew results.

Because of the way engines optimize, you can't think of the executed code being literally what you typed. You tell the engine what you want and it will do its job to optimize it. This makes testing performance a challenge.

Often times the engine is smarter than you. Trying to out-smart it (such as caching the length of the `for` loop rather than calculating it each time) will often result in *slower* code.

The JS spec doesn't require anything in terms of performance except for ES6's Tail Call Optimization.

V8 optimizations actually have noticeable performance increases. Reference (https://github.com/petkaantonov/bluebird/wiki/Optimization-killers). However, there's a likely chance that V8 will upgrade so that your optimizations are slower -or- that V8 will be replaced by something. This will often result in your optimizations being sub-optimal.

"There is nothing more permanent than a temporary hack." "Chances are, the code you write now to work around some performance bug will probably outlive the performance bug in the browser itself."

"premature optimization is the root of all evil." The critical code (estimated at 3% of what you write) should be optimized, but most optimizations aren't necessary and cause more trouble later.

Tail call: calling another function (`bar`) from inside a function (`foo`) where `bar` is the last line of `foo`.

```javascript
function foo(x) {
  return x;
}

function bar(y) {
  return foo(y + 1); // tail call -- nothing to do once foo is done
}

function baz() {
  return 1 + bar(29); // not a tail call -- still need to add 1 after bar is called
}

baz(); // 30
```

TCO: the engine doesn't create a new stack frame for `foo` when called from inside `bar`. It realizes that it's the last part of a function and then uses `bar`'s stack. Faster and uses less memory.

This is beneficial in recursion where an equation may require 1,000's of stack frames but with TCO it's just one.

```javascript
// Not TCO optimized
function factorial(n) {
  if (n < 2) return 1;
  return n * factorial(n - 1);
}

// TCO optimized
function factorialTCO(n) {
  function fact(n, res) {
    if (n < 2) return res;

    return fact(n - 1, n * res);
  }

  return fact(n, 1);
}
```

## Appendix A: *asynquence* Library

Sequence: a series of steps (async and sync) on top of Promises that complete linearly.

Step: wrapper around a Promise.

Every step is wired using Promises to make them async.

With Promises, the focus is on the individual step. With a sequence, it focuses on long chains/steps.

If you want a step to result in a success no matter what or you want a retry/until loop, it's a challenge to do this yourself.

If you feel like you're writing Promise boilerplate code, look into this library. It's a pretty powerful abstraction for working with Promises.

## Appendix B: Advanced Async Patterns

To be read if using asynquence.
