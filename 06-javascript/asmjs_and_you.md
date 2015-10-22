# Asm.js and You

Asm.js has recently taken the JavaScript community by storm as a means of extracting extra performance from plain, vanilla JavaScript. In some cases, this can be true: compilers can target asm.js instead of machine code to produce extremely efficient applications. Unfortunately, making use of these improvements for most developers will be challenging or impossible.


### The use case

Asm.js was developed by Mozilla to enable highly performant applications to be run on the web. In doing so, it also allows existing C++ applications to compile to JavaScript. This means that games like DOOM or software like CPython can be compiled to JavaScript and run in the browser with no installation necessary.

What asm.js is *not* however, is a general purpose tool for every-day application development. Asm.js is a subset of JavaScript: in order to use it, you can only take advantage of a very small subset of the features that JavaScript has to offer:

- Functions, but no closures
- Numeric types only; no strings, arrays, objects, etc.
- Static arrays of functions

Additionally, Asm.js requires that most expressions use type hints. These type hints are binary and unary operators that signal the types of the data that they are operating on. For instance, consider the following function:

```js
function add(a, b) {
    return a + b;
}
```

In asm.js, the same function might look like this:

```js
function add(a, b) {
    // Type hints for arguments:
    a = a | 0;
    b = b | 0;
    // Type hint for return type:
    return a + b | 0;
}
```

The `| 0` signals to the compiler that the value that it operates on should be treated as an `int`. If you wanted to use this function with floating point values (`double`s, this example), you'd need to write something like the following:

```js
function add(a, b) {
    // Type hints for arguments:
    a = +a;
    b = +b;
    // Type hint for return type:
    return +(a + b);
}
```

In doing this, the compiler can know ahead of time the exact type of every value that will pass in and out of the `add()` function. The code that is produced is extremely fast. Benchmarks put asm.js-optimized code at roughly 1.25x the cost of "native". That is, asm.js code runs about 25% slower than code compiled directly to machine code. While this obviously isn't perfect, it is certainly sufficient for almost every resource-heavy task.


### Costs of Asm.js

Despite the incredible performance wins, there are costs to using asm.js. While the performance of asm.js applications in general may be high, asm.js code is not a silver bullet for performance.


#### Size

First and foremost, asm.js code is significantly larger than non-asm.js code. This happens for a variety of reasons:

- Type annotations take up a significant amount of space
- Many constructs in JavaScript like closures are not available in asm.js, meaning that code is necessary to manage state
- Memory management code needs to be implemented by the asm.js script. That is, to allocate and deallocate memory, a `malloc` and `free` implementation need to be made available in the script.
- Aside from the `Math` object, there is essentially no standard library for asm.js. All data structures, string support, and other constructs need to be implemented manually.
- Tools that compile code to asm.js generally output very verbose code that is difficult to minify.
- Minification is difficult because the type annotations must not be changed. This makes options like UglifyJS's `-c` flag useless.

A popular tool for generating asm.js code is Emscripten[^emscripten]. Emscripten takes LLVM bitcode and converts it to JavaScript. The following is a Hello World application taken from the Emscripten test suite:

[^emscripten]: https://github.com/kripken/emscripten

```cpp
#include<stdio.h>

int main() {
  printf("hello, world!\n");
  return 0;
}
```

Compiling this script with Emscripten 1.22.0 generates a 248KB JavaScript file. Minifying this yields a 141KB file, and the result gzips to roughly 39KB. Though the final product is indeed much smaller than the original generated source, this is an outrageous amount of code needed solely to `console.log` the value `"hello, world!\n"`.

Much of the generated code is designed to support web workers, memory allocation, and supporting the necessary components of `stdio.h`. Larger programs will not substantially increase the size of the generated JavaScript file unless they include more dependencies or increase the complexity significantly.

This does, however, show the cost of using asm.js for trivial tasks. If you use tools to generate asm.js code, the size of the output may cost more to download, parse, and compile than the application will save from faster execution times.


#### Communication Overhead

As mentioned previously, asm.js code can only work with numeric data. That is, to implement other data structures you must do so using the simplest of primitives. To represent a string, for instance, you must break it down to an array of unsigned integer values (the equivalent of character sequences in C or C++). Moving this data in and out of asm.js can be very costly, though. Consider the following:

```js
// Returns whether `input` contains the character `X`.
function containsX(input) {
    return input.indexOf('X') !== -1;
}

console.log(containsX('fooXbar'));  // true
```

This is obviously a very simple function. A string is passed, and the function returns whether the string contains the uppercase letter "X".

