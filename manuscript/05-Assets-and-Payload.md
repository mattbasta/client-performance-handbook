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
- **X-Backend-Server**: Used as debugging information for some load balancers.

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
Cache-Control: public, max-age=N
Date: <now>
```

If you wanted the content to expire after 300 seconds, you would replace `N` with `300`.


#### My content will change when the cookie changes.

For this, we want to use the `Vary` header. This tells the client that if a request is made where one of the specified request headers is different, don't use the cached copy. So for instance, let's assume the client makes a request with `Cookie: mycookie` and gets a response containing:

```raw
Cache-Control: public
Vary: Cookie
```

The client will keep the response in the cache. If the client were to immediately request the content again, it would hit the cache.

Now let's say another response contains the header `Set-Cookie: anothercookie`, and the client then tries to make the original request again. Now, the `Cookie` header does not match, and the cache is not hit.


### CORS

Some sites take advantage of a technology known as CORS to make cross-domain requests. For instance, a page on `x.example.com` might want to make an AJAX request to `y.example.com`. Normally, this is not allowed and the `XMLHttpRequest` object would fire an `error` event. CORS allows the remote hostname to specify a set of HTTP headers that instruct the browser to allow the request to take place.

The downside to CORS is that for any non-idempotent request (i.e.: any HTTP method other than HEAD, GET, or OPTIONS), the browser makes a second request known as a "CORS pre-flight". The pre-flight request is an OPTIONS request to the URL that was requested. The browser expects to receive a response containing headers that look like this:

```raw
Access-Control-Allow-Methods: GET,POST,OPTIONS
Access-Control-Allow-Origin: x.example.com
```

When the browser sees this response, it knows that it is allowed to perform the request and submits the what the `XMLHttpRequest` object would otherwise have submitted had it not been pointed at a remote domain.

This pre-flight procedure blocks the "actual" HTTP request from taking place, meaning that for every one AJAX request made, two network requests are made. On high-latency connections, this can cause a significant amount of sluggishness for sensitive network requests.

There are a number of solutions: the first is to simply not make the request from a remote domain. If possible, put the endpoint that you are accessing on the same hostname. This cannot always be done, such as when the remote hostname is managed by a third party.

The second solution is to use an iframe proxy. In this approach, requests are routed through an iframe using `postMessage` (checking origins, of course) so that the HTTP request is made from the destination origin. This, of course, requires that the remote host can host such an HTML file and that it does not specify an `X-Frame-Options` header that blocks framing.

![Routing the request through an iframe](images/cors_iframe_proxy.png)

A third approach is to use the the `Access-Control-Max-Age` header, which specifies how long the browser should cache the result of the pre-flight request for. The header should look something like this:

```raw
Access-Control-Max-Age: 86400
```

Unfortunately, `Access-Control-Max-Age` is not supported in Firefox at the time of writing, and support in Chrome seems to set a maximum value of 600 (ten minutes)[^chrome_cors_bug]. Internet Explorer support is unknown.

[^chrome_cors_bug]: http://crbug.com/131368

A fourth option is to use JSONP. This involves injecting a `<script>` tag into the page, which--when loaded--makes a function call to a callback method to pass results. This approach works well, though JSONP is not always available. Additionally, making JSONP requests to a third party introduces security issues.

If a pre-flight request must take place, SPDY can be implemented on the remote host to help minimize the overhead of creating a separate connection for the pre-flight.


## Gzip and Compression

When making a HTTP request, the client will send an `Accept-Encoding` header containing a list of compression formats that it supports. In virtually all browsers, this looks something like `Accept-Encoding: gzip,deflate`.

All browsers released after 2000 support Gzip. Gzip is based on DEFLATE, and support for it goes back all the way to Internet Explorer 5.5. Partly out of lack of usage and partly out of a lack of standardized implementation, browsers are beginning to drop DEFLATE support. It is not advisable to use DEFLATE for any reason, as it will almost certainly cause issues.

The strength of Gzip is its ability to combine repeated strings. For content like HTML or CSS, this is very good because there is usually a lot of repetition. Other content may not compress quite as well. A file containing uniformly random noise, for instance, might even get larger.

Some file formats are already compressed. PNGs, WebP, video and audio, ZIP archives, and Office files are just some of the formats that will not benefit at all from additional compression. In fact, the CPU resources required to re-compress the content will likely outweigh potential benefits, if any.

The "gzippability" of a file can be improved slightly by making its contents more repetitive. For instance, consider the following CSS:

```css
.button.ready, button.ready {
    color: red;
    border-radius: 5px;
    height: 10px;
}
button.disabled, .button.disabled {
    border-radius: 15px;
    height: 15px;
    color: red;
}
```

In this sample, there's two innocuous-looking declarations. Both are simple enough, but only very small strings are repeated. This snippet, if rewritten could provide a bit of extra benefit:

```css
.button.ready, button.ready {
    border-radius: 5px;
    color: red;
    height: 10px;
}
.button.disabled, button.disabled {
    border-radius: 15px;
    color: red;
    height: 15px;
}
```

By alphabetizing declarations and ordering selectors, the Gzipped size of the snippet drops from 128 bytes to 123. Five bytes of savings isn't much, but this number increases with the size of the file as there are more opportunities for repetition to be decreased. For sizable files, simply performing safe sort operations can decrease file size by up to 5%.

Gzip significantly decreases in efficiency for files under 1KB. Many files at this size will fit into a single TCP packet anyway, or are even outweighed by their response headers. Compressing them can end up being a waste of CPU resources on the server. As with the example above, a savings of only four bytes may not be worth the trouble.


### Alternative Compression

While Gzip may be available as a powerful tool that works good in general, better compression can be achieved with purpose-built compression algorithms that are decompressed in JavaScript. For example, consider smaz[^smaz], an algorithm for compressing small strings. Such an algorithm can have a huge impact on files containing lots of small strings, like localization packages. If all of the strings only contain small amounts of repetition, Gzip will compress the file quite poorly, while an algorithm like smaz will provide very impressive compression (up to 50% in some cases). At least two JS ports of smaz currently exist.

[^smaz]: https://github.com/antirez/smaz

In other circumstances, a different approach may be taken. For large 3D games compiled with Emscripten, there may be hundreds of megabytes, if not gigabytes of assets that need to be downloaded. These assets are usually bundled into a single resource pack to save bandwidth and processed and stored into something like IndexedDB. Gzip may provide very good compression in this case, but it may also have a very high CPU cost. Having the resources decoded sooner may be more valuable than decreasing the load time of the large resource bundle, making an algorithm like lz4[^lz4] or Snappy[^snappy] more desirable.

[^lz4]: https://code.google.com/p/lz4/

[^snappy]: https://code.google.com/p/snappy/


### SDCH

Chrome recently added SDCH to its `Accept-Encoding` list. SDCH--short for Shared Dictionary Compression over HTTP--is a mechanism for compression across multiple files. This is accomplished using a "shared dictionary"--sort of like a big listing of all commonly repeated strings of text across all files. Instead of each file containing a mapping of repeated strings like in Gzip, the central dictionary is used.

SDCH is currently only supported in Chrome, though Firefox support may be forthcoming once some kinks are worked out. Very few websites actually use SDCH compression, unfortunately, and it appears Google has put the project on ice, perhaps because of lack of browser buy-in.


## Images

There are three viable image formats available on the web today:

JPEG
: The JPEG ("Joint Photographic Experts Group") image format is useful for storing photos and other graphics containing irregular or highly-textured content. JPEGs do not have an alpha channel.

GIF
: Previously encumbered by patents, the controversy surrounding licensing on the GIF ("Graphics Interchange Format") format has ended. GIFs are commonly used to encode small images, or images with few colors. Graphics with fewer than 256 colors can be losslessly represented by GIF. Using extensions, GIFs can represent far more colors or include animations, though this has a serious negative impact on file size. GIFs can indicate a transparent color, though they do not have an alpha channel.

PNG
: The PNG ("Portable Network Graphics") format was developed to circumvent the patent issues surrounding GIF. Unlike GIF, PNGs support true-color imaging, full alpha transparency, and gamma correction. PNGs can be compressed to eliminate some of those features. PNG does not support animation.

BMP
: One of the earliest formats, BMP ("Bitmap") encodes the color of each pixel individually. This results in very large file sizes, though images are encoded losslessly. BMP does not support an alpha channel. Despite its widespread support, the use of BMP files on the web is highly discouraged.

SVG
: Unlike other formats in this list, SVG ("Scalable Vector Graphics") stores image data in vector representation. Line art, regular shapes, and other smooth graphics can be represented by SVG with very small file sizes. Image data is encoded as shapes in an XML format. SVG images can be zoomed and scaled with no loss in quality.


Additionally, there are a number of formats that are only implemented by a subset of browsers:

JPEG XR
: Also known as "HD Photo," JPEG XR ("Extended Range") was developed by Microsoft to provide better compression, lossless compression support, separate encoding for different parts of an image, better color accuracy, alpha transparency, and more. JPEG XR is only supported in IE9 and up, however, and provides virtually no tangible benefits in comparison to WebP[^jpeg_xr_analysis].

TIFF
: TIFF ("Tagged Image File Format") is one of the first image formats supported on the web, though it currently has little to no support in any modern browsers. Many users can view TIFF files, though, through the use of third party plugins. While TIFF files are generally very large, they do offer a plethora of features that are not available in any other format, such as layers and sub-files. TIFF files are comparable in some ways to Photoshop's PSD format. Currently, TIFF is available to Internet Explorer and Safari users only; Mozilla has publicly stated that it has no plans to add support.

APNG
: PNG does not natively support animations (as GIF does), so APNG ("Animated Portable Network Graphics") was developed to fill this need. APNG is only supported in Firefox, though a Chrome extension is available.

WebP
: Currently--at the time of writing--only supported in Chrome[^firefox_webp], WebP hosts all features of PNG and GIF while providing very good compression of photographic images (like JPEG) with very little perceivable quality loss. Originally criticized for lacking animation support among other features, the latest versions of WebP have addressed all noted drawbacks.

[^jpeg_xr_analysis]: http://people.mozilla.org/~josh/lossy_compressed_image_study_october_2013/

[^firefox_webp]: Though WebP is only officially supported in Chrome, Firefox does include a WebP decoder, as it is built into the libwebm library that Firefox uses to decode WebM files. A lack of support is purely political. See https://bugzil.la/856375


### Use the right encoding for the job

Choosing the appropriate image format can sometimes be tricky. The wrong format can cause exceptional performance problems with a website. For example, encoding a photograph as a PNG file could triple the file size of the image.

The flow chart below should make it easier to choose which format to use.

![Image formats and their properties](images/image_format_choices.png)

Note that the rules above are not hard and fast. Sometimes, it may be more efficient to encode an image as PNG instead of JPEG if the image is sufficiently small, or if the number of colors in the image is lower than 256. Experiment to see which format provides the best file size for your needs.


### Responsive images and performance

A recent trend in web design is the use of high-resolution images to improve the appearance of pages on high-dpi (or "retina") displays. These images are usually saved at 300 dpi, though, and this can cause issues for users on slow connections (this is likely, as many users with high-dpi displays are using modern smartphones and tablets on mobile networks).

To begin with, realize that a 72dpi image that's enlarged to 300dpi is not going to look any sharper on a retina display than the 72 dpi original. An image will only be as high-quality as its original version.

Additionally, there's a fine balance between size and quality: rather than increasing the image's resolution, it may be more effective to simply increase the quality setting. A JPEG at 70% quality at 300dpi will look almost identical to a full-quality JPEG at 200DPI when scaled. Depending on the application encoding the image and the number of pixels, the smaller-but-higher-quality image may have a smaller file size.

Lastly, ensure you are using the appropriate encoding for the image format that you're using. Ensure that the image loads progressively (starts at a low quality and gets clearer) rather than sequentially (starts at the top and loads down). More information on encoding will be provided in a later section.


#### Change the image via media queries in CSS

One of the most common approaches is to swap out a non-retina image for the retina version using CSS. This is done with a fairly simple `@media` block:

```css
.hero-content {
    background-image: url(/images/hero.jpg);
}

