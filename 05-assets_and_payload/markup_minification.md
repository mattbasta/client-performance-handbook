# Markup Minification

Markup is an often-neglected aspect of minification. The HTML that is sent to the browser is the source of information for which assets should be loaded. If the user is on a bandwidth-constrained connection, lots of excessive whitespace, unused elements, and garbage nodes can have a very negative effect on page load time.


## Whitespace Removal

For most websites, whitespace removal is trivial. In HTML, two adjacent whitespace characters are treated as one, so removing any duplicate whitespace characters is a safe operation.

This isn't true for certain elements, though. The `<pre>` and `<textarea>` tags are sensitive to whitespace, meaning whitespace should not be removed from these elements. Additionally, elements can have the `white-space: pre` CSS property, which makes them behave the same way. Pages with many inline `<script>` tags may also run into issues if the JavaScript inside the tags omits semicolons or contains strings with whitespace that would otherwise be trimmed.

For these pages that contain whitespace-sensitive elements, it is usually simpler to disable site-wide whitespace removal than to try to trim around the affected elements. Relatively few pages on a site tend to contain elements like this, so the performance regression should be relatively minor.

Most templating languages allow provide syntax for stripping whitespace. This is often quite useful and allows you to manually avoid whitespace-sensitive elements. For static markup files, tools like HTMLTidy[^htmltidy] are available. HTMLTidy--though a bit dated--provides a simple way to "minify" markup files.

[^htmltidy]: http://tidy.sourceforge.net/

In some cases, stripping whitespace is simply good code hygiene for a project: some Java and PHP frameworks, for instance, will spuriously output tiny amounts of whitespace at the beginning and end of every response. This can quickly cause a mess and make viewing the source of a page a real chore.


### Compression and Whitespace Removal

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


## Keeping Markup Minimal

When adding new features, it tends to be easy to add more and more markup to an interface. Markup bloat, however, does more than just increase the size of the HTML. The size of the DOM in memory can also play a role in speed. Parsing the markup, allocating the nodes for it, and calculating the layout for the document will all affect performance for very heavy pages.

The amount of markup that you need is fairly subjective, however. Use extra tags sparingly in page elements that repeat. Always attempt to use the fewest elements possible for any particular feature.


## SVG

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
