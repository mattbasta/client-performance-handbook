# Gzip and Compression

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

While Gzip may be available as a powerful tool that works good in general, better compression can be achieved with purpose-built compression algorithms that are decompressed in JavaScript. For example, consider smaz[^1], an algorithm for compressing small strings. Such an algorithm can have a huge impact on files containing lots of small strings, like localization packages. If all of the strings only contain small amounts of repetition, Gzip will compress the file quite poorly, while an algorithm like smaz will provide very impressive compression (up to 50% in some cases). At least two JS ports of smaz currently exist.

In other circumstances, a different approach may be taken. For large 3D games compiled with Emscripten, there may be hundreds of megabytes, if not gigabytes of assets that need to be downloaded. These assets are usually bundled into a single resource pack to save bandwidth and processed and stored into something like IndexedDB. Gzip may provide very good compression in this case, but it may also have a very high CPU cost. Having the resources decoded sooner may be more valuable than decreasing the load time of the large resource bundle, making an algorithm like lz4[^2] or Snappy[^3] more desirable.

[^1]: https://github.com/antirez/smaz
[^2]: https://code.google.com/p/lz4/
[^3]: https://code.google.com/p/snappy/


### SDCH

Chrome recently added SDCH to its `Accept-Encoding` list. SDCH--short for Shared Dictionary Compression over HTTP--is a mechanism for compression across multiple files. This is accomplished using a "shared dictionary"--sort of like a big listing of all commonly repeated strings of text across all files. Instead of each file containing a mapping of repeated strings like in Gzip, the central dictionary is used.

SDCH is currently only supported in Chrome, though Firefox support may be forthcoming once some kinks are worked out. Very few websites actually use SDCH compression, unfortunately, and it appears Google has put the project on ice, perhaps because of lack of browser buy-in.
