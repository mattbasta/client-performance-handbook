# Minification

## Markup

Markup is an often-neglected aspect of minification. The HTML that is sent to the browser is the source of information for which assets should be loaded. If the user is on a bandwidth-constrained connection, lots of excessive whitespace, unused elements, and garbage nodes can have a very negative effect on page load time.


### Whitespace Removal

For most websites, whitespace removal is trivial. In HTML, two adjacent whitespace characters are treated as one, so removing any duplicate whitespace characters is a safe operation.

This isn't true for certain elements, though. The `<pre>` and `<textarea>` tags are sensitive to whitespace, meaning whitespace should not be removed from these elements. Additionally, elements can have the `white-space: pre` CSS property, which makes them behave the same way. Pages with many inline `<script>` tags may also run into issues if the JavaScript inside the tags omits semicolons or contains strings with whitespace that would otherwise be trimmed.

For these pages that contain whitespace-sensitive elements, it is usually simpler to disable site-wide whitespace removal than to try to trim around the affected elements. Relatively few pages on a site tend to contain elements like this, so the performance regression should be relatively minor.

Most templating languages allow provide syntax for stripping whitespace. This is often quite useful and allows you to manually avoid whitespace-sensitive elements. For static markup files, tools like HTMLTidy[^htmltidy] are available. HTMLTidy--though a bit dated--provides a simple way to "minify" markup files.

[^htmltidy]: http://tidy.sourceforge.net/

In some cases, stripping whitespace is simply good code hygiene for a project: some Java and PHP frameworks, for instance, will spuriously output tiny amounts of whitespace at the beginning and end of every response. This can quickly cause a mess and make viewing the source of a page a real chore.


#### Compression and Whitespace Removal

An interesting question that can be asked is how well Gzip compression works as a means of negating the costs of whitespace in HTML output, or whether stripping whitespace manually is beneficial. One argument is that after compression, the benefits of removing whitespace (decreased size, mostly) are moot. In theory, Gzip compression should simply compress whitespace away entirely.

Let's have a look:

```bash
curl https://github.com > /tmp/github.html
cat /tmp/github.html | wc
#     329    1064   16463
```

The homepage for Github weighs in at roughly 16kb of markup. Let's see how much of that is whitespace:

```bash
# `tr` deletes characters
cat /tmp/github.html | tr -d "\n\r\t " | wc
#       0       1   13683
```

`16kb - 13kb` tells us that the homepage contains about `3kb` worth of whitespace. That's about 18% of the output. An 18% savings could be fairly substantial, but of course not all whitespace can be safely removed (spaces between words, spaces between inline HTML elements, etc.). If we safely remove whitespace, we get a different result:

```bash
# The first `sed` command strips leading whitespace
# The second `sed` command deletes blank lines
cat /tmp/github.html | sed "s/^[ \t]*//" | sed "/^$/d" | wc
#      268    1064   14757
```

A more complex example than the one used above could be crafted, but for the purposes of this example, the two `sed` commands remove all whitespace that can be safely removed.

The result of this operation shows that the output is now approximately 14kb, showing that roughly only 2kb of whitespace can be removed. Let's now compare the gzipped versions of these two outputs:

```bash
cat /tmp/github.html | gzip | wc
#      15     143    4636
cat /tmp/github.html | sed "s/^[ \t]*//" | sed "/^$/d" | gzip | wc
#      16     117    4440
```

After compressing the output, less than 200 bytes are saved! In measuring the packet sizes of the connection to `github.com`, removing 200 bytes would not decrease the number of TCP packets sent to the client. Consequently, there is absolutely no performance savings from the decrease in size, and a potential net loss in performance if the whitespace is stripped from the output at the time of the request.

Another argument against whitespace removal is CPU cost. Whitespace removal using simple regular expressions is relatively cheap and can be optimized quite well. HTML elements like `<pre>` can make this significantly more complicated, though, and CSS rules like `white-space: pre` can make it impossible to perform this test without partially rendering the page. In these cases, whitespace trimming can be extremely CPU intensive and take more time to complete than any conceivable payload savings would yield.