/* If the screen is high-dpi... */
@media only screen and (-webkit-min-device-pixel-ratio: 1.5),
    only screen and (min--moz-device-pixel-ratio: 1.5),
    only screen and (min-device-pixel-ratio: 1.5) {

    /* ...then use a high-dpi image instead */
    .hero-content {
        background-image: url(/images/hero_2x.jpg);
    }
}
```


#### The `<picture>` element

Upcoming versions of WebKit and Blink will support a new HTML element known as the `<picture>` element. This element allows developers to specify multiple versions of the same image with criteria for which version to display. An example taken from the specification:

```html
<picture>
  <source media="(min-width: 45em)" srcset="large.jpg">
  <source media="(min-width: 18em)" srcset="med.jpg">
  <img src="small.jpg" alt="The president giving an award.">
</picture>
```

By default (and in older browsers), the `<img>` tag shows `small.jpg`. In browsers that support the element, a media query (the `media=""` attribute) chooses `med.jpg` or `large.jpg` depending on the width of the element on the page. The element has additional functionality which I won't get into that allows alternative resolutions of each of those images to be specified in the `srcset=""` attribute.

At the time of writing, Chrome includes an implementation of `<picture>`, which can be enabled using the "Enable experimental Web Platform features" flag under `chrome://flags`.


