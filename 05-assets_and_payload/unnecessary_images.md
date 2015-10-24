# Unnecessary Images

One of the best ways to decrease the number of images used on a page is to replace the images with graphics generated using CSS and inline SVG. For instance, small linear gradients were often represented using some code like this:

```css
.element-with-a-horiz-gradient {
    background: url(/images/grad-horiz.png) repeat-x;
}
```

While effective, this technique creates an extra request for every unique gradient used on the page. Gradient images like this (i.e., images that repeat horizontally) are difficult or even impossible in some cases to add to a sprite.

A better approach would be to use CSS gradients instead:

```css
.element-with-a-horiz-gradient {
    /* A fallback for old browsers that don't support gradients. */
    background: #eaeaea;
    /* The gradient */
    background: linear-gradient(to bottom, #ddd 0%, #fff 100%);
}
```

Note that all modern browsers support CSS gradients.

Some other images can be replaced with generated code:

- Small triangles can be replaced with elements (or pseudoelements) with borders set up to form a triangle.
- Certain effects that would have been stored as expensive animated GIFs can be implemented using CSS transformations and CSS animations.
- CSS declarations like `border-radius` and `box-shadow` can be used to create complex and intricate graphics.
- Inline SVG can be used to create very basic shapes. It is wise to only use inline SVG for graphics that are not repeated or reused, as inline SVG needs to redownload on every page load.

The biggest and perhaps only downside to using generated graphics rather than external images is the rendering performance cost. Radial gradients, for instance, are expensive to render. Large numbers of small linear gradients, multiple layered linear gradients, and multiple `box-shadow`s can add rendering pauses to your site. This can lead to the infamous "scroll jank."

Always test your site before and after adding these kinds of graphics to ensure performance is within an acceptable range. Also consider the trade-off of eliminating an HTTP request versus the cost of rendering an additional gradient or shadow.


## Spacer GIFs: Not even once

When CSS was young and tables were acceptable ways of laying out pages, it was common practice to use "spacer GIFs" to add space between elements and text. Spacer GIFs were so useful that Netscape briefly added support for a `<spacer>` tag[^1]. Today, spacer GIFs are completely and utterly unnecessary. CSS--even in its most basic form--makes the original usefulness of spacer GIFs redundant at best. These GIFs require time to request, download, decode, render, and paint onto the page. Performant pages and spacer GIFs are mutually exclusive.

I've seen some folks attempt to use transparent spacer GIFs as data URIs in stylesheets as a way of placating the rendering bugs of old versions of IE. This is shameful and you should never do this.

[^1]: https://developer.mozilla.org/en-US/docs/Web/HTML/Element/spacer
