# Caching Correctly

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


## I want my content to never be cached.

If you want you content to never be stored by the browser, simply send `Cache-Control: no-cache`. This disables caching in all browsers[^ie6_nocache].

[^ie6_nocache]: IE6 will sometimes ignore this and cache anyway. If you care about IE6, use `Cache-Control: no-cache, no-store`.


## I want my content to be cached, but for the client to check back.

In this case, the client will cache the content, but will check to see if it changed every time the content is requested. If the content hasn't changed, the server will respond with a `304 Not Modified` and an empty body, saving time.

The `must-revalidate` directive does the trick here. It tells the client that every time, it must check back to see if the resource has changed.

```raw
Cache-Control: max-age=0, must-revalidate
ETag: <etag>
Last-Modified: <last-modified-date>
```


## I want my content to be cached until a certain date.

In this case, you have a piece of content with a known expiration date sometime in the future. You don't want the client to check if the content is valid until that expiration date has passed.

```raw
Cache-Control: public
Date: <now>
Expires: <expiration-date>
```


## I want my content to expire exactly N seconds after it is requested.

This can obviously be accomplished with the `Expires` header, but that means that you need to compute the `Expires` header on the fly. If you're hard-coding the value, it's simpler to say:

```raw
Cache-Control: public, max-age=N
Date: <now>
```

If you wanted the content to expire after 300 seconds, you would replace `N` with `300`.


## My content will change when the cookie changes.

For this, we want to use the `Vary` header. This tells the client that if a request is made where one of the specified request headers is different, don't use the cached copy. So for instance, let's assume the client makes a request with `Cookie: mycookie` and gets a response containing:

```raw
Cache-Control: public
Vary: Cookie
```

The client will keep the response in the cache. If the client were to immediately request the content again, it would hit the cache.

Now let's say another response contains the header `Set-Cookie: anothercookie`, and the client then tries to make the original request again. Now, the `Cookie` header does not match, and the cache is not hit.