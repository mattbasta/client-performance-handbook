# Minification

In the book *High Performance Web Sites* (Chapter 10), Steve Souders differentiates between minification and obfuscation. Today, the two concepts have largely merged into one: all proper minification tools tend to perform many complex optimizations.

Ten years ago, tools like JSMin and YUI's CSS compressor were state-of-the-art. Today, the tools that have replaced them are more akin to compilers. UglifyJS and Google Closure Compiler have taken JavaScript optimization to a whole new level and made deep, complex minification techniques both safe and reliable as well as easy-to-use for the average web developer.


## Markup

Markup is an often-neglected aspect of minification. The HTML that is sent to the browser is the source of information for which assets should be loaded. If the user is on a bandwidth-constrained connection, lots of excessive whitespace, unused elements, and garbage nodes can have a very negative effect on page load time.


### Whitespace Removal

For most websites, whitespace removal is trivial. In HTML, two adjacent whitespace characters are treated as one, so removing any duplicate whitespace characters is a safe operation.

This isn't true for certain elements, though. The `<pre>` and `<textarea>` tags are sensitive to whitespace, meaning whitespace should not be removed from these elements. Additionally, elements can have the `white-space: pre` CSS property, which makes them behave the same way. Pages with many inline `<script>` tags may also run into issues if the JavaScript inside the tags omits semicolons or contains strings with whitespace that would otherwise be trimmed.

For these pages that contain whitespace-sensitive elements, it is usually simpler to disable whitespace removal than to try to trim around the affected elements. Relatively few pages on a site tend to contain elements like this, so the performance regression should be relatively minor.

In some cases, stripping whitespace is simply good code hygiene: some Java and PHP frameworks, for instance, will spuriously output tiny amounts of whitespace at the beginning and end of every response. This can quickly cause a mess and make viewing the source of a page a real chore.

Whitespace removal is fairly simple when using tools: Apache has a mod_trim plugin that automates this process, though the mod_pagespeed plugin from Google will perform whitespace removal in addition to many other optimizations. Django has a `StripWhiteSpaceMiddleware` class that can be flipped on. Smarty templates include a `{strip}` block that will simply ignore whitespace to begin with in the templates. Plugins are available for nearly every platform, framework, and server software with varying extra features.

Stripping whitespace is something of a micro-optimization, though. After Gzip, the difference between the original document and a whitespace-free document are minimal at best. Some measurements have put the size difference below 1KB on average. This happens because whitespace is often repeated in the same way throughout a file.

Note that whitespace in client-side templates does not suffer from this downside. When compiled to JavaScript, whitespace can account for a significant amount of space thanks to template features that split strings up in unique ways. Removing whitespace from client-side templates is definitely recommended.


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
- Remove declarations from overridden rules sets: `div{color:red;border:0} a{color:blue} div{color:green}` to `div{border:0}a{color:blue}div{color:green}`. Note that this optimization must always merge into the "right" rule set, as the middle rule set may apply to the same element. Note also that if the right rule set's selector is a selector list, the optimization can still be performed. Additionally, if the rule set(s) in the middle have a lower specificity than the selector for the left and right rule sets and the left and right rule sets have identical selectors, the two can be merged into the right rule set (rather than just removing the overridden rules).
- Remove old declarations and old vendor prefixes. `-moz-border-radius` has been unprefixed since Firefox 4, which was released in 2011. Similarly, `-webkit-border-radius` was unprefixed in Chrome 5, which was released in 2010. `-webkit-gradient` was replaced in Chrome 10 (2011). All modern browsers support unprefixed gradients.


### Improving Compression

Compression of CSS can be improved by increasing Gzip's ability to remove duplicate strings from the CSS. By making your CSS more consistent, the final compressed size of the code can be reduced without decreasing the size of the code.

- Make declaration names, element names, colors, pseudo elements and classes, attribute names, and other identifiers lowercase. Consistent capitalization ensures that repeated versions of the same string use the same representation.
- Sort selector lists (`.a, .c, .b` to `.a, .b, .c`)
- Sort declarations `{a:1; c:3; b:2}` to `{a:1; b:2; c:3}`).

When sorting, the sort order (alphabetically, dashes first or at the end, using a custom comparison function, etc.) doesn't matter, so long as the sort is deterministic. For instance, if you prefer to group `position`, `left`, `right`, `top`, and `bottom`, that's fine, so long as--when sorted--those declarations always appear in the same location relative to their neighbors and the same order.


## JS