Certainly, these conclusions are not true 100% of the time. Stripping whitespace at compilation time (when your web application is compiled to a binary, like with golang, or when your template syntax is compiled to your runtime language, like with Smarty, Jinja2, or Handlebars) is extremely efficient and can provide substantial benefits. Whitespace that is removed at compile time costs nothing at runtime, meaning there is no downside to using it. Additionally, the downsides of whitespace being inappropriately stripped from elements styled like `<pre>` tags is negated because stripping must be explicitly triggered.

Consider the following Smarty template:

```smarty
{strip}
    <ul>
        <li>This has a lot of whitespace!</li>
        <li>Lots of indentation.</li>
        <li>Line breaks everywhere.</li>
    </ul>
{/strip}
```

When rendered for the first time, Smarty's template compiler will simply ignore any duplicate whitespace characters:

```html
<ul><li>This has a lot of whitespace!</li><li>Lots of indentation.</li><li>Line breaks everywhere.</li></ul>
```


### Keeping Markup Minimal

When adding new features, it tends to be easy to add more and more markup to an interface. Markup bloat, however, does more than just increase the size of the HTML. The size of the DOM in memory can also play a role in speed. Parsing the markup, allocating the nodes for it, and calculating the layout for the document will all affect performance for very heavy pages.

The amount of markup that you need is fairly subjective, however. Use extra tags sparingly in page elements that repeat. Always attempt to use the fewest elements possible for any particular feature.


### SVG

SVG is an XML-based format, so most of the normal markup optimizations apply to it. However, many further optimizations are possible that are specific to the way SVGs are implemented.

SVGs tend to include lots of extra data. This includes information like the name of the program that generated the file, settings used when creating the image, the title of the image, etc. The following nodes are safe to remove from most SVGs:

- **`DOCTYPE`s**: The `DOCTYPE` node can take up a significant amount of space and is rarely (if ever) used by clients. Simply remove it.
- **`<title>` and `<description>` Nodes**: These nodes do nothing for embedded SVGs and simply take up space.
- **`<metadata>` Nodes**: These include what their name implies: metadata. It is not used for any client-side rendering purposes.
- **Empty `<defs>` Nodes**
- **RDF**: Some `<svg>` elements will an `xmlns:rdf="http://www.w3.org/1999/02/22-rdf-syntax-ns#"` attribute. This signals that RDF is used in the file. RDF is not used to render the SVG, and the `xmlns` and `<rdf:foo>` nodes can be removed.
- **Unused `id=""` Attributes**: Most IDs that are embedded in SVGs are not used for anything. If you aren't referencing an ID from CSS or using it for spriting purposes, the IDs are safe to remove.
- **`xlink` Attributes**: For simple images that are not used outside of the page they're embedded on, all traces of `xlink` can be removed.
- **Unnecessary `<g>` elements**: Many vector graphics applications will include `<g>` elements to group nodes as they're grouped within the application. While these can be (and often are) used to apply layout and visual transformations and effects, they are often generated regardless of whether or not they do anything to the image.

As mentioned in the section on SVGs, the precision of floating point numbers used in SVGs can be reduced to save space. In general, three digits of precision is usually sufficient for most purposes. Some applications will include up to eight digits: reducing this will significantly decrease the post-compression size of the file.

Another minor optimization for SVGs is to eliminate unnecessary whitespace within paths. For example, the following two SVG paths are identical:

```xml
<path d="M10 10 H 90 V 90 H 10 L 10 10" />
<path d="M10 10H90V90H10L10 10" />
```

Notice how whitespace around letters is not necessary. Each command is only one character, but numbers can be multiple characters. Whitespace is only necessary to eliminate ambiguity.

Certain commands within a path can also be eliminated. For instance, a `c` command followed by another `c` command could be combined into a single "polybezier."


## CSS