```js
var asmModule = (function(module) {
    // Create a heap
    var heap = new ArrayBuffer(0x1000);
    // Return access to the module and the heap
    return {
        methods: module(window, {}, heap),
        heap: heap
    };
}(function mod(stdlib, foreign, heap) {
    "use asm";

    // Create an ArrayBufferView that will let us read the data
    // back out of the heap.
    var intheap = new stdlib.Uint8Array(heap);

    // The charCode value of "X"
    var Xval = 88;

    // A function that returns 1 for strings that contain "X"
    // and 0 for strings that do not. `index` is the position
    // of the string in the heap.
    function containsX(index) {
        index = index | 0;
        var i = 0;
        var iter = 0;
        iter = intheap[index | 0] | 0;
        while ((iter | 0) > 0) {
            if ((intheap[index + iter + 1 | 0] | 0) == (Xval | 0)) {
                return 1;
            }
            iter = (iter - 1) | 0;
        }
        return 0;
    }

    // Return access to our containsX function
    return {containsX: containsX};
}));
```

At this point, we've created an object named `asmModule` that contains the boilerplate necessary to initialize an asm.js module. This isolates the asm.js code from normal JavaScript.

You'll notice that the `containsX` function accepts an index parameter rather than a string, as its plain old-fashioned counterpart did. This is intentional: as mentioned previously, asm.js can't work with non-numeric data. That means that strings can't be passed directly to asm.js functions.

To get around this limitation, we access the `ArrayBuffer` "heap" that's available at `asmModule.heap` and assign the string data into it. In this example, we'll simply write the string data into the `ArrayBuffer` starting at position zero, but a more useful asm.js module would need to expose memory management methods like `malloc` and `free` to reserve chunks of the heap without conflicting with the internals of the asm.js module itself.

```
var uintarr = new Uint8Array(asmModule.heap);

function saveStringToArr(arr, input, index) {
    // Index 0 is the string's length
    arr[index] = input.length;
    // Assign the charCode values to their respective positions
    // in the array buffer
    for (var i = 0; i < input.length; i++) {
        arr[index + i + 1] = input.charCodeAt(i);
    }
}

// We're putting the string at index 0
var index = 0;

// Save the string to the array buffer
saveStringToArr(uintarr, "hello X world", index);

// Test whether the string contains an "X"
console.log(!!asmModule.methods.containsX(index));
```

This example illustrates the point I'm trying to make using strings, but the example works the same for more or less complex data structures, like objects or booleans.

At a low string length (roughly two dozen characters), we find that in Firefox, the asm.js version performs marginally better than the vanilla JavaScript version.[^asm_overhead_perf] Longer strings (just under 4KB) perform outrageously worse.

[^asm_overhead_perf]: http://jsperf.com/asm-js-comm-overhead-test

![Comparison of asm.js vs vanilla JS](images/asm_overhead_comparison.png)

What is actually happening here? The performance issue is not the result of asm.js being slow. Rather, it's the result of the `saveStringToArr()` function performing poorly. The physical process of moving data from a JavaScript string into an `ArrayBuffer` costs quite a lot of CPU cycles. You'll notice that even though Chrome's performance on the "fast" asm.js test is rather low, it still suffers from the overhead of copying the data for the "slow" asm.js test. Conversely, Chrome performs equally well with a long and short string while using the `indexOf` method.

Let's consider another use case: binary data processing. Imagine a function that looks like the following:

```
function compressPNGImage(arrayBuffer) {
  // Implementation is left as an exercise to the reader
  // ...
}
```

Such a function might accept an `ArrayBuffer` containing the binary content of a PNG file (perhaps provided through a browser's File API) and modify the `ArrayBuffer` in-place. In this case, an asm.js implementation will almost certainly always out-perform a vanilla JavaScript version, as the ahead-of-time compilation that Firefox will perform will greatly reduce the overhead of the JIT compiler. The input, in this case, can be used directly by asm.js (where the `ArrayBuffer` is passed as the "heap"), meaning the problems experienced in the previous example simply don't exist.

The lesson to be gleaned here is that there are many cases where asm.js may provide good performance benefits, but the approach required to take advantage of these benefits may in some cases negate them entirely. In practical terms, using asm.js for one-off data processing functions throughout a project will likely not perform nearly as well as expected. Instead, asm.js must usually be used from start to end for data processing or on binary data in order to fully realize the performance wins.


### Browser Considerations

One potential downside of using asm.js is that its benefit is most profound in Firefox only. Firefox uses a dedicated compiler called OdinMonkey to analyze and convert asm.js code directly to machine code (without needing to run it, as with a standard JIT compiler).

Other browsers do not special-case asm.js. Chrome's V8 team, for example, chose not to implement the `"use asm";` pragma that asm.js code requires. Instead, V8 chose to optimize the JIT compiler to better understand and refine the types of patterns used in asm.js[^chrome_asm]. This approach benefits all JavaScript, not just asm.js.

[^chrome_asm]: https://code.google.com/p/v8/issues/detail?id=2599

Despite this, Firefox's asm.js implementation largely remains the most performant implementation at runtime (as can be seen by the benchmark in the previous section). CPU-heavy applications will almost certainly perform better in Firefox's JavaScript engine than in Chrome's (or any other browser's). If non-Firefox performance is quite bad, it may mean that your users will be forced to use Firefox instead of their browser of choice, which can be a bad user experience.
