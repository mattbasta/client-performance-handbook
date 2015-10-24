# Assets and Payload

## Gzip and Compression


## Images


### Decreasing image file sizes

Most image editing applications include a significant amount of metadata in the images that are produced. This data includes information about the software that created the image, legal information, comments, description, title, etc. In the context of the web, none of this information is useful to a browser and only takes up valuable bytes.

Additionally, most image editing software uses encoding algorithms that are sub-par. Inefficiencies in the algorithms mean that bytes are needlessly wasted as image data is written in a more verbose way than necessary. This is not to say that the images will decrease in quality when re-encoded: the images should be losslessly re-written to a smaller file representation.

There are many tools available that do this for many different file types. The following are a few of the most popular command line tools:

- **pngcrush**: A tool for compressing PNG images and stripping unneeded metadata
- **PNGOUT**: Another tool for compressing PNG images
- **Jpegoptim**: A tool that provides lossless JPEG optimization
- **Gifsicle**: Rewrites GIF files to their smallest possible form

If you use OS X, an open source tool called ImageOptim can be used to automate these tools with a dead-simple GUI.

![ImageOptim compressing images from this very book](images/image_optim.png)

You can download ImageOptim for free from its homepage: http://imageoptim.com/



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


## One Asset to Rule Them All?

One of the most common questions that comes up with regard to assets is whether sites should have a single, large asset (like a master `include.js` file) or to split the assets into multiple smaller JS files.


### Pros and Cons

There are a number of pros and cons for each approach:


#### Single Large Asset

Pros:

- For small to medium-sized assets, this minimizes the number of connections that need to be made. High-latency users will benefit.
- Throughput tends to be higher for connections that send more data. Having one large asset helps to send more data at once.
- It is simpler to manage a single file than multiple files.
- For JavaScript, the `async` attribute can be used safely (albeit with few benefits).

Cons:

- For most sites, sharing content across pages means that assets will be downloaded that are not used or needed. Users that don't visit the other pages will have wasted that bandwidth.
- If one smaller part of the file changes, the whole file needs to be updated. For websites that are deployed frequently, this may mean that users rarely have a warm cache.
- For JavaScript files, the whole file must be loaded in order for its contents to start executing (versus executing smaller chunks of JavaScript as they arrive).


#### Multiple Assets

Pros:

- Caching is improved, as small changes only invalidate the larger bundles that contain them. Individual assets can be grouped with other assets according to how often they change.
- For very large assets, downloading multiple files in parallel will almost always be faster than downloading one very very large file.

Cons:

- Multiple assets use multiple connections, which means unless the site uses domain sharding, it will reach the six connection limit fairly quickly.
- The issue of waste is not completely eliminated. If one page on a site uses all of two files, and another page on the site uses only part of each of those two files, it will either need to load both files (and waste the bandwidth for both parts that it doesn't use) or a third file will need to be created (preventing reuse and requiring the user to re-download information that's already in their cache).
- If multiple JavaScript files are used, it may not be possible to use the `async` attribute on the script tags without implementing a system for ensuring the files are initialized in the proper order.


#### HTTP/2

When using HTTP/2 (or SPDY), the above points become invalid. Since a single connection is used and headers are compressed, all individual files can be requested with very little overhead. Each file can be cached independently, saving significant amounts of bandwidth (all is related to caching are absent).

This may not be ideal, though, since older browsers (and Internet Explorer in most cases) cannot use HTTP/2. It may also not be possible for the application layer to know whether the client is connected via HTTP/2 or not, making it impossible to decide whether to conditionally serve combined assets.


### Number of assets

The number of assets used is crucial. The following table is a general guide for determining how many of each asset should be used for a single page.

| Amount (gzipped)     | Ideal # assets |
|----------------------|----------------|
| 0-500KB              | 1              |
| 500-1024KB           | 2              |
| 1MB-3MB              | 3              |
| 3MB+                 | 4              |

For example, if there is 600KB of JavaScript on a single page, two JavaScript files should be used.

Across a site, however, it may be smarter to combine assets differently to promote cache reuse. For example, consider the following list of assets:

| Page      | Depends On    |
|-----------|---------------|
| Homepage  | jquery.js     |
| Homepage  | forms.js      |
| Homepage  | scroll.js     |
| Login     | jquery.js     |
| Login     | ajax.js       |
| Login     | forms.js      |
| Dashboard | jquery.js     |
| Dashboard | ajax.js       |
| Dashboard | charts.js     |
| Dashboard | fonts.js      |
| Dashboard | scroll.js     |
| Dashboard | urls.js       |

In this scenario, it might make more sense to create two bundles:

| jquery.js | ajax.js   |
| scroll.js | charts.js |
| forms.js  | fonts.js  |
|           | urls.js   |

used in the following way:

| Page      | Bundles    |
|-----------|------------|
| Homepage  | #1         |
| Login     | #1, #2     |
| Dashboard | #1, #2     |

In this case, the login page shares the same JS as the dashboard, even though most of the second bundle is unused. This doesn't matter, though, since virtually all users visiting the login page will eventually visit the dashboard (and their cache will be warm).

Figuring out how to perfectly distribute files and how many bundles of files to create isn't always an exact science. Getting the split "close enough" will go a long way. After a certain amount of effort, the diminishing returns will yield little (if any) benefit.


### Combining Assets

One final technique that can potentially improve performance is to combine multiple disparate assets together. SVGs can be included as inline markup in HTML. For small images that take less time to load than for their connections to be made, it may be worthwhile to convert them to data URIs and include them as part of the site's CSS.

In extreme cases (such as when dealing with users on very poor network connections), it may even be worthwhile to inline assets into the HTML. This may involve including CSS in `<style>` tags, JavaScript in `<script>` tags, etc. This is not generally advisable due to security, potential performance pitfalls (blocking the site's critical path), and other minor reasons, but for very high-latency, low-bandwidth connections it may be necessary.
