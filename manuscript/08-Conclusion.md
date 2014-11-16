# Conclusion

## Notable Tickets

Throughout this book, I've made note of many tickets in bug trackers for different browser vendors. This section is simply a collection of those tickets, along with a summary of each of the issues.

### Firefox

- **Bug 688580:** Deferred scripts run at the wrong time http://bugzil.la/688580
- **Bug 730101:** Implement prerendering support http://bugzil.la/730101
- **Bug 856375:** Implement WebP support http://bugzil.la/856375


### Chromium

- **Issue 128055:** This issue prevents CSS properties from using hashes to reference individual components of an SVG image. This functionality is available in IE9 and up as well as Firefox. http://crbug.com/128055
- **Issue 131368:** Chromium does not respect CORS header caching for longer than ten minutes. http://crbug.com/131368
- **Issue 312327:** This issue causes subresource requests not to be used when an asset is requested in a document. This prevents subresource requests from being useful. http://crbug.com/312327


### Internet Explorer

Note that IE does not have a public issue tracker. The following are tickets that have been filed against third party issue trackers documenting bugs in Internet Explorer.

- `readystatechange` exhibits incorrect behavior surrounding `document.readyState === 'interactive'`. http://bugs.jquery.com/ticket/12282
- IE9 and below do not support `defer` correctly. https://github.com/h5bp/lazyweb-requests/issues/42


## Using Chrome's net-internals Tool

An extremely useful tool for debugging network-related problems is Google Chrome's net-internals tool. net-internals gives you access to most information about ongoing requests in the browser.

To access net-internals, simply visit `chrome://net-internals` in a new tab. The tool loads immediately and begins capturing data.

net-internals captures quite a lot of information (you can see the number of events it observes at the top of the tool). Capturing only the events you care about helps to filter noise from the output. To stop capturing data, press the link with the name "capturing" in the header. This will switch to the "Capturing" view, which contains a "Stop" button to stop capturing events. Pressing "Stop" will turn the header of the tool green to indicate that no new events are being captured.

In the upper left corner of the tool is a dropdown menu. This menu allows you to switch between different net-internals features.


### Features

#### Proxy

The proxy feature shows information about HTTP proxy servers that the browser is using. In general, the information on this page is not very useful for debugging performance-related issues, though it often can provide a sanity check for sketchy wi-fi connections in hotels or other businesses.


#### DNS

This page of net-internals shows a detailed description of DNS lookups that take place in the browser. It can be useful for determining when or whether DNS lookups are taking place.

For example, you might notice in the developer tools that clicking a link between pages triggers a costly DNS lookup. You might find with the DNS feature of net-internals that the browser is not preemptively performing a DNS lookup on the previous page, which might indicate that you need to add DNS lookup hints (for example, using `<link rel="dns-prefetch" ...>`).


#### Sockets

The Sockets page shows all sockets that are open in the browser. This tool is helpful for determining what connections are being made to a particular host, how, and why the requests are being made the way that they are.

For example, you might use this page to determine whether a server is properly using HTTP 1.1's persistent connection functionality to re-use connections to the server. A server that is misconfigured will show that its sockets disappear from the list of live sockets immediately after the last request for a page completes.


#### SPDY and QUIC

The SPDY and QUIC pages show active connections for their respective protocols. The QUIC page is perhaps less useful because QUIC is virtually nonexistent outside Google, but the SPDY tab shows a wealth of useful information about active SPDY sessions.

First, the SPDY tab shows which SPDY protocol is being used for a particular host. This can help to find configuration issues with SPDY-enabled servers. A server running SPDY 3, for example, should be updated to use SPDY 3.1 or newer. This will also show servers reporting as `h2-14`, which indicates that they are configured to use HTTP2.

In many cases, servers use a default SPDY configuration and no settings need to be applied manually. If SPDY or HTTP2-related problems arise, however, this page of net-internals can help to debug why the connection is failing. A very small send or receive window, for instance, might indicate that something is misconfigured.