#### Use vector graphics where possible

SVG is fairly well-supported across the board. Unlike raster graphics like PNG and JPEG, SVG images don't pixelate as they are scaled. On high-resolution displays, SVG images remain crisp and don't become fuzzy. Additionally, SVG files are usually much smaller than PNG or JPEG files, making them great candidates for interfaces.

Another vector options is web fonts. Fonts are encoded as vectors, and can be loaded using CSS's `@font-face` block. Most modern browsers support the WOFF format, but TTF or ODT fallbacks should be provided. As an added bonus, multiple small images can be encoded into a single font file (one image per glyph), reducing the number of connections needed when not running on SPDY.

Beware, however, when using fonts: most pre-made icon fonts (like Font Awesome) include hundreds of icons. These can bloat the font file if they are unused, resulting in significant amounts of wasted bandwidth. Use a tool to remove unused glyphs from the font file before uploading it.


### Decreasing image file sizes

Most image editing applications include a significant amount of metadata in the images that are produced. This data includes information about the software that created the image, legal information, comments, description, title, etc. In the context of the web, none of this information is useful to a browser and only takes up valuable bytes.

Additionally, most image editing software uses encoding algorithms that are sub-par. Inefficiencies in the algorithms mean that bytes are needlessly wasted as image data is written in a more verbose way than necessary. This is not to say that the images will decrease in quality when re-encoded: the images should be losslessly re-written to a smaller file representation.