There are two types of optimizations for CSS: the first makes the stylesheet smaller by removing code that is not useful (or rewriting it to be smaller). The second type of optimization rewrites other rules to be more consistent to make Gzip more effective.

These optimizations can be made in one of two ways:

1. They can be made using a simple set of regular expressions to isolate, change, and remove parts of a stylesheet. This approach does only a fair job, but tends to be more resilient to malformed code and old "CSS hacks."
2. They can also be made by using a proper lexer and parser that deconstructs the stylesheet into an object, performs optimizations just as a compiler would optimize an AST tree, and reconstructs the CSS in its smallest possible representation. While this approach can perform much more complex optimizations, it tends to fail when the tool encounters CSS that it does not recognize or does not know how to parse.

Virtually all CSS minification tools take the first approach. Some notable ones include YUI Compressor, clean-css, slimmer, and minify. For most sites, anything more than this is excessive, though sites with problematically large CSS files can opt for the second approach to produce even more compact stylesheets.

With regard to minifiers that properly parse the CSS, there are only a handful that exist: CSSO, crass[^crass_disclaimer], CSSTidy, and closure-stylesheets. These tend to produce much smaller output, but can fail quite spectacularly if presented with unrecognized input.

[^crass_disclaimer]: Full disclosure: crass is a project developed by the author


### Smaller Code

There are a number of basic optimizations that you can perform for CSS without very much effort. The one that provides the most gains is whitespace removal. CSS contains a great deal of whitespace and very little of it affects the document. Any good minifier will remove all unnecessary whitespace.

Other basic code removal optimizations include the following:

- Remove the semicolon after the last declaration in a rule set: `div {color:red;}` to `div {color:red}`
- Converting colors to their shortest possible forms (`#XXYYZZ` to `#XYZ`, `rgb(0,0,0)` to `#000`, `blanchedalmond` to `#ffebcd`, `rgba(255,255,255,.1)` to `hsla(0,0,100%,.1)`, etc.).
- Remove duplicate declarations (`{color:red;color:red;}` to `{color:red}`). This seems like an unnecessary optimization, but it removes a surprising amount of code. Note that in some contrived cases, this optimization can actually *increase* the final gzipped size.
- Remove duplicate selectors in a selector list (`.foo, .bar, .foo` to `.foo`). Duplicate selectors usually appear as a result of the use of CSS preprocessors.
- Remove duplicate classes in a selector (`.bar.bar` to `.bar`, or `.bar:first-child:first-child` to `.bar:first-child`).
- Remove empty rule sets (e.g.: `.foo {}`).
- Collapse lists of values (`margin: 0 0 0 0` to `margin: 0`). Note that this only applies to certain declarations, like `margin`, `border-width`, `padding`, etc.
- Delete mismatched vendor prefixes. In some cases, preprocessors can generate code like the following: `@-webkit-keyframes{from,to{-moz-transform:rotate(0)}}` (usually with a corresponding normal `@keyframes` and `-webkit-transform`, as well). Because these vendor prefixes do not match, they will never be applied.
- Delete invalid vendor prefixes. Oftentimes, CSS generators and preprocessors will add all possible vendor prefixes to CSS3 features (`@keyframes`, `border-radius`, `linear-gradient`, etc.). It is usually the case, though, that some of those vendor prefixes were never in use by browsers to begin with. E.g.: `@-ms-keyframes` is often generated, but CSS animations were never prefixed in Internet Explorer.
- In some cases, `none` can be replaced with `0`.
- Font weights can be replaced with their numeric equivalents to save a few bytes.
- The universal selector (`*`) can be removed from larger selectors. E.g.: `*.foo` to `.foo`
- In cases where specificity does not matter, selector lists containing the universal selector can be reduced to just the universal selector. E.g.: `*, .foo, .bar` to `*`
- Combine identical media queries: `@media screen and (min-width:1px),screen and (min-width:1px)` to `@media screen and (min-width:1px)`
- Convert `from` keyframes to `0%` and `100%` keyframes to `to`.
- Combine adjacent rule sets with identical selectors: `.foo {color:red} .foo{border:0}` to `.foo{color:red;border:0}`
- Combine adjacent rule sets with identical bodies: `.foo {color:red} .bar {color:red}` to `.foo,.bar {color:red}`
- Remove declarations from overridden rules sets: `div{color:red;border:0} a{color:blue} div{color:green}` to `div{border:0}a{color:blue}div{color:green}`. Note that this optimization must always merge into the "right" rule set, as the middle rule set may apply to the same element. Note also that if the right rule set's selector is a selector list, the optimization can still be performed. The optimization can even be performed if (only) the left rule set is inside an `@` block.
- Remove old declarations and old vendor prefixes. `-moz-border-radius` has been unprefixed since Firefox 4, which was released in 2011. Similarly, `-webkit-border-radius` was unprefixed in Chrome 5, which was released in 2010. `-webkit-gradient` was replaced in Chrome 10 (2011). All modern browsers support unprefixed gradients.
- Two rule sets with identical selectors that are not adjacent may be combined into the right rule set if and only if all rule sets in between have a lower specificity than the rule sets being combined and the left rule set does not have any `!important` flags that are also set in the middle rule sets. This can also be done if none of the declarations in the left rule set are found in any of the middle rule sets.


