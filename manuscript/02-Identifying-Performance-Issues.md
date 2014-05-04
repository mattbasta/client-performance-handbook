# Identifying Performance Issues

The first step to solving any web performance issue is to identify it. In many cases, this can be the hardest part of the increasing site performance: simply knowing that your site is slow in some cases gives you few clues as to what might be wrong or what should be changed.

To begin, as mentioned in the previous chapter, you must start collecting performance data about your site. If you're using Google Analytics already, this is a great first step, but you should plan to collect additional, more granular information to help find out what may need tuning.

The following sections outline resources that are at your disposal that can be used in both a development and production environment identify and diagnose performance issues in the browser.


## The Navigation Timing API

The single best source of information (besides the browser's developer tools) for performance-related information is the navigation timing API. This is available in all modern browsers except (at the time of writing) Safari. This API is a JavaScript object that contains the timestamps that `DOMContentLoaded` and the window `load` event start and stop, as well as other timestamps related to the browser's request and the server's response.

```js
var perfData = window.performance.timing;

// Total page load time
var totalLoadTime = perfData.loadEventEnd - perfData.navigationStart;

// Time until the page was ready
var readyTime = perfData.domContentLoadedEventEnd - perfData.navigationStart;

// Time taken on the client after the server finished returning the HTML
var clientTime = perfData.loadEventEnd - perfData.responseEnd;

// Time required to perform a DNS lookup for the page's hostname
var dnsTime = perfData.domainLookupEnd - perfData.domainLookupStart;

var sslHandshakeTime;
// Time required to perform the SSL handshake
if (perfData.secureConnectionStart) {
    sslHandshakeTime = perfData.connectEnd - perfData.secureConnectionStart;
}

alert('It took ' + ((perfData.loadEventEnd - perfData.navigationStart) / 1000) + ' seconds to load.');
```

During development, it's handy to keep a series of small scripts in bookmarklets or GitHub gists that can be pasted into the developer tools console. I keep one, for example, to output time to `DOMContentLoaded` and time to the window `load` event and refresh the page (very useful for collecting information for a benchmark).

The following members are available in the `window.performance.timing` object:

navigationStart
: The time that the client first started loading the page.

unloadEventStart
: If `unload` handlers were registered, this is the time that the client started firing the `unload` event on the previous page. If no `unload` event handler is registered on the previous page, this will be zero. Note that this may happen concurrently with the start of the next page's load (i.e.: the client may begin fetching the URL and following redirects as the `unload` event fires).

unloadEventEnd
: If `unloadEventStart` is not zero, this is the time that the client finished firing the `unload` event on the previous page.

redirectStart
: If the client encounters a redirect while loading the page, this will contain the start timestamp of the first request that was redirected. If there were no redirects during the page load process, this will contain zero.

redirectEnd
: If `redirectStart` is not zero, this will contain the timestamp of the last byte received from a redirect request.

fetchStart
: The timestamp that the client begins processing the request that loads the page. Note that this timestamp is marked before any connection or cache lookup is made.

domainLookupStart
: The timestamp that the client started performing a DNS lookup for the hostname of the page being loaded. If the DNS lookup was cached, this will be the same as `fetchStart`.

domainLookupEnd
: The timestamp that the client completed a DNS lookup. If no lookup was performed, this is the same as `fetchStart`.

connectStart
: The timestamp immediately before the client creates a TCP connection to the remote server that hosts the page. If the page is cached or an existing connection to the remote server is used, this contains the same value as `domainLookupEnd`. If the client needs to retry the connection for some reason (e.g.: there's a failure and the connection closes), this represents the start of only the last connection made to the server.

connectEnd
: The timestamp immediately after the client has finished establishing a connection to the server. If the page was cached or no new connection was made, this contains the same value as `domainLookupEnd`.

secureConnectionStart
: This member may not exist if the client does not support it (it will be `undefined`). It contains the timestamp that a secure connection was established but before the client performed an SSL handshake. If the page was not requested over HTTPS, this contains zero. This value (when non-zero) will always fall between `connectStart` and `connectEnd`.

requestStart
: The timestamp that the browser began a request to the remote server. This is the point at which the browser first started to send request headers. If the request needed to be restarted, it will contain the timestamp of the last attempt to send the headers.

  Note that no `requestEnd` exists: this is intentional, as computing this time can be expensive and it doesn't necessarily correspond with a time that the server receives the request.

  If the page is cached, this is the timestamp that the cache is queried.

responseStart
: The timestamp that the server receives the first byte of the response. If the page is cached, this is the timestamp that the cache responds with content.

responseEnd
: The timestamp that the server receives the last byte of the response. If the page is cached, this is the timestamp that the cache finishes sending content.

domLoading
: The timestamp at which the browser sets `document.readyState` to `"loading"`. This is the time that the browser begins constructing the DOM.

domInteractive
: The timestamp at which the browser sets `document.readyState` to `"interactive"`. This is the time that the browser has finished parsing all of the markup on the page but hasn't finished loading all of the assets.

domContentLoadedEventStart
: The timestamp immediately before the browser begins firing the `DOMContentLoaded` event.

domContentLoadedEventEnd
: The timestamp immediately after the browser finishes firing the `DOMContentLoaded` event.

domComplete
: The timestamp at which the browser sets `document.readyState` to `"complete"`. At this point, all resources required to load the page have finished processing.

loadEventStart
: The timestamp immediately before the browser begins firing the `load` event on the `window` object.

loadEventEnd
: The timestamp immediately after the browser finishes firing the `load` event on the `window` object.


Google Analytics will collect some basic information for you, including the total load time, DNS lookup time, time taken to create a connection, time between request and the beginning of the server's response, time until `fetchStart`, `domInteractive`, and `domContentLoadedEventStart`. [^ga_perf_timing] These different timings are exposed under Behavior > Site Speed > Page Timings. On that page, you can select the various metrics from the dropdown under DOM Timings and Technical.

[^ga_perf_timing]: This is based on an inspection of http://www.google-analytics.com/ga.js

Google Analytics only collects eight precomputed values, though, and each of the individual values of the navigation timing object are not collected or exposed. It's trivial to collect this information yourself, though there are libraries (such as Boomerang[^boomerang_github]) that will collect this information for you. Ideally, you would post this information via `XMLHttpRequest` or `navigator.sendBeacon` and aggregate it in a solution like Hive or Splunk (any data analytics solution that you're comfortable with will do).

[^boomerang_github]: https://github.com/yahoo/boomerang


## Timing Data Inside JavaScript

One of the critical components to page load is JavaScript execution and initialization. When developing, it's easy to use your browser's developer tools to profile code and identify operations that introduce delays. For example, you might notice something like the following:

![An expensive style and layout recalculation in the Chrome Developer Tools Timeline](images/timeline_recalculation.png)

The above shows that somewhere during the execution of the JavaScript on the page, something happened that caused a style recalculation to occur (taking 11ms) and a layout recalculation followed (taking another 5ms). Without the developer tools, though, collecting this information requires special consideration. Most JavaScript is compiled into a single file (rather than many small files), meaning that even when the developer tools *are* available, the information that's provided doesn't necessarily indicate what exactly was the root cause of the delay.

The simplest way to combat this is to add timing markers during the minification process (minification is described in further detail in later chapters). During the minification process, many scripts are concatenated before being run through a tool to decrease the size of the resulting code. It might look something like this:

```js
var fs = require('fs');

var filesToMinify = glob('assets/');

fs.writeFile(
    'output.js',
    filesToMinify.map(function(filePath) {
        return fs.readFileSync(filePath).toString();
    }).join('\n')
);

```

To add timing markers, the above would be updated to look something like the following:

```js
var fs = require('fs');

var filesToMinify = glob('assets/');

var timingMarker = (
    ';' +
    'Timing = window.Timing || {};' +
    'Timing.files = Timing.files || {};' +
    'Timing.markers = Timing.markers || {};'
);

fs.writeFile(
    'output.js',
    timingMarker +
    'Timing.files["output.js"] = Date.now();' +
    'Timing.markers["output.js"] = [];' +
    filesToMinify.map(function(filePath) {
        var marker = 'Timing.markers["output.js"].push({name: ' + JSON.stringify(filePath) + ', start: Date.now()});';
        var source = fs.readFileSync(filePath).toString();
        return marker + source;
    }).join('\n')
);

```

When `output.js` (or any other JS file built in this manner) is included on the page, it creates a very handy object that looks something like this:

```js
Timing = {
    files: {
        // All of your JS files will appear here
        "output.js": 1399228214354,
        "languages.js": 1399228215002
    },
    markers: {
        // Each of the JS files compiled into the minified JS files will appear here
        "output.js": [
            {name: "assets/js/utils.js", start: 1399228214354},
            {name: "assets/js/dom.js", start: 1399228214354},
            {name: "assets/js/ajax.js", start: 1399228214355},
            {name: "assets/js/events.js", start: 1399228214355}
        ],
        "languages.js": [
            {name: "assets/lang/en-US.js", start: 1399228215002}
        ]
    }
};
```

From this data, you can do some interesting things. For instance, the following code--when used as a bookmarklet--generates a nice chart in your console:

```js

Object.keys(Timing.files).forEach(function(file) {
    console.log(file + Array(100 - file.length).join('-'));
    var markers = Timing.markers[file];
    markers.forEach(function(marker, i) {
        console.log(
            marker.name +
            Array(100 - marker.name.length).join(' ') +
            (i !== markers.length - 1 ? markers[i + 1].start - marker.start : 0)
        );
    });
});

```

![The output of running the above script](images/timing_marker_bookmarklet.png)

Each line in the diagram shows how long each script took to execute.

There are plenty of other ways to generate this kind of timing information. In applications that wrap "modules" in functions (such as modules written under the AMD pattern), it would be easy to write a simple wrapper around `define` to generating timing markers for each module.

Using this data is more challenging. Especially for large applications with many hundreds (or thousands) of kilobytes of code, this solution can produce a lot of information. Sending all of this back to the server can be a burden, and processing it can be even more of a problem.

Instead, you can pare down the data on the client side. Many files will take less than a few milliseconds to execute; the data for those can simply be thrown out. Reporting data for scripts that take longer than 10ms is a great starting point. The reporting can be further refined to set thresholds for files that are known to cause long pauses (perhaps they inject HTML or add lots of complex styles).


## Storing Timestamps at Important Page Events

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
// Always make sure the Timing object exists before writing to it.
Timing = window.Timing || {};
Timing.events = Timing.events || {};
// Save a timestamp for the the "headFlush" event.
Timing.events.headFlush = Date.now();
</script>

<?php

// Flush the content of the page so far to allow the browser to start loading
// the JavaScript and CSS.
flush();

require('database.php');
DB::init();

$user = DB::getUser();

require('templates.php');
Templates::render('homepage.tmpl', [
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

Templates::render('footer.tmpl');

?>
</body>
</html>
```

Having this information is helpful because it allows you to identify when parts of the page are loading relative to your JavaScript, even if those pieces don't load in concert with the times listed in the navigation timing API.


## Make More Graphs

## Simplify
