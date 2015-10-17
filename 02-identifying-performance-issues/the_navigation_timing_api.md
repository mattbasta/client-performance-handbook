# The Navigation Timing API


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

alert('Page loaded in ' + (totalLoadTime / 1000) + 's');
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