### Improving Compression

Compression of CSS can be improved by increasing Gzip's ability to remove duplicate strings from the CSS. By making your CSS more consistent, the final compressed size of the code can be reduced without decreasing the size of the code.

- Make declaration names, element names, colors, pseudo elements and classes, attribute names, and other identifiers lowercase. Consistent capitalization ensures that repeated versions of the same string use the same representation.
- Sort selector lists (`.a, .c, .b` to `.a, .b, .c`)
- Sort declarations `{a:1; c:3; b:2}` to `{a:1; b:2; c:3}`).

When sorting, the sort order (alphabetically, dashes first or at the end, using a custom comparison function, etc.) doesn't matter, so long as the sort is deterministic. For instance, if you prefer to group `position`, `left`, `right`, `top`, and `bottom`, that's fine, so long as--when sorted--those declarations always appear in the same location relative to their neighbors and the same order.


### CSS Minification Myths

When I'm asked about CSS minification, I oftentimes hear questions like "Can't you just combine all of the similar rule sets?" or "Isn't it safe to simply combine all of the rule sets in the stylesheet with identical selectors?"

Usually, the answer is "No." Most CSS optimizations are simply too unsafe to use in practice. Consider the following:

```html
<div class="class anotherClass">How is this text styled?</div>
```

```css
.class {
    color: red;
}
.anotherClass {
    color: green;
    font-weight: normal;
}
.class {
    font-weight: bold;
}
```

The `.class` rule sets--despite appearing to be safe to combine--will have a different behavior after the combination. As the code above is written, the text in the `<div>` will be green and bold. If the `.class` rule sets are combined into the first of the two, the text will no longer be bold. If they are combined into the second of the two, the text will be red and bold.

Because there are multiple ways to reference the same element, it's possible that any two non-adjacent pieces of CSS will be overridden by any of the code in-between. The only exception to this rule is--as stated in the previous sections--100% of the code between the two rule sets has a lower specificity (in which case it will never override either of the pieces of code).

There are some basic guidelines that can help you to identify unsafe optimizations:

- If the optimization reorders rule sets that do not have identical or similar selectors.
- If the optimization removes declarations that can apply under even obscure circumstances.
- It is unsafe to combine any two rule sets that do not have identical selectors or identical bodies.
- Optimizations involving changing the contents of rule sets that have a selector list are usually unsafe, unless all other rule sets involved have identical selector lists.
- Optimizations that move rule sets containing declarations with the `!important` flag above or below other rule sets with the flag on the same declaration are unsafe.
- Removing any `@` block is generally unsafe unless it can be proven that it will never apply or that its contents cannot change any styles (e.g.: mismatched vendor prefixes).
- Moving or combining `@` blocks above or below other `@` blocks is usually unsafe.

