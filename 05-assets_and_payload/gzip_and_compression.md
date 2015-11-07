# Gzip and Compression

When making a HTTP request, the client will send an `Accept-Encoding` header containing a list of compression formats that it supports. In virtually all browsers, this looks something like `Accept-Encoding: gzip,deflate`. Support for this header allows servers to compress the payload of a response. Before sending the content to the client, the server will encode the content with an algorithm like Gzip, reducing its size. The client will then decode the content to make it usable.

Performing extra processing on the server may seem counterintuitive: generally, doing extra work means that performance is decreased. In this case, compression allows the server to spend slightly more time preparing the content to be sent while vastly reducing the amount of time spent sending the content over the network.

All browsers released after 2000 support Gzip. Gzip is based on DEFLATE, and support for it goes back all the way to Internet Explorer 5.5. Partly out of lack of usage and partly out of a lack of standardized implementation, browsers are beginning to drop DEFLATE support. It is not advisable to use DEFLATE for any reason, as it will almost certainly cause issues.

The strength of Gzip is its ability to combine repeated strings. For content like HTML or CSS, this is good because there is generally a lot of repetition. Other content, such as compressed PNG or web font files, may not see much or any benefit.

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


## Brotli

Google has recently open-sourced a compression algorithm called *Brotli*[^1], and the algorithm has been published (at the time of writing) as an IETF draft[^2]. Brotli provides compression that can be 10-25% better than Gzip in many cases, and provides decompression speeds that are generally better than Gzip.

As of Firefox 44, Brotli can be used as a content encoding on the web[^3]. Google has indicated that they intend to implement Brotli support in Chrome, as well[^4]. Once this support is available in stable browser builds, they will advertise their support in the `Accept-Encoding` header:

```
Accept-Encoding: gzip,deflate,br
```

Enabling Brotli support on your server involves installing the appropriate plugin. Google has provided `ngx_brotli`[^5], a plugin for Nginx servers. At the time of writing, no plugins are available for IIS, Apache HTTPD, or Apache Traffic Server.

[^1]: https://github.com/google/brotli
[^2]: https://tools.ietf.org/html/draft-alakuijala-brotli-07
[^3]: https://bugzilla.mozilla.org/show_bug.cgi?id=366559
[^4]: https://code.google.com/p/chromium/issues/detail?id=472009
[^5]: https://github.com/google/ngx_brotli


## Alternative Compression

While Gzip may be available as a powerful tool that works good in general, better compression can be achieved with purpose-built compression algorithms that are decompressed in JavaScript. For example, consider smaz[^6], an algorithm for compressing small strings. Such an algorithm can have a huge impact on files containing lots of small strings, like localization packages. If all of the strings only contain small amounts of repetition, Gzip will compress the file quite poorly, while an algorithm like smaz will provide very impressive compression (up to 50% in some cases). At least two JS ports of smaz currently exist.

In other circumstances, a different approach may be taken. For large 3D games compiled with Emscripten, there may be hundreds of megabytes, if not gigabytes of assets that need to be downloaded. These assets are usually bundled into a single resource pack to save bandwidth and processed and stored into something like IndexedDB. Gzip may provide very good compression in this case, but it may also have a very high CPU cost. Having the resources decoded sooner may be more valuable than decreasing the load time of the large resource bundle, making an algorithm like lz4[^7], or Snappy[^8] more desirable.

[^6]: https://github.com/antirez/smaz
[^7]: https://code.google.com/p/lz4/
[^8]: https://github.com/google/snappy


### SDCH

"SDCH" is a technology that Chrome began to support as a valid value in `Accept-Encoding`. SDCH--short for *Shared Dictionary Compression for HTTP*--is a mechanism for compression across multiple files. This is done using a "shared dictionary": a listing of every string that's shard across all of the files to transfer. Instead of each file containing a mapping of repeated strings like in Gzip, the server and client use the central dictionary instead.

Unlike Gzip or other algorithms, SDCH is best-suited for tasks that involve dynamic content. For instance, a shared dictionary could be created that contains the parts of a each page on a site that never change. When a user makes a request, the server only needs to send the unique pieces of the page and an ordered list of each element from the shard dictionary to stitch together.

SDCH is only supported in Chrome, though Firefox support may be forthcoming[^9]. Very few websites implement SDCH compression unfortunately, and it appears Google has put the project on ice.

[^9]: https://bugzilla.mozilla.org/show_bug.cgi?id=641069
