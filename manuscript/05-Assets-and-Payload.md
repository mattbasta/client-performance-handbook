# Assets and Payload

> A man that carries nothing through life finds his grave the fastest

Oftentimes, the slowest part of loading any single page is the time spent downloading the page's content and assets. There are many rules that browsers follow--or in some cases, don't follow--that make it difficult to understand when, how, and why requests are made and what order they're made in. There are also many little-known techniques that can be used to decrease the amount of time it takes to load these assets, whether it be by fetching content in advance, or by providing hints that clients can use to improve performance.

There are also techniques that can be used to minimize the amount of content sent over the wire. The fewer packets that need to be sent, the less time it takes to complete any given request.


## HTTP

C> HTTP: You're Stuck With It
C>
C> Welcome to the web!

HTTP is a blessing and a curse: it's the protocol that makes the web possible. A client makes a request for a piece of content with a given identifier and a set of information describing the client, and a server responds with some metadata about the resource and the content of the resource. On paper, it sounds great. In practice, HTTP is naïve and inefficient, and does little to help out with improving performance.

Most of this inefficiency resides in the fact that HTTP was designed as a serial protocol. One request would be made, and a response would be returned. The client would make another request (or more), and more responses would be returned. When Tim Berners-Lee and his team first developed the protocol, it wasn't known that within two decades content on the "WorldWideWeb" would be as rich, diverse, and interactive as it is today.


### Minimizing Headers

When a web application makes many requests to a server for many files, most of the information being sent is redundant. The client's `User-Agent` header, connection information, cache data (if available), and other information is identical between requests. SPDY minimizes the impact of this by allowing header compression, but the first request (or response) is still impacted by large amounts of header data. If this crosses the TCP window size, it may take multiple packets just to allow the server to begin processing the request or sending the actual response content.

To alleviate this, take great care to minimize the number of custom headers sent back and forth between the client and the server.

Consider the following HTTP headers from Wikipedia, when retrieved using Google Chrome:

```raw
HTTP/1.1 200 OK
Server: Apache
X-Content-Type-Options: nosniff
Content-language: en
X-UA-Compatible: IE=Edge
Vary: Accept-Encoding,Cookie
Last-Modified: Sat, 10 May 2014 12:32:06 GMT
Content-Encoding: gzip
Content-Type: text/html; charset=UTF-8
X-Varnish: 4067608437, 4126058259 4125899187, 2089471119 2040225699
Via: 1.1 varnish, 1.1 varnish, 1.1 varnish
Content-Length: 38922
Accept-Ranges: bytes
Date: Sun, 11 May 2014 00:35:30 GMT
Age: 43369
Connection: keep-alive
X-Cache: cp1067 miss (0), cp4009 hit (3), cp4008 frontend hit (126)
Cache-Control: private, s-maxage=0, max-age=0, must-revalidate
Set-Cookie: GeoIP=::::v6; Path=/; Domain=.wikipedia.org
```

All of this data is sent to the client on every request. Let's take a look at each individual header:

Server
: This header is entirely unnecessary.

X-Content-Type-Options
: This header is used in IE and Chrome as a security feature.

Content-language
: This could instead be specified in the HTML using a `lang="en"` attribute.

X-UA-Compatible
: Only Internet Explorer uses this header, and the value can be set in the HTML via a `<meta>` tag.

Vary
: This header is used for caching, though the example here is likely to prevent caching proxies from serving Gzipped content to clients that don't support it and to allow separate caches per user. Virtually all clients support Gzip, so that part is probably unnecessary.

Last-Modified
: This header is also used for caching.

Content-Encoding
: This header turns on Gzip. It is very beneficial.

Content-Type
: The content type is a normal header to send, though the encoding does not need to be sent as part of the header. It can instead be sent as part of a `<meta>` tag, or detected by the client (a process that is surprisingly accurate).

X-Varnish
: This is internal data that is not used by the client. It can be eliminated.

Via
: This header is largely unneeded, and is unnecessary in the absence of proxy servers.

Content-Length
: This header defines how much content was returned, and is not harmful.

Accept-Ranges
: This broadcasts the server's ability to return partial content. This is useful for video files, where the client may seek to a particular timestamp in the video.

Date
: This header is required by HTTP.

Age
: This header is only used by intermediate caching proxies. This header is probably not necessary in this situation since the `Cache-Control` header always requires browsers to check to see whether the content is fresh before using it.

Connection
: This header is no longer needed, as HTTP/1.1 connections are persistent by default.[^http11_persistent]

X-Cache
: Like `X-Varnish`, this header is not used by the client.

Cache-Control
: This header controls how the client (and any intermediate caching proxies) deal with the content. Details on how to use `Cache-Control` will be provided in the next section.

Set-Cookie
: This header sets a cookie. In this circumstance, though, the client's cookie is already set to the value provided, meaning the header is redundant and can be eliminated.

[^http11_persistent]: http://dxr.mozilla.org/mozilla-central/source/netwerk/protocol/http/nsHttpConnection.cpp#830

If the unnecessary headers were removed (or moved into the body) where possible, the response would look like this:

```raw
HTTP/1.1 200 OK
X-Content-Type-Options: nosniff
Vary: Accept-Encoding,Cookie
Last-Modified: Sat, 10 May 2014 12:32:06 GMT
Content-Encoding: gzip
Content-Type: text/html
Content-Length: 38922
Accept-Ranges: bytes
Date: Sun, 11 May 2014 00:35:30 GMT
Cache-Control: private, s-maxage=0, max-age=0, must-revalidate
```