There are many tools available that do this for many different file types. The following are a few of the most popular command line tools:

- **pngcrush**: A tool for compressing PNG images and stripping unneeded metadata
- **PNGOUT**: Another tool for compressing PNG images
- **Jpegoptim**: A tool that provides lossless JPEG optimization
- **Gifsicle**: Rewrites GIF files to their smallest possible form

If you use OS X, an open source tool called ImageOptim can be used to automate these tools with a dead-simple GUI.

![ImageOptim compressing images from this very book](images/image_optim.png)

You can download ImageOptim for free from http://imageoptim.com/


### Image encoding and quality

For many common formats--in particular, PNG and JPEG--there are a number of settings that you can use to improve the size, perceived performance, and quality of the images that you generate.


#### PNG

For small PNG files with few colors, 8-bit PNGs are the best option. In Adobe products, this option is listed as "PNG-8". Using PNG-8 instead of "true color" PNGs can save a significant amount of bandwidth, especially if the number of colors the image uses is minimized.

A little known fact is that the 8-bit PNGs do indeed support full alpha transparency. Adobe Photoshop, however, does not support this feature (reading or writing). Adobe Fireworks supports this option, though, as do other PNG utilities.

Additionally, if you have the option, enable PNG interlacing for any file larger than a couple kilobytes. Interlacing enables the full PNG to begin rendering earlier in the loading process and gradually increase in quality as more data loads.


#### JPEG

When using Adobe products, saving a JPEG under the "Save for Web" or "Optimize" menus provides an option labeled "Optimized". This should always be checked: it reduces file size at the cost of dropping support for some very old software.

