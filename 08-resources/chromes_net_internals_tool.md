# Chrome's Net Internals Tool

A useful tool for debugging network-related problems is Google Chrome's `net-internals` tool. `net-internals` gives you access to most information about ongoing requests in the browser.

To access net-internals, visit `chrome://net-internals` in a new tab. The tool loads and begins capturing data immediately.

`net-internals` captures a great deal of information (you can see the number of events it observes at the top of the tool). Capturing only the events you care about helps to filter noise from the output. To stop capturing data, press the link with the name "capturing" in the header. This will switch to the "Capturing" view, which contains a "Stop" button to stop capturing events. Pressing "Stop" will turn the header of the tool green to indicate that no new events are being captured.

In the upper left corner of the tool is a dropdown menu. This menu allows you to switch between different net-internals features.


## Features

### Proxy

The proxy feature shows information about HTTP proxy servers that the browser is using. In general, the information on this page is not very useful for debugging performance-related issues, though it often can provide a sanity check for sketchy wi-fi connections in hotels or other businesses.


### DNS

This page of net-internals shows a detailed description of DNS lookups that take place in the browser. It can be useful for determining when or whether DNS lookups are taking place.

For example, you might notice in the developer tools that clicking a link between pages triggers a costly DNS lookup. You might find with the DNS feature of net-internals that the browser is not preemptively performing a DNS lookup on the previous page, which might indicate that you need to add DNS lookup hints (for example, using `<link rel="dns-prefetch" ...>`).


### Sockets

The Sockets page shows all sockets that are open in the browser. This tool is helpful for determining what connections are being made to a particular host, how, and why the requests are being made the way that they are.

For example, you might use this page to determine whether a server is properly using HTTP 1.1's persistent connection functionality to re-use connections to the server. A server that is misconfigured will show that its sockets disappear from the list of live sockets immediately after the last request for a page completes.


### HTTP/2 and QUIC

The HTTP/2 and QUIC pages show active connections for their respective protocols. The QUIC page is perhaps less useful because QUIC is virtually nonexistent outside Google. The HTTP/2 tab, however, shows a wealth of useful information about active sessions.

In many cases, servers use a default HTTP/2 configuration and no settings need to be applied manually. If HTTP/2-related problems arise, however, this section of `net-internals` can help to debug why the connection is failing. A very small send or receive window, for instance, might indicate that something is amiss in the server's configuration.


### Cache

This page simply provides statistics about the browser cache. Clicking the link at the top of the page to view cache entries will take you to the Chrome cache viewer, which lists all items currently in the cache. This page is not a part of net-internals, but it can provide valuable information about why and how items are cached in the browser. Clicking an item in the list shows the cached headers and a binary blob representing the cached response.


### Modules

This page details loaded browser extensions and is not very useful for performance debugging.


### Tests, Bandwidth, and HSTS

Tests is a page that provides useful diagnostics for why a particular URL failed to load. The URL will attempt to be loaded with multiple different configurations (using the system proxy settings, without proxies, Firefox's proxy settings, IPv6 hostname support, etc.). If the URL fails to load due to a connection issue, the error associated with the failure will be displayed.

Bandwidth simply shows the total number of kilobytes sent and received for the current browsing session.

HSTS runs in a similar vein. Though the tools on this page do not generally help with most performance issues, it can help explain why a server's HSTS configuration does not work as expected. HSTS misconfiguration can result in a server forcing a browser to redirect from HTTP to HTTPS each time a user visits the site. The tools on this page are also capable of saving user-provided HSTS information to simulate HSTS on a server. This can help with determining whether HSTS will benefit a site.

All of these features are only available while net-internals is capturing events.


### Prerender

If your site takes advantage of prerendering (discussed in chapter five), this page will be of great value. This page shows the following information about prerendering:

- Which pages are currently being prerendered (if any)
- How a prerender was triggered (omnibox, `<link>` tag, etc.)
- Whether the prerender was successful and why

If prerendering does not seem to be working properly, this page will provide an explanation for why.


### Events

The Events page is a complete log of network events that occur in the browser. Almost everything that occurs in the browser's network stack gets logged in this list. A search box at the top allows you to sort and filter the event log. Hovering the `(?)` to the left of the search box shows options for sorting.