The result is less than half the size, and means the client can begin receiving actual page content sooner.

Regarding cookie data, this information should be minimized as much as possible. Cookie information is sent on every single request to the host that it is defined for, meaning that a bloated cookie can cause a very significant amount of overhead if the request (or response) headers get split into multiple TCP packets.

On the client side, if there is more than a few dozen bytes worth of information that needs to be persisted, use a JavaScript interface like `localStorage` to store the information. If the information needs to be passed to the server, send it as the body of a POST request.


### Unneeded Response Headers

The following headers are obsolete and should no longer be used:

- **Pragma**: This header was made obsolete by HTTP/1.1. Use `Cache-Control: no-cache` instead.
- **Connection: keep-alive**: HTTP/1.1 connections are persistent by default. The `Connection` header should only be used to send `Connection: close`, though it's unclear why that behavior would be desirable.
- **Status**: This header is redundant and should not be used.

The following headers provide information which is not used by the client and should be removed:

- **Server**: While this header can be used for statistical purposes by researchers, it is unused in virtually all other circumstances.
- **X-Cache**: Set by some caching proxies.
- **X-Backend-Server**: Used as debuging information for some load balancers.

The following headers can be placed within HTML instead:

- Content-Language
- Content-Type (charset information)
- X-UA-Compatible


### Caching Correctly

Most site owners have a very hard time getting caching to work properly. If you've ever used YSlow or the Audit tab of the Chrome developer tools, you'll find that it often complains about poor `Cache-Control` headers. This is easy to remedy, however, if you know how your content is going to be accessed.

There are some common terms that you should know about:

Last Modified Date
: This is a HTTP date indicating the timestamp that the resource was last modified. If this value is available, it should be specified wherever possible. At worst, it goes unused. At best, the value allows the server to respond to a client with a `304 Not Modified` when it otherwise would have responded with a `200 OK`.

ETag
: This is a unique identifier representing a particular version of the resource. It can be, for instance, the MD5 hash of the content. When the content changes, the ETag should update.

Expires
: An HTTP date indicating the timestamp upon which the client should not keep a cached copy beyond.

Age
: This is the equivalent of the "created by" time for a piece of content. It is not useful for anything other than caching proxies, since an origin server can always calculate when a piece of content is good until (or it just created the content and the "age" is zero).

Date
: The `Date` header is simply the HTTP date of the current timestamp.

max-age
: This is a flag that can be set for `Cache-Control`. A `max-age=0` will require the client to check with the server each time the content is requested to see if it changed (the same behavior as `must-revalidate`). For content that should be cached for a long time, use `max-age=86400` to cache it for one day. If you change your content's URL every time you create a new version, you can specify up to `max-age=31536000` (one year).


All HTTP dates are formatted per RFC 1123.


#### I want my content to never be cached.

If you want you content to never be stored by the browser, simply send `Cache-Control: no-cache`. This disables caching in all browsers[^ie6_nocache].

[^ie6_nocache]: IE6 will sometimes ignore this and cache anyway. If you care about IE6, use `Cache-Control: no-cache, no-store`.


#### I want my content to be cached, but for the client to check back.

In this case, the client will cache the content, but will check to see if it changed every time the content is requested. If the content hasn't changed, the server will respond with a `304 Not Modified` and an empty body, saving time.

The `must-revalidate` directive does the trick here. It tells the client that every time, it must check back to see if the resource has changed.

```raw
Cache-Control: max-age=0, must-revalidate
ETag: <etag>
Last-Modified: <last-modified-date>
```


#### I want my content to be cached until a certain date.

In this case, you have a piece of content with a known expiration date sometime in the future. You don't want the client to check if the content is valid until that expiration date has passed.

```raw
Cache-Control: public
Date: <now>
Expires: <expiration-date>
```


#### I want my content to expire exactly N seconds after it is requested.

This can obviously be accomplished with the `Expires` header, but that means that you need to compute the `Expires` header on the fly. If you're hard-coding the value, it's simpler to say:

```raw
Cache-Control: max-age=300
Date: <now>
```

The client will never keep the content for longer than 300 seconds from the start of the response.


#### My content will change when the cookie changes.

For this, we want to use the `Vary` header. This tells the client that if a request is made where one of the specified request headers is different, don't use the cached copy. So for instance, let's assume the client makes a request with `Cookie: mycookie` and gets a response containing:

```raw
Cache-Control: public
Vary: Cookie
```

The client will keep the response in the cache. If the client were to immediately request the content again, it would hit the cache.

Now let's say another response contains the header `Set-Cookie: anothercookie`, and the client then tries to make the original request again. Now, the `Cookie` header does not match, and the cache is not hit.


### CORS
### Gzip and Compression
## Images
### Use the right encoding for the job
### Responsive images and performance
### Stripping metadata
### Image encoding and quality
### Proxies and images
### SVG is a curse
### The `<picture>` element
### Spriting
## Prefetching and Prerendering
## Minification
### Markup
#### SVG
### CSS
### JS
## One Asset to Rule Them All?
### Number of assets
### Combining assets
### Weighing the benefits of multiple files
