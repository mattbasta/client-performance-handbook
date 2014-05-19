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




## JS
