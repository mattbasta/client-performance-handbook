# Postloading

Postloading is the process of deferring the download of certain page elements until after the page has already finished loading. For example, some page elements may not be visible by default. In this case, it may be beneficial to allow the page to become interactive and download the elements in the background.

Using this technique can provide substantial benefits if more than a few of your assets are not used on the initial page load, or if some of those assets are far "below the fold".

These benefits may also apply to your back-end, as well. Video files are often large and consume a lot of bandwidth. A video positioned below the fold on a page may needlessly use bandwidth if visitors never scroll down. Delaying the video's download until the user begins scrolling can decrease bandwidth usage and server load.


## Post-loading Images

In some cases, images on a page are not necessary to make the page interactive to the user. Blocking page load may be harmful: loading indicators may display, JavaScript may block, or other assets may get delayed.

This is a simple technique for requesting images after a page has loaded. Replace your `<img>` tags in the following manner to begin:

```html
<!-- Before -->
<img src="/images/my-great-image.jpg" alt="My great image">

<!-- After -->
<div class="postload-image"
     data-src="/images/my-great-image.jpg"
     title="My great image">
  <noscript>
    <img src="/images/my-great-image.jpg" alt="My great image">
  </noscript>
</div>
```

The following JavaScript makes this work:

```js
window.addEventListener('load', function() {
    // This `setTimeout` prevents us from blocking the end of the load event.
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

Note that any sizing or positioning on your `<img>` tag should be updated to apply to the corresponding `.postload-image` element.


## Drawbacks

Abusing postloading can lead to bad user experiences. Postloading too much content can make the page sluggish. Postloading scripts or CSS can make some features take longer than a user might expect to become ready to use. If you postload a large third-party library, for instance, code that uses that library might display a loading indicator until the postload code finishes.

When postloading (or un-postloading, for that matter), be sure to test the user's experience on a variety of browsers, devices, and connections.
