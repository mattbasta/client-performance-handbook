# JavaScript

## Improving CPU-Heavy Code

Sometimes, performance issues with a particular piece of JavaScript code can simply be traced back to inefficiencies with the code that runs under the hood of the JavaScript engine. There is a lot of talk about the leaps and bounds made by JIT compilation in JavaScript, though there is little talk about what works and what doesn't with regard to this process.

At a high level, JIT compilation works in the following way:

1. Code runs in "interpreter mode", where each each instruction is executed manually by another application (the "interpreter").
2. As the code runs in the interpreter, the JIT compiler takes profile information about typing and application flow.
3. The JIT compiler uses the profile information it collects to create an optimized version of the program it is running. This optimized version is usually raw machine code.

Unbeknownst to most developers, there are in fact patterns that will cause some code to run significantly faster than other pieces of code. In small, simple pieces of JavaScript it is rarely the case that such patterns will pose any significant performance issue. Large, complex, or frequently-running pieces of code will indeed surface such issues.

The following are some symptoms of poorly-performing CPU-heavy code:

- Significant pauses in application execution that are not associated with external operations (XHR, image or video decoding, `localStorage` access, etc.) or garbage collection.
- Consistent, frequent stuttering, such as during drag-and-drop or when using scroll-related behavior (e.g., parallax effects, "sticky" headers, etc.).
- Pauses or delays that happen when pressing UI elements that trigger large amounts of JavaScript.

To illustrate how two pieces of similar code can have tremendous performance differences, consider the following two snippets of JavaScript:

```js
var badInput = [
  1, 2, 3, 4, 5, '6', '7', 8, 9, [10]
];

var dump = [];
function process(x) {
  dump.push(x * 10);
}
for (var i = 0; i < 1000; i++) {
  process(badInput[i % badInput.length]);
}
```

```js
var goodInput = [
  1, 2, 3, 4, 5, 6, 7, 8, 9, 10
];

var dump = [];
function process(x) {
  dump.push(x * 10);
}
for (var i = 0; i < 1000; i++) {
  process(goodInput[i % goodInput.length]);
}
```

![Comparison of the two pieces of code](images/type_perf_comparison.png)

Notice how the version with consistent array element types is significantly faster[^jsperf_jit_breakage]. The only difference between the two snippets is the input used. The "bad input" test uses a mix of numbers, strings, and an object which needs to be cast to a string, then a number. The second snippet uses numbers exclusively.

[^jsperf_jit_breakage]: http://jsperf.com/jit-breakage

Obviously, the performance difference is striking, despite producing identical output. There are a few reasons for this:

- Many CPU cycles are wasted converting strings and objects to numbers.
- The `process()` function is passed data with inconsistent types. When the JIT compiler runs for the first ("bad input") test, it has to add checks to `process()` to see what type `x` is and handle that appropriately. In the second ("good input") test, the JIT compiler only needs to add code for numbers.

