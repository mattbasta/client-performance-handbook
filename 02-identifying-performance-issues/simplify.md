# Simplify!

As mentioned in the introduction, simplifying code is a very important aspect of performance improvements. Besides the technical debt that complex or poorly written code bears, complex code tends to hide performance issues and can cloud the data that you collect about your application.

Consider the following snippet:

```js
// Please don't ever do this.
var containsTruthyValue = !!(
    Object.keys(mapping)
    .filter(mapping.hasOwnProperty.bind(mapping))
    .map(function(key) {return mapping[value];})
    .filter(function(x) {return !!x;})
    .length);
```

The above piece of code is an actual snippet that I reviewed (and rejected). To someone familiar with functional programming, the concept is simple:

- Find all of the members of the `mapping` object
- Remove any element in the list of members that aren't truthy
- Test whether the result has anything left

On paper, the steps sound straightforward, and in a lazy language this would probably be a satisfactory solution. What's actually going on, however, is chaotic. Four arrays are allocated and destroyed. Dozens, if not hundreds of function calls are made. Not to mention, this code does not short-circuit after it finds its answer.

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

Not only is this code smaller and more straightforward, it's also faster. Keeping code simple and obvious is infinitely more valuable, even if it means the final product is less sleek or pretty.

External monitoring is important to any performance-critical application, but code quality issues like the one shown above are equally important. A poorly written library can add hundreds of milliseconds of delay to page load time and make it harder to fix other unrelated performance issues later.


### Mitigation Techniques

The basics are the most important things you can put in place to prevent bad code from affecting performance:

- Linting allows you to block bad code based on pre-defined patterns. Linting also helps catch mistakes and common coding errors.
- Code reviews are critical to catching bad code. If the author of a block of code is the only engineer that looks it over, mistakes and bad practices will never be caught.
- Unit and integration testing proves that code works. Having well-tested code exposes problems before they escape into the wild. Having good tests is another way of measuring performance: slow tests can be a sign of slow code.

Regular explorations of code helps to keep code fresh. Neglected code tends to bit-rot as it loses mindshare. Invite engineers to regularly look through old or neglected code to see whether there's improvements that can be made, even if those ideas end up on a backlog.

Make time for cleaning up tech debt. Slow or inefficient code often gets little attention, especially if it already works. Making sure resources are available for cleanup ensures bad code doesn't stay around for any longer than it needs to.

Tests can be made more valuable as a performance tool by instrumenting each test. If test times increase substantially, it's easy to know what commit caused the regression. Setting test timeouts can also help to prevent code from being merged to version control.
