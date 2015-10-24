# JavaScript Minification


## Whitespace Removal

One of the most basic--and oldest--techniques for minifying JavaScript is to simply remove whitespace. By implementing a series of clever regular expressions, whitespace can be effectively stripped from a JavaScript file. Douglas Crockford implemented one of the first and most notable minifiers that uses this technique: JSMin[^1]. The code behind this tool is quite short and very fast, though it may not be compatible with new ES6 syntax and does not always provide perfect results.

More powerfully, a JavaScript parser could be used (Esprima[^2] or Acorn[^3], for instance) to generate a parse tree, and another script could be used to re-write the tree as JavaScript (Escodegen[^4] is a popular choice).

By removing whitespace, code simply takes up less space. It also promotes compression, as there is less variability in the types of strings that exist within the file: the amount of whitespace and type of whitespace used does not affect whether the code compresses to separate values within Gzip.

[^1]: http://www.crockford.com/javascript/jsmin.html
[^2]: https://github.com/ariya/esprima
[^3]: https://github.com/marijnh/acorn
[^4]: https://github.com/Constellation/escodegen


## Name Mangling

Name mangling is a technique used to reduce the amount of space used by identifiers. An identifier is any token (or word) in the JavaScript that represents some object. For example, an identifier might be the name of a variable, or the name of an member on an object. Consider the following code:

```js
function foo() {
    var counter = 0;
    return function() {
        counter += 2;
        return counter;
    };
}
```

By name mangling this piece of code, we can rename `counter` to something shorter:

```js
function foo() {
    var e = 0;
    return function() {
        e += 2;
        return e;
    };
}
```

Notice how the functionality of the code has not changed. Name mangling can usually be done for any variable that exists in the scope of a function, or for any function argument.

There are a number of rules that must be followed when name mangling. First, name mangling may not be done for variables in the global scope, or you run the risk of breaking external code that relies on those variables.

```js
MyGlobal = {member: 123};
function foo() {
    // Name mangling `MyGlobal` will break this.
    return window["MyGlobal"];
}
```

Name mangling also cannot be done for object members. Since we don't know where the member will end up being used, it's not safe to mangle its name.

```js
function input(data) {
    data.thisCantBeMangled = true;
    var xhr = new XMLHttpRequest();
    xhr.open('POST', '/ajax', true);
    xhr.send(JSON.stringify(data));
}
```

In the example above, if `thisCantBeMangled` gets name mangled, the server will get an unexpected JSON blob. Object members **cannot** be name mangled automatically without running the risk that this will happen. Note that this can cause object literals to remain very large, and some code (especially code written for Base.js) will not minify well.

Lastly, any good name mangler must respect scoping rules. For example:

```js
function foo() {
    var x = 0;
    function bar(x) {
        // `x` in this scope is different than `x` in the parent scope.
        return Math.sin(x) + 1;
    }
    return function() {
        x += 0.01;
        return bar(x);
    };
}
```

In the code above, the mangler must respect the fact that even though `x` in `foo()` has the same name as `x` in `bar()`, they are two different variables.


## Dead Code Removal

Oftentimes there is some JavaScript code that cannot possibly be reached. It may exist intentionally, or by accident (oxbow code). In either case, it must be provably deletable. An easy example:

```js
function foo() {
    function bar() {
        console.log('I will never run.');
    }
    return Math.sin(Date.now());
}
```

Since `bar` is never referenced, it will never be run. As such, it is safe to remove it. Some more complicated example:

```js
if (someVariable && false) {
    console.log('This code has been disabled.');
}

var x = 3;
if (x > 4 || !x) {
    console.log('This branch will never be triggered.')
}
```

The first `if` statement is straightforward: `&& false` will prevent the statement from being executed, and a simple test during minification can determine this. The second `if` statement is much harder to identify as dead, however. To a human, it is perhaps obvious that `x` satisfies neither condition. To properly determine this computationally, however, we need to keep track of the value of `x` and implement a system which is capable of performing basic evaluation of the JavaScript. This can quickly become an intractable problem as more and more variables come into play. The implementation required for this approach generally provides diminishing returns (most programmers are careful not to leave too much dead code in their applications), and consequently most minifiers do not have exhaustive implementations.


## Constant Folding

Constant folding is the process of taking expressions which have a definite result and evaluating them at minification time. For instance, `60 * 60 * 24 * 7` is the number of seconds in a week. A minifier could easily determine that each of the binary operators can be evaluated to a definite value and replace the expression with `604800`.

Constant folding can be performed for strings, as well: `"first" + "second"` could easily be combined into `"firstsecond"`. Some operators can be combined: `!!!foo` is a commonly written by new JavaScript programmers to negate a value, but it could simply be written as `!foo` since `!!` is effectively a boolean typecast and `!` returns a boolean to begin with!


## Recommended Tools

The simplest minification tool to use (at present) is UglifyJS2[^5]. Uglify is a JavaScript-based tool (running in Node.js) that supports almost all of the optimizations discussed in this chapter and provides "good enough" results in cases where other tools do a better job.

Here's a simple example of using UglifyJS2 to minify a file called `include.js`:

```bash
# Install UglifyJS2
npm install uglify-js

# Perform minification:
#  -m enables name mangling
#  -c enables other optimizations
uglifyjs js/include.js -m -c > js/include.min.js
```

Depending on the input, UglifyJS2 can be quite resource intensive. Minifying very large amounts of JavaScript can take upwards of a few minutes to complete.

Another powerful tool is Google Closure Compiler[^6]. Closure Compiler is written in Java and supports the most optimizations of any minification tool.

Once downloaded, it can be run using the following command:

```bash
java -jar compiler.jar --js_output_file=js/include.min.js js/include.js
```

To turn on advanced optimizations, simply pass the `--compilation_level ADVANCED_OPTIMIZATIONS` flag. Note that for some JavaScript, this may cause problems. Advanced optimizations are not recommended for legacy or otherwise unusual code.

Without advanced optimizations, Closure Compiler is usually equivalent to UglifyJS2 in terms of speed. With advanced optimizations enabled, Closure Compiler can take significantly longer. For many megabytes of JavaScript, Closure Compiler can take a very long time.

Of course, as mentioned at the beginning of the chapter, JSMin is one of the first minification tools for JavaScript. While it does not do very much, it is quite fast. In some cases, it may be more desirable to create only slightly-minified files in a short amount of time. For example, if you need to minify hundreds of megabytes of JavaScript, it may be appropriate to minify the code with JSMin first (to provide a smaller version of the code), then run the code through UglifyJS later when more resources are available.

JSMin, once installed, can be run with the following command:

```bash
jsmin <js/include.js >js/include.min.js
```

Note that the angle brackets are not wrapping `js/include.js` like an HTML tag. Instead, the `<` is signaling the shell to read `js/include.js` into STDIN and `>` signals the shell to pipe the output to `js/include.min.js`.

[^5]: https://github.com/mishoo/UglifyJS2
[^6]: https://developers.google.com/closure/compiler/