It may seem improbable that someone would write code that looks like the first snippet. However, consider a function that performs some action (say, updating the score in a game). Sometimes, the function is called by an action that the player takes, and an integer is passed. Other times, the function is called by an action that the computer player takes, and a string is passed (perhaps it comes from an `XMLHttpRequest`'s `responseText`).

Let's have a look at another example:

```js
// Setup code:
var dump = [];
var names = ['Matt', 'Tom', 'Lucy', 'Sally'];
function Person(name, id, age) {
  this.name = name;
  this.id = id;
  this.age = age;
  this.previous = null;
}

var i;
var person;


// -- Example 1 --

for (i = 0; i < 100; i++) {
  // Create a new Person object
  person = new Person(
    names[i % names.length],
    i,
    20 + i % 10
  );
  dump.push(person);
  // Compute a value and set it to a member of the object just created.
  if (i > 0) {
    person['previous'] = dump[i - 1];
  }
}

dump = [];

// -- Example 2 --

for (i = 0; i < 100; i++) {
  // Create a new Person object, as before.
  person = new Person(
    names[i % names.length],
    i,
    20 + i % 10
  );
  dump.push(person);
  // Take a fixed value and set it to a member of the object that's computed.
  person['age' + person.age.toString()] = 'ripe old age';
}

```

Which of the two versions of the code above will be more performant? The difference is very simple:

- The first example computes `i - 1`, performs a lookup to `dump`, and assigns the value it fetched to `person.previous`.
- The second example performs a lookup for `person.age`, computes `'age' + person.age.toString()`, and assigns `'ripe old age'` to the member that it computed.

Had this code run strictly in an interpreter, the difference between the two is negligible. The JIT compiler, however, is capable of making the first example significantly faster:

![Comparison of second two pieces of code](images/type_change_perf_comparison.png)

The first example is far faster[^jsperf_jit_breakage_objects], but why? The answer lies in how the JIT compiler is able to optimize objects. When you create any sort of object in JavaScript, it has a "shape." The shape of an object is all of the different properties and methods that the object has assigned to it. This information is used to create the equivalent of a C++ class representing the object. For instance, the `Person` object in the last example might have a shape that looks like the following:

[^jsperf_jit_breakage_objects]: http://jsperf.com/jit-breakage-objects

```c++
class Person {
    public:
        string name;
        int id;
        int age;
        Person *previous;
};
```

This is, of course, an over-simplified explanation of what is going on, but it illustrates the concept.

In the second half of the example above, a new property is being computed for the object:

```js
person['age' + person.age.toString()] = 'ripe old age';
```

When this happens, the object's shape changes. No longer can the object use the `Person` shape from the C++ example above. Instead, one of two things happens:

1. A new shape needs to be created.
2. The JIT needs to stop trying to optimize the code and fall back to the interpreter.

In the first case, another representation will end up being created:

```c++
class Person__with_age7 { // New name to indicate the difference
    public:
        string name;
        int id;
        int age;
        Person *previous;

        int age7; // New!
};
```

Creating this new shape is expensive, however. Additionally, `Person` and `Person__with_age7` may not be able to fit into the same array anymore:

```c++
// Because all of the objects are not `Person` anymore, the JIT compiler
// may not be able to put them in an optimized array:
Person* dump[100];
// Instead, it may need to create a more generic, slower array:
JSObject* dump[100];
```

The first half of the code, however, can use the optimized representations all the time, since the shape does not change. This means that it can run far faster.


### Improving JIT performance

There are a number of things you can do to improve the performance of your CPU-bound application:

- Always pass the same type of value to arguments in a function. Do not pass a combination of strings and integers, for example, to the same argument.
- Have a consistent return type for your functions. Functions should always return only one type.
- Do not change the shape of your objects:
  - Do not assign arbitrary members to objects.
  - Do not assign values of different types to the same object member.
- Though it is not a problem in Firefox, Chrome is up to twice as fast if you use a constructor (`new Foo()`) instead of an object literal when following all of the above advice.


### Web Workers

On the web, JavaScript is single-threaded. This means that no two pieces of code will ever execute simultaneously (with few, obscure exceptions). Some types of computation, however, are best done on a separate thread. Long-running pieces of code, for instance, should not block the UI from being used.

The solution is Web Workers. Workers create a separate JavaScript execution context that runs on its own thread. Issues with thread safety are removed, as the interface between a worker and a page exists as a single message passing callback, and all data transferred between the two contexts is serialized.

Here's an example of a script that uses a Monte Carlo method to calculate pi:

```js
var startTime = Date.now();

var total = 100000000;
var numWithin = 0;
var x;
var y;
for (var i = 0; i < total; i++) {
    x = Math.random();
    y = Math.random();
    if (x * x + y * y < 1) {
        numWithin++;
    }
}
// <div id="result"></div>
document.querySelector('#result').innerHTML = 4 * numWithin / total;

var endTime = Date.now();
// <div id="time"></div>
document.querySelector('#time').innerHTML = endTime - startTime;
```

When run, the output shows the approximation of pi and the duration of the script in milliseconds:

```
3.14159844
9817
```

Now, this is an unacceptable amount of time to make the page essentially unusable. Instead, let's create a web worker:

```js
// This is stored in worker.js

var total = 100000000;
var numWithin = 0;
var x;
var y;
for (var i = 0; i < total; i++) {
    x = Math.random();
    y = Math.random();
    if (x * x + y * y < 1) {
        numWithin++;
    }
}

postMessage(4 * numWithin / total);
```

And this is the new script for the page:

```js
var startTime = Date.now();
var worker = new Worker('worker.js');
worker.onmessage = function(e) {
    var endTime = Date.now();

    // <div id="result"></div>
    document.querySelector('#result').innerHTML = e.data;
    // <div id="time"></div>
    document.querySelector('#time').innerHTML = endTime - startTime;
};
```

And that's it! The worker will behave exactly as you expect it to: the call to `new Worker()` will load `worker.js` and start running the code. When `postMessage()` is called with the result, `worker.onmessage` is fired. The result is kept in `e.data`. The output of this new script is essentially identical to the output of the previous version.

Using a worker doesn't necessarily decrease the amount of time for the script to run, though; it simply moves the work to another thread. This may increase perceived performance, but it will not make the application work any faster. To take full advantage of web workers, you must break up your work into smaller chunks and spread the work over multiple workers. Let's modify the scripts above slightly:

```js
// This is stored in worker.js

this.onmessage = function(e) {
    work(e.data);
};
postMessage('ready');

function work(total) {
    var numWithin = 0;
    var x;
    var y;
    for (var i = 0; i < total; i++) {
        x = Math.random();
        y = Math.random();
        if (x * x + y * y < 1) {
            numWithin++;
        }
    }

    postMessage(numWithin);
}
```

And this is the new script for the page:

```js
var startTime = Date.now();
var workerPool = [];
var numWorkers = 10;
var numCompletedWorkers = 0;
var numIterations = 100000000;
var numWithin = 0;
for (var i = 0; i < numWorkers; i++) {
    workerPool.push(getNewWorker());
}

function getNewWorker() {
    var worker = new Worker('worker.js');
    var isReady = false;
    worker.onmessage = function(e) {
        if (!isReady && e.data === 'ready') {
            isReady = true;
            // Give the worker something to do.
            worker.postMessage(numIterations / numWorkers);
            return;
        }

        // The worker has completed.
        numWithin += e.data;
        numCompletedWorkers++;
        // If all workers have completed, run the completion function.
        if (numCompletedWorkers === numWorkers) {
            workersCompleted();
        }
    };
}

function workersCompleted() {
    var endTime = Date.now();

    // <div id="result"></div>
    document.querySelector('#result').innerHTML = 4 * numWithin / numIterations;
    // <div id="time"></div>
    document.querySelector('#time').innerHTML = endTime - startTime;
}

```

That's a lot more code, but it allows us to break the work up into multiple, smaller chunks. Each chunk runs in parallel. In this case, the work is broken into ten chunks. Here is the updated result:

```
3.14154388
899
```

That's much faster! Instead of taking almost ten seconds to execute, our code now executes in less than one.

Keeping lots of separate files around can be problematic, though. Additionally, each worker creates its own HTTP request, which can become quite costly if you begin using many workers. The solution is to embed the worker into the same script using `Blob` objects. Here is the second example from above, combined into a single file:

```js
function worker() {
    // Because this function will be converted to a string,
    // it CANNOT access variables outside of itself.

    var total = 100000000;
    var numWithin = 0;
    var x;
    var y;
    for (var i = 0; i < total; i++) {
        x = Math.random();
        y = Math.random();
        if (x * x + y * y < 1) {
            numWithin++;
        }
    }

    self.postMessage(4 * numWithin / total);
}

var startTime = Date.now();

var workerBlob = new Blob(
    // Take the string representation of the function and
    // pass it as a self-executing closure.
    ['(' + worker.toString() + ')()'],
    {type: 'text/javascript'}
);

// Turn the blob containing the worker's source code into a URI
// that the Worker can access and create the Worker.
var worker = new Worker((URL || webkitURL).createObjectURL(workerBlob));
worker.onmessage = function(e) {
    var endTime = Date.now();

    // <div id="result"></div>
    document.querySelector('#result').innerHTML = e.data;
    // <div id="time"></div>
    document.querySelector('#time').innerHTML = endTime - startTime;
};
```

The above script works just as the second example works: `worker()` runs in a separate thread and passes the result back to the page when it has completed. Using this approach for the pooled approach above is left as an exercise to the reader.

Notice that the `worker()` function does not behave as a normal JavaScript function. For instance, `worker()` does NOT have access to the `startTime` variable, despite the appearance that you could otherwise access it via lexical scope. The approach shown above works by taking `worker()` and converting it back to source code with `toString()`. This source code has no connection to the original script; it simply lifts the contents of `worker()` out into the `Blob` object that is passed into the `Worker`. If you are careful, however, parameters can be passed as static strings:

```js
function worker(startTime) {
    // `worker` can now read `startTime`, but cannot change it.
    // ...
}

var startTime = Date.now();

var workerBlob = new Blob(
    // Pass `startTime` as the first argument to `worker`.
    ['(' + worker.toString() + ')(' + JSON.stringify(startTime) + ')'],
    {type: 'text/javascript'}
);

```


### Typed Arrays

In many CPU-intensive routines, one of the most common operations is the manipulation of arrays of numbers. This may be for any number of things: cryptography, 3D rendering, multimedia processing, etc. In all of the above, arrays of integers or floating point numbers must be manipulated. A relatively recent introduction to JavaScript can help optimize these cases: typed arrays.

A typed array is similar to a normal JavaScript array, with a few differences:

1. A typed array can only contain one variable type. For example, a typed array might only contain 32-bit floating point numbers, or 16-bit integers.
2. There are no `null` values in a typed array. All values are initialized to the equivalent of zero by default.
3. Typed arrays are of a fixed length. Once they are instantiated, they cannot be resized.
4. Typed arrays have no literal syntax.

Let's see an example of how you might use a typed array:

```js
var img = new Image();
// Add the data URI for an image here:
img.src = 'data:image/png;base64,...';

var canvas = document.createElement('canvas');
canvas.width = 640; canvas.height = 480;
var ctx = canvas.getContext('2d');
ctx.drawImage(img, 0, 0);
var imgdata = ctx.getImageData(0, 0, 640, 480);

// This function returns the 0-255 hue value of an RGB color.
function getHue(r, g, b) {
    r /= 255, g /= 255, b /= 255;
    var max = r > g ? (r > b ? r : b) : (g > b ? g : b);
    var min = r < g ? (r < b ? r : b) : (g < b ? g : b);

    if (r === g && g === b) return 0;

    var h, d = max - min;
    switch (max) {
        case r: h = (g - b) / d + (g < b ? 6 : 0); break;
        case g: h = (b - r) / d + 2; break;
        case b: h = (r - g) / d + 4; break;
    }
    return h / 6 * 256 | 0;
}

// Count the number of pixels
var pixels = imgdata.width * imgdata.height;
// Create a typed array where each element represents one
// possible hue.
var counts = new Uint32Array(256);

var hue;
// For each pixel, get the hue and increment that hue's
// element in the typed array.
for (var i = 0; i < pixels; i++) {
    hue = getHue(
        imgdata.data[i * 4],
        imgdata.data[i * 4 + 1],
        imgdata.data[i * 4 + 2]
    );
    counts[hue] = counts[hue] + 1;
}

// Find the hue that appears most commonly.
var maxHue = 0, maxVal = 0;
for (var j = 0; j < 256; j++) {
    if (counts[j] > maxVal) {
        maxVal = counts[j];
        maxHue = j;
    }
}

console.log(maxHue);
```

Notice the use of the `Uint32Array` typed array. It accepts a similar set of arguments as the common `Array` constructor. In fact, we could simply replace `Uint32Array` with `Array`, and the output would be identical. This example, however, performs much better when using the typed array:

![Comparison of typed arrays vs. plain arrays](images/typed_array_perf_comparison.png)

The performance of typed arrays is almost double that of plan arrays in Firefox and Chrome[^jsperf_typed_arrays]! This happens for a few reasons:

[^jsperf_typed_arrays]: http://jsperf.com/find-common-color

- With plain arrays, the JavaScript engine does not know in advance what type of data will be stored in the array and cannot optimize it until the code has run many times.
- When the plain array is created, it contains `null` for each of the elements, rather than a numeric type.
- The typed array forces its contents to the hidden `Uint32` type, meaning it defaults to `0` for each value. The JavaScript engine and JIT compiler do not need to guess what type of data the array will hold.

You should attempt to identify appropriate use cases for typed arrays throughout your applications in any place where numeric data is being heavily processed.