#### Cache

This page simply provides statistics about the browser cache. Clicking the link at the top of the page to view cache entries will take you to the Chrome cache viewer, which lists all items currently in the cache. This page is not a part of net-internals, but it can provide valuable information about why and how items are cached in the browser. Clicking an item in the list shows the cached headers and a binary blob representing the cached response.


#### Modules

This page details loaded browser extensions and is not very useful for performance debugging.


#### Tests, Bandwidth, and HSTS

Tests is a page that provides useful diagnostics for why a particular URL failed to load. The URL will attempt to be loaded with multiple different configurations (using the system proxy settings, without proxies, Firefox's proxy settings, IPv6 hostname support, etc.). If the URL fails to load due to a connection issue, the error associated with the failure will be displayed.

Bandwidth simply shows the total number of kilobytes sent and received for the current browsing session.

HSTS runs in a similar vein. Though the tools on this page do not generally help with most performance issues, it can help explain why a server's HSTS configuration does not work as expected. HSTS misconfiguration can result in a server forcing a browser to redirect from HTTP to HTTPS each time a user visits the site. The tools on this page are also capable of saving user-provided HSTS information to simulate HSTS on a server. This can help with determining whether HSTS will benefit a site.

All of these features are only available while net-internals is capturing events.


#### Prerender

If your site takes advantage of prerendering (discussed in chapter five), this page will be of great value. This page shows the following information about prerendering:

- Which pages are currently being prerendered (if any)
- How a prerender was triggered (omnibox, `<link>` tag, etc.)
- Whether the prerender was successful and why

If prerendering does not seem to be working properly, this page will provide an explanation for why.


#### Events




### Usage


## Mentioned Tools

- CSS sprite tool: Spritegen - http://spritegen.website-performance.org/
- CSS sprite tool: Spritificator - http://potch.me/projects/spritificator/
- CSS minification tool: clean-css - https://github.com/GoalSmashers/clean-css
- CSS minification tool: mincss - https://mincss.readthedocs.org/
- JavaScript minifier: JSMin - http://www.crockford.com/javascript/jsmin.html
- JavaScript minifier: Closure Compiler - https://developers.google.com/closure/compiler/
- JavaScript minifier: UglifyJS2 - https://github.com/mishoo/UglifyJS2
- JavaScript parser: esprima - https://github.com/ariya/esprima
- JavaScript parser: acorn - https://github.com/marijnh/acorn
- JavaScript code generator: escodegen - https://github.com/Constellation/escodegen


## Glossary

### CSS

Declaration
: A declaration is an item within a rule set that applies a given property to the set of matched elements. E.g.: `color: red;`

Rule Set
: A rule set is a selector and zero or more declarations wrapped in curly braces. E.g.: `.myElement {font-weight: bold;}`

Selector
: A selector is the string which describes which elements a rule set applies to. E.g.: `#myElement .withAClass p`


### JavaScript

Event
: An object representing something that happened in the browser and the triggering of JavaScript code that has been put in place to accept it.

JIT Compilation
: JIT compilers, or Just-In-Time compilers, are a cross between interpreters and static compilers. Code running in a JIT-enabled interpreter will run slowly as the interpreter finds out how the code works. Then, the JIT compiler will replace the slow code with fast, optimized code.

Run To Completion
: The process of triggering a JavaScript function, then waiting for all subsequent synchronous processing to complete.


## Thanks

I'd like to use this space to thank my friends, family, and past and present coworkers for helping me to make this book possible. So many people have been supportive, offered advice or consultation, helped with fact-checking, proofreading, and more, and made contributions to improve this work.

In particular, I'd like to thank:

Chris Van (@cvanw)
: Many thanks to Chris for his incessant focus on performance and attention to detail. Your pull requests and comments, as well as your ever-inquisitive nature have driven me to make this book as thorough and exhaustively accurate as I could possibly make it.
