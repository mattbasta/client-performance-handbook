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
