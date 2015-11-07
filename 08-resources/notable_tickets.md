# Notable Tickets

Throughout this book, I've made note of many tickets in bug trackers for different browser vendors. This section is simply a collection of those tickets, along with a summary of each of the issues.

## Firefox

- **Bug 641069:** Implement SDCH https://bugzil.la/641069
- **Bug 688580:** Deferred scripts run at the wrong time https://bugzil.la/688580
- **Bug 730101:** Implement prerendering support https://bugzil.la/730101
- **Bug 856375:** Implement WebP support https://bugzil.la/856375


## Chromium

- **Issue 128055:** This issue prevents CSS properties from using hashes to reference individual components of an SVG image. This functionality is available in IE9 and up as well as Firefox. https://crbug.com/128055
- **Issue 131368:** Chromium does not respect CORS header caching for longer than ten minutes. https://crbug.com/131368
- **Issue 312327:** This issue causes subresource requests not to be used when an asset is requested in a document. This prevents subresource requests from being useful. https://crbug.com/312327
- **Issue 472009:** Chromium intends to implement Brotli as an accepted content encoding. https://crbug.com/472009


## Internet Explorer

Note that IE does not have a public issue tracker. The following are tickets that have been filed against third party issue trackers documenting bugs in Internet Explorer.

- `readystatechange` exhibits incorrect behavior surrounding `document.readyState === 'interactive'`. http://bugs.jquery.com/ticket/12282
- IE9 and below do not support `defer` correctly. https://github.com/h5bp/lazyweb-requests/issues/42