Another common idea is to test which CSS rules are applied to a particular page any only output the rules which match the page's content (i.e.: never serving CSS which is overridden or unused). There are a number of projects which can perform this type of optimization, such as mincss by Peter Bengtsson[^mincss]. You feed HTML and CSS into the tool and you're given a "cleaned" version of your stylesheet.

[^mincss]: https://mincss.readthedocs.org/

This approach can save a lot of bandwidth, especially for sites with highly customizable designs. Unfortunately, it usually means all of the CSS on the page must be included inline in `<style>` tags (and consequently, all of the CSS is never cached and must be downloaded on every page load), or the server needs to keep track of what HTML was sent to the browser so that it can serve a customized stylesheet. Note that this has additional drawbacks:

- Content that is dynamically generated on the client will not receive styles, since the server did not see the dynamic content and filtered out the styles for it.
- The server must parse each HTML response (consuming memory to store the DOM), plus parse and process the stylesheet(s) on every request (assuming the response is not static). The resources required to perform these operations is non-trivial at best. For common web platforms like PHP and Python, the memory required to store the HTML and CSS representations is far larger and the CPU costs far higher than that of a compiled language--like C or C++--that's used in the client.
- Content inserted with SSI (Server Side Includes), ESI (Edge Side Includes), application middleware, Nginx plugins, and other post-processing code will not be seen by the script and will consequently not be styled.

Some of this can be avoided with caching, but ultimately the benefits of applying this technique will provide limited and diminishing returns. For average web applications, the decrease in CSS payload is not substantial enough to significantly improve performance (though of course there are exceptions).


## JS

### Whitespace Removal

One of the most basic--and oldest--techniques for minifying JavaScript is to simply remove whitespace. By implementing a series of clever regular expressions, whitespace can be effectively stripped from a JavaScript file. Douglas Crockford implemented one of the first and most notable minifiers that uses this technique: JSMin[^jsmin]. The code behind this tool is quite short and very fast, though it may not be compatible with new ES6 syntax and does not always provide perfect results.

[^jsmin]: http://www.crockford.com/javascript/jsmin.html

More powerfully, a JavaScript parser could be used (Esprima[^esprima] or Acorn[^acorn], for instance) to generate a parse tree, and another script could be used to re-write the tree as JavaScript (Escodegen[^escodegen] is a popular choice).

[^esprima]: https://github.com/ariya/esprima
[^acorn]: https://github.com/marijnh/acorn
[^escodegen]: https://github.com/Constellation/escodegen

By removing whitespace, code simply takes up less space. It also promotes compression, as there is less variability in the types of strings that exist within the file: the amount of whitespace and type of whitespace used does not affect whether the code compresses to separate values within Gzip.


### Name Mangling

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


### Dead Code Removal

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


### Constant Folding

Constant folding is the process of taking expressions which have a definite result and evaluating them at minification time. For instance, `60 * 60 * 24 * 7` is the number of seconds in a week. A minifier could easily determine that each of the binary operators can be evaluated to a definite value and replace the expression with `604800`.

Constant folding can be performed for strings, as well: `"first" + "second"` could easily be combined into `"firstsecond"`. Some operators can be combined: `!!!foo` is a commonly written by new JavaScript programmers to negate a value, but it could simply be written as `!foo` since `!!` is effectively a boolean typecast and `!` returns a boolean to begin with!


### Recommended Tools

The simplest minification tool to use (at present) is UglifyJS2[^uglifyjs2]. Uglify is a JavaScript-based tool (running in Node.js) that supports almost all of the optimizations discussed in this chapter and provides "good enough" results in cases where other tools do a better job.

[^uglifyjs2]: https://github.com/mishoo/UglifyJS2

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

Another powerful tool is Google Closure Compiler[^closure_compiler]. Closure Compiler is written in Java and supports the most optimizations of any minification tool.

[^closure_compiler]: https://developers.google.com/closure/compiler/

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
