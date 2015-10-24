# Assets and Payload

## Gzip and Compression


## Images


### Post-loading Images

In many cases, images on a page are not necessary to make the page interactive to the user. In these cases, blocking page load may be harmful: loading indicators may be displayed to the user, JavaScript may be blocked, or other assets may be delayed while the images download (possibly due to the number of concurrent HTTP connections).

This is a simple technique for requesting images only after a page has loaded:

```html
<div class="postload-image" data-src="/images/my-great-image.jpg"
     title="My great image">
  <noscript>
    <img src="/images/my-great-image.jpg" alt="My great image">
  </noscript>
</div>
```

The following JavaScript should be included:

```js
window.addEventListener('load', function() {
    // This `setTimeout` prevents us from blocking the end of the load event
    // in some browsers.
    setTimeout(function() {
        // Get a list of all of the postloaded images on the page
        var postloadedImages = document.querySelectorAll('.postload-image');
        var img;
        var imgSrc;
        for (var i = 0; i < postloadedImages.length; i++) {
            img = postloadedImages[i];
            imgSrc = img.getAttribute('data-src');
            // Set the background of each image to its URL
            img.style.backgroundImage = 'url(' + imgSrc + ')';
        }
    }, 0);
});
```

Adding support for pre-defined height and width of the images is left as an exercise to the reader.


### Spacer GIFs: Not even once

Many years ago, when CSS was young and tables were an accepted way of laying out pages, it was common practice to use spacer GIFs to add vertical space between elements and horizontal space between text.

Today, spacer GIFs are completely and utterly unnecessary. CSS--even in its most basic form--makes the original usefulness of spacer GIFs redundant at best. These GIFs require time to decode and position, as well as the time required to make the connection to download them.

I've seen some folks attempt to use transparent spacer GIFs as data URIs in stylesheets as a way of placating the rendering bugs of old versions of IE. This is shameful and should never be done.


### You don't need that image

One of the best ways to decrease the number of images used on a page is to replace the images with graphics generated using CSS and inline SVG. For instance, small linear gradients were often represented using some code like this:

```css
.element-with-a-horiz-gradient {
    background: url(/images/grad-horiz.png) repeat-x;
}
```

While effective, this technique creates an extra request for every unique gradient used on the page. Additionally, gradient images like this (that are repeated horizontally) are usually difficult or even impossible in some cases to sprite.

A better approach would have been to use CSS gradients instead:

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
- Inline SVG can be used to create very basic shapes. It is wise to only use inline SVG for graphics that are not repeated or reused.

The biggest and perhaps only downside to using generated graphics rather than external images is the rendering performance cost. Radial gradients, for instance, are extremely expensive to render. Large numbers of small linear gradients, multiple layered linear gradients, multiple `box-shadow`s, and others can add sizable rendering pauses to your site, and make scroll performance slow to a crawl.

Always test your site before and after adding these kinds of graphics to ensure performance is within an acceptable range. Also consider the trade-off of eliminating an HTTP request versus the cost of rendering an additional gradient or shadow.
