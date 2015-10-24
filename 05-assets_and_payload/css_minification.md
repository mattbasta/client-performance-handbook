# CSS Minification

There are two types of optimizations for CSS: the first makes the stylesheet smaller by removing code that is not useful (or rewriting it to be smaller). The second type of optimization rewrites other rules to be more consistent to make Gzip more effective.

These optimizations can be made in one of two ways:

1. They can be made using a simple set of regular expressions to isolate, change, and remove parts of a stylesheet. This approach does only a fair job, but tends to be more resilient to malformed code and old "CSS hacks."
2. They can also be made by using a proper lexer and parser that deconstructs the stylesheet into an object, performs optimizations just as a compiler would optimize an AST tree, and reconstructs the CSS in its smallest possible representation. While this approach can perform much more complex optimizations, it tends to fail when the tool encounters CSS that it does not recognize or does not know how to parse.

Virtually all CSS minification tools take the first approach. Some notable ones include YUI Compressor, clean-css, slimmer, and minify. For most sites, anything more than this is excessive, though sites with problematically large CSS files can opt for the second approach to produce even more compact stylesheets.

With regard to minifiers that properly parse the CSS, there are only a handful that exist: CSSO, crass[^crass_disclaimer], CSSTidy, and closure-stylesheets. These tend to produce much smaller output, but can fail quite spectacularly if presented with unrecognized input.

[^crass_disclaimer]: Full disclosure: crass is a project developed by the author


## Smaller Code

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


## Improving Compression

Compression of CSS can be improved by increasing Gzip's ability to remove duplicate strings from the CSS. By making your CSS more consistent, the final compressed size of the code can be reduced without decreasing the size of the code.

- Make declaration names, element names, colors, pseudo elements and classes, attribute names, and other identifiers lowercase. Consistent capitalization ensures that repeated versions of the same string use the same representation.
- Sort selector lists (`.a, .c, .b` to `.a, .b, .c`)
- Sort declarations `{a:1; c:3; b:2}` to `{a:1; b:2; c:3}`).

When sorting, the sort order (alphabetically, dashes first or at the end, using a custom comparison function, etc.) doesn't matter, so long as the sort is deterministic. For instance, if you prefer to group `position`, `left`, `right`, `top`, and `bottom`, that's fine, so long as--when sorted--those declarations always appear in the same location relative to their neighbors and the same order.


## CSS Minification Myths

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
