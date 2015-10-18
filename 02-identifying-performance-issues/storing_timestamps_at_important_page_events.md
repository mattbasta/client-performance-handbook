# Storing Timestamps at Important Page Events

By "events," I don't mean DOM events. Instead, I mean events that are considered important in the normal page load process that aren't already reported elsewhere.

For example, in *Even Faster Web Sites*, Steve Souders recommends performing an early header flush that sends the `<head>` tag and the beginning of the page's body. This allows the browser to begin downloading assets listed at the start of the document while the server performs the rest of the processing for the page in parallel. In this scenario (and scenarios where more than one flush is performed), it's extremely helpful to know when the various flushes are taking place and how long it takes for the client to perform the necessary actions involved.

The following example shows how you might log this information. It uses PHP, though this should be possible in any language.

```php
<!DOCTYPE html>
<html>
<head>
  <title>My great page</title>
  <link rel="stylesheet" href="/static/include.css">
  <script src="/static/incude.js" defer async></script>
</head>
<body>

<script>
// Always make sure the Timing object exists before
// writing to it.
Timing = window.Timing || {};
Timing.events = Timing.events || {};
// Save a timestamp for the the "headFlush"
// event.
Timing.events.headFlush = Date.now();
</script>

<?php

// Flush the content of the page so far to allow
// the browser to start loading the JavaScript
// and CSS.
flush();

// Perform some operations that are going to take
// a while.
App\DB::init();

$user = App\DB::getUser();

App\Templates::render('homepage.tmpl', [
    'user' => $user,
]);

?>
<script>
// Save a timestamp for the "bodyFlush" event.
Timing.events.bodyFlush = Date.now();
</script>
<?php

// Flush again for the body
flush();

App\Templates::render('footer.tmpl');

?>
</body>
</html>
```

Having this information is helpful because it allows you to identify when parts of the page are loading relative to your JavaScript, even if those pieces don't load in concert with the times listed in the navigation timing API.

If you're using a CSP (Content Security Policy) with your page, using inline script tags may be difficult or impossible. Instead, you can use the server time (in the PHP example above, the appropriate function is `microtime()`) and output the timestamps in data attributes on DOM elements. Your scripts can then extract them from the DOM with `document.querySelector()` or your favorite JavaScript selector library.
