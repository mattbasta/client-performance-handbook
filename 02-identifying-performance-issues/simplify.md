# Simplify!

As mentioned in the introduction, simplifying code is a very important aspects of performance improvements. Besides the technical debt that complex or poorly written code bears, complex code tends to hide performance issues and can cloud the data that you collect about your application.

Consider the following snippet:

```js
// Please don't ever do this.
var containsTruthyValue = !!(Object.keys(mapping)
    .filter(mapping.hasOwnProperty.bind(mapping))
    .map(function(key) {return mapping[value];})
    .filter(function(x) {return !!x;})
    .length);
```

The above piece of code is an actual snippet that I reviewed (and rejected). To someone familiar with functional programming, the concept is simple:

- Find all of the members of the `mapping` object
- Remove any element in the list of members that aren't truthy
- Test whether the result has anything left

On paper, the steps sound straightforward, and in a lazy language this would probably be a satisfactory solution. What's actually going on, however, is chaotic. Four arrays are allocated and destroyed. Dozens, if not hundreds of function calls are made. Rather than finding one positive result and short circuiting, the code performs its manipulations on every single member of the original object and then throws all that data away. What an unholy waste.

An equivalent snippet of code simply would have been:

```js
var containsTruthyValue = false;
for (var key in mapping) {
    // Wow, for loops aren't so bad.
    if (mapping.hasOwnProperty(key) && mapping[key]) {
        containsTruthyValue = true;
        // I may be homely, but I'm fast as heck
        break;
    }
}
```

Not only is this code smaller (40 bytes less when minified with uglify.js) and more straightforward, it's also faster. Keeping code simple and obvious is infinitely more valuable, even if it means the final product is less sleek or pretty.

Furthermore, it's the case that if code is written in a way that's non-standard or confusing, it's more difficult for a compiler (or minifier) to safely optimize it. The wins that compilers could provide that are lost to codebases that have been prematurely optimized are so great that we cannot begin to imagine them.