"Progressive" JPEGs are another option you may receive (in Adobe products and elsewhere), which behaves similarly to the interlaced PNG option. Any large JPEGs should be encoded as progressive.

As far as quality is concerned, a quality setting of "High" (or about 60% on a 0 to 100 scale) is perhaps the highest that any image needs to be encoded as. Above that, the file size increases tremendously with little improvement in quality.


### SVG is a curse

The only vector graphics format that works natively in all browsers is SVG. SVGs are generally relatively small, and can be very compressed well in some cases. Unfortunately, visually complex images that include irregular shapes can bloat the size of the SVG. For instance, consider the following path:

```xml
<path d="M5.9482467,13.4977018 L-2.59854728,6.47190027 C-3.38658022,5.83462046 -4.65285911,5.84398612 -5.42686033,6.49281906 C-6.20086155,7.141652 -6.1894866,8.18425239 -5.40145365,8.8215322 L4.55314211,16.8717831 L5.94824679,18 L7.34955712,16.8770089 L17.3949616,8.82675797 C18.1864999,8.19242996 18.2036227,7.14988283 17.4332066,6.49816377 C16.6627904,5.84644472 15.3965762,5.83234649 14.605038,6.4666745 L5.9482467,13.4977018 Z M5.9482467,13.4977018" id="Path 5" fill="#C7C7C7" transform="translate(6.000000, 12.000000) rotate(-270.000000) translate(-6.000000, -12.000000)"></path>
```

Though this is a simple shape, it can't be described using nice, round numbers. Shaving decimal places from the numbers in an SVG can help to decrease its size significantly:

```xml
<path d="M5.948,13.497 L-2.598,6.471 C-3.386,5.834 -4.652,5.843 -5.426,6.492 C-6.200,7.141 -6.189,8.184 -5.401,8.821 L4.553,16.871 L5.948,18 L7.349,16.877 L17.394,8.826 C18.186,8.192 18.203,7.149 17.433,6.498 C16.662,5.846 15.396,5.832 14.605,6.466 L5.948,13.497 Z M5.948,13.497" fill="#C7C7C7" transform="translate(6, 12) rotate(-270) translate(-6, -12)"></path>
```

When gzipped, the second version is over 100 bytes smaller.

- All numbers were rounded to three decimal places. Decreasing the precision of numbers in an SVG can cause some minor visual glitches, but these are generally small enough to not be noticeable. When these glitches become problematic, simply increase the precision for the critical numbers.
- Trailing zeros were removed. All numbers are treated the same in SVG, so having a decimal place or trailing zeros does not affect the output.
- Removed needless `id` element. Many vector graphics applications save lots of information to SVG files that simply isn't used.

This simple optimization can generally save a significant amount of space. This is not true in all cases, though. For example, small and uniform PNG files can actually compress better than their SVG counterparts (albeit in raster form). Even some much larger graphics--depending on their contents--can compress better in raster form.

Additionally, browser support for SVG features is mediocre at best. No modern browsers support the full SVG spec, and most have no plans to implement the remaining portions due to cost, complexity, and lack of demand.

Complex SVGs can also introduce very significant rendering challenges. Any SVG larger than a few kilobytes is generally complex enough to cause double or even triple-digit (millisecond) render times. Depending on what's above or below the SVG (on the Z-axis, not the Y-axis), having SVGs on your page can have very serious negative impacts on scrolling performance: scrolling can become jittery, stutter, or--in the worst circumstances--cause your page to become completely unusable. This is especially true on mobile devices where CPU and GPU resources are limited.

Keeping the render performance of your SVGs within an acceptable range unfortunately involves keeping them simple. Reducing the number of shapes, effects, etc. in an SVG will minimize the amount of computation that the client needs to perform. Different browsers have wildly different performance characteristics for SVGs, so there are no blanket rules that can be followed.

Most SVG features should be avoided for accessory features of pages. For instance, avoid using gradients, animations, filter effects, and masking in SVG images that are used as part of the UI for a site rather than the content. This helps to avoid introducing computationally expensive render operations to each of your the pages on your site.


### Spriting



### Post-loading Images
### Spacer GIFs: Not even once
### You don't need that image

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
