# Premature Optimization

Simplifying code and eliminating tech debt often coincides with premature optimization. Too often, engineers "optimize" code with the goal of improving performance without understanding the results.

One of my biggest pet peeves is the following code pattern:

```js
var arr = returnsSomeArray();
var sum = 0;
for (var i = 0, len = arr.length; i < len; ++i) {
    sum += arr[i];
}
```

Somewhere along the way, someone realized that caching the value of an array's `length` property saves some time. And in fact, caching `length` provides a 15% speedup (at the time of writing) in Google Chrome[^1]. Some books and online resources have taken to showing snippets of code like this and touting them as easy improvements to code.

[^1]: Results taken from http://jsperf.com/array-length-vs-cached, tested in Chrome 46.

If I had to take an educated guess, I'd say that the overwhelming majority of arrays used in JavaScript across all pages have fewer than 50 elements. If that's the case, let's time the code above using the `performance.now()` browser API, which gives sub-millisecond accuracy:

```js
// Tested in Chrome 46
var arr = returnsSomeArray(); // 50 elements
var sum, i;
var start, end;

// The easy way
start = performance.now();
sum = 0;
for (i = 0; i < arr.length; ++i) {
    sum += arr[i];
}
end = performance.now();
console.log(end - start); // ~0.075ms


// The "better" way
sum = 0;
start = performance.now();
for (i = 0, len = arr.length; i < len; ++i) {
    sum += arr[i];
}
end = performance.now();
console.log(end - start); // ~0.045ms
```

The output here is startlingly small: the loop took less than a tenth of a millisecond to run each of the two loops. For the sake of the example, I didn't use a tool to warm up the JIT (see the chapter on JavaScript for more on that). If the code ran for longer, the JavaScript compiler probably could have optimized the lookup away as a "loop invariant," rendering the entire exercise useless.

Given the number of `for` loops in a piece of code, it may be the case that the amount of time saved by caching `length` is far less than the time spent downloading the extra bytes to cache the value of `length`. Depending on how the code is used, you could also argue that the three hundred microseconds saved would never add up to the time it physically took to type the "optimization" out.

There is a lesson to be learned here. Premature optimization is indeed a powerful form of tech debt. Making code less readable, even a little bit, is almost universally a bad practice. It's nearly impossible to quantify the benefit of making changes to things on the web that take less than a fraction of a millisecond to happen.

As a courtesy to the reader, every form of premature optimization will not be enumerated in this book. The anecdote above about caching `length` in loops is a single type. Any code patterns that claim to improve performance ("code smells") are oftentimes snake oil. Deeply consider the impact of the what you and your fellow engineers write from the standpoint of maintenance and iteration. Remember, as said in previous chapters, client performance is almost never about individual instances of "slow" code. Rather, poor performance is usually the result of larger compound issues or the interaction of multiple systems.