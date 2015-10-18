# The Navigation Timing API

The single best source of information (besides the browser's developer tools) for performance-related information is the navigation timing API. This is available in all modern browsers. This API is a JavaScript object that contains important timestamps, like when `DOMContentLoaded` and the window `load` event start and stop. It also includes information on the times related to interactions with your web server, and even the user's DNS server (if applicable).

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

<dl>
    <dt>navigationStart</dt>
    <dd>The time that the client first started loading the page.</dd>
    <dt>unloadEventStart</dt>
    <dd>If <code>unload</code> handlers were registered, this is the time that the client started firing the <code>unload</code> event on the previous page. If no <code>unload</code> event handler is registered on the previous page, this will be zero. Note that this may happen concurrently with the start of the next page's load (i.e.: the client may begin fetching the URL and following redirects as the <code>unload</code> event fires).</dd>
    <dt>unloadEventEnd</dt>
    <dd>If <code>unloadEventStart</code> is not zero, this is the time that the client finished firing the <code>unload</code> event on the previous page.</dd>
    <dt>redirectStart</dt>
    <dd>If the client encounters a redirect while loading the page, this will contain the start timestamp of the first request that was redirected. If there were no redirects during the page load process, this will contain zero.</dd>
    <dt>redirectEnd</dt>
    <dd>If <code>redirectStart</code> is not zero, this will contain the timestamp of the last byte received from a redirect request.</dd>
    <dt>fetchStart</dt>
    <dd>The timestamp that the client begins processing the request that loads the page. Note that this timestamp is marked before any connection or cache lookup is made.</dd>
    <dt>domainLookupStart</dt>
    <dd>The timestamp that the client started performing a DNS lookup for the hostname of the page being loaded. If the DNS lookup was cached, this will be the same as <code>fetchStart</code>.</dd>
    <dt>domainLookupEnd</dt>
    <dd>The timestamp that the client completed a DNS lookup. If no lookup was performed, this is the same as <code>fetchStart</code>.</dd>
    <dt>connectStart</dt>
    <dd>The timestamp immediately before the client creates a TCP connection to the remote server that hosts the page. If the page is cached or an existing connection to the remote server is used, this contains the same value as <code>domainLookupEnd</code>. If the client needs to retry the connection for some reason (e.g.: there's a failure and the connection closes), this represents the start of only the last connection made to the server.</dd>
    <dt>connectEnd</dt>
    <dd>The timestamp immediately after the client has finished establishing a connection to the server. If the page was cached or no new connection was made, this contains the same value as <code>domainLookupEnd</code>.</dd>
    <dt>secureConnectionStart</dt>
    <dd>This member may not exist if the client does not support it (it will be <code>undefined</code>). It contains the timestamp that a secure connection was established but before the client performed an SSL handshake. If the page was not requested over HTTPS, this contains zero. This value (when non-zero) will always fall between <code>connectStart</code> and <code>connectEnd</code>.</dd>
    <dt>requestStart</dt>
    <dd>
        The timestamp that the browser began a request to the remote server. This is the point at which the browser first started to send request headers. If the request needed to be restarted, it will contain the timestamp of the last attempt to send the headers.<br><br>
        Note that no `requestEnd` exists: this is intentional, as computing this time can be expensive and it doesn't necessarily correspond with a time that the server receives the request.<br><br>
        If the page is cached, this is the timestamp that the cache is queried.
    </dd>
    <dt>responseStart</dt>
    <dd>The timestamp that the server receives the first byte of the response. If the page is cached, this is the timestamp that the cache responds with content.</dd>
    <dt>responseEnd</dt>
    <dd>The timestamp that the server receives the last byte of the response. If the page is cached, this is the timestamp that the cache finishes sending content.</dd>
    <dt>domLoading</dt>
    <dd>The timestamp at which the browser sets <code>document.readyState</code> to <code>"loading"</code>. This is the time that the browser begins constructing the DOM.</dd>
    <dt>domInteractive</dt>
    <dd>The timestamp at which the browser sets <code>document.readyState</code> to <code>"interactive"</code>. This is the time that the browser has finished parsing all of the markup on the page but hasn't finished loading all of the assets.</dd>
    <dt>domContentLoadedEventStart</dt>
    <dd>The timestamp immediately before the browser begins firing the <code>DOMContentLoaded</code> event.</dd>
    <dt>domContentLoadedEventEnd</dt>
    <dd>The timestamp immediately after the browser finishes firing the <code>DOMContentLoaded</code> event.</dd>
    <dt>domComplete</dt>
    <dd>The timestamp at which the browser sets <code>document.readyState</code> to <code>"complete"</code>. At this point, all resources required to load the page have finished processing.</dd>
    <dt>loadEventStart</dt>
    <dd>The timestamp immediately before the browser begins firing the <code>load</code> event on the <code>window</code> object.</dd>
    <dt>loadEventEnd</dt>
    <dd>The timestamp immediately after the browser finishes firing the <code>load</code> event on the <code>window</code> object.</dd>
</dl>

Google Analytics will collect some basic information for you, including the total load time, DNS lookup time, time taken to create a connection, time between request and the beginning of the server's response, time until `fetchStart`, `domInteractive`, and `domContentLoadedEventStart`. [^1] These different timings are exposed under Behavior > Site Speed > Page Timings. On that page, you can select the various metrics from the dropdown under DOM Timings and Technical.

[^1]: This is based on an inspection of http://www.google-analytics.com/ga.js

Google Analytics only collects eight precomputed values, though, and each of the individual values of the navigation timing object are not collected or exposed. It's trivial to collect this information yourself, though there are libraries (such as Boomerang[^2]) that will collect this information for you. Ideally, you would post this information via `XMLHttpRequest` or `navigator.sendBeacon` and aggregate it in a solution like Hive or Splunk (any data analytics solution that you're comfortable with will do).

[^2]: https://github.com/yahoo/boomerang
