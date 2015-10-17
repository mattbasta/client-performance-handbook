# Premature Optimization

While simplifying code and eliminating tech debt is very important, it often coincides with premature optimization. Too often, engineers write "optimized" code with the intention of preventing a component from being slow without understanding the benefits that their efforts are yielding.

One of my biggest pet peeves is when I see code like the following:

```js
var sum = 0;
for (var i = 0, len = arr.length; i < len; ++i) {
    sum += arr[i];
}
```

Somewhere along the way, someone realized that caching the value of an array's `length` property saves some time. And in fact, caching `length` provides a 20% speedup (at the time of writing) in Google Chrome[^array_len_cached].

[^array_len_cached]: Results taken from http://jsperf.com/array-length-vs-cached, tested in Chrome 35.

Nobody cares. 20% of a fraction of a millisecond is still a fraction of a millisecond. The overhead from the loop will amount to--if you add all of the invocations of that code up from all of time--probably only a few seconds. Total.

The following code is identical in every way, except it's smaller and easier to read:

```js
var sum = 0;
for (var i = 0; i < arr.length; ++i) {
    sum += arr[i];
}
```

Given the number of `for` loops in a piece of code, it might even be the case that the amount of time saved by caching `length` is far outweighed by the amount of time spent downloading the extra code to cache the value of `length`.

The lesson to be learned here is that spending time optimizing code would be better spent analyzing the application as a whole and addressing lower-hanging fruit.