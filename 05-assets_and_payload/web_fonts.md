# Web Fonts

Web fonts are a fairly recent technology that has enabled designers to apply much more creativity to pages. Web fonts enable pages to use typography that is not already installed on the user's machine. This has previously been a thorn in the side of designers and developers for a very long time.

Many years ago, great debates raged about whether `font-family: Arial, Helvetica, sans-serif;` was correct. Arial and Helvetica are very similar, but include subtle differences that can cause not just visual discrepancies, but also issues with how content flows on-screen.

With the advent of web fonts, however, a new class of performance problems has arisen. Questions about the best way to load web fonts to ensure text is visible to the user as early as possible have arisen. Websites with poor performance often render "without text," when they use fonts that take a long time to load. This is especially painful on mobile connections, where latency may be high.


## Cleaving away legacy code

Before web fonts were available, web developers used a variety of techniques to deliver styled text to users. Unfortunately, many of these techniques degraded performance in substantial ways. Some of these approaches include:

- Embedding styled text in images
- Using libraries like SiFR or Cufon to render fonts dynamically
- Using Flash

Ultimately, web fonts offer the greatest performance in almost all cases. True web fonts also grant the benefit of accessibility for disabled users (screen readers cannot understand images). Content can also be selected and copied, and there is no library needed to render the fonts. Web fonts also allow content to be responsive, as they are not a fixed size like images are.

You should remove any legacy font rendering technologies from your stack.


## Reducing font file size

Web fonts have a major downside: they add a substantial amount of payload to a page. Downloading a font also blocks text from rendering, since the browser has no way of knowing what the letters look like with the font. As such, minimizing the amount of content sent to the browser is critical to ensuring a good user experience when using web fonts.


### Minimum character sets

The first and simplest way to reduce the payload of web fonts is to remove characters that won't be used on the website. If your pages are exclusively English text, non-Latin characters can easily be removed from the font file.

Most web font sites, like Google Fonts, allow you to choose the character sets to make available in the font file. Simply changing your configuration to load only the minimum character set required is an easy performance win.

Tools are also available online for manually editing the characters in a font file. You can use these to add or remove individual characters or sets of characters as needed.


### WOFF2

WOFF2 is the successor for the WOFF format. It was invented by Google, and offers better compression than the regular WOFF format. At the time of writing Firefox and Chrome both offer support for WOFF2.

*Everything Fonts* offers a tool for converting TTF files to WOFF2 format[^1]. You can take advantage of this to create your own WOFF2 files. Implementing these files in your CSS is straightforward:

```css
@font-face {
    font-family: "My Great Font";
    src: url('https://example.com/assets/fonts/myfont.woff2') format('woff2'),
         url('https://example.com/assets/fonts/myfont.woff') format('woff');
}
```

Just add your WOFF2 font as a format before your WOFF font.

Additionally, make sure your server serves the WOFF2 files with the correct MIME type. If you notice your browser downloading both the WOFF2 and WOFF files, it may be because the server is responding with a `Content-Type` of `text/plain`. Make sure your server serves `.woff2` files with the MIME type `application/font-woff2`. In nginx, you can use this configuration:

```
types {
    application/font-woff2  woff2;
}
```

In Apache httpd, you can use this configuration:

```
AddType  application/font-woff2  .woff2
```

[^1]: https://everythingfonts.com/ttf-to-woff2

### Remove legacy browser support

If your site does not support IE8 or below or Safari 7 or below, you can safely remove support for EOT, SVG, TTF, and OTF fonts from your CSS. Only WOFF and WOFF2 are necessary to work across all modern browsers going back to IE9. This will help reduce the number of assets that your site needs to serve, and helps reduce load during your deployment process.


## Load fonts from your own servers

Loading fonts through a service like Google Fonts can be very convenient, but it can also introduce performance problems. Simply using the `@import` rule provided by Google can result in a serious performance regressions:

- The `@import` rule won't be loaded until the CSS that it lives in is loaded
- The `@import` rule can result in a DNS lookup for the host that the font is served from
- The CSS file loaded by the `@import` rule will block your page from rendering
- Once the CSS file loaded by the `@import` rule has loaded, it will still take time for the font file to load and for your text to be rendered.

If you have the rights to host a font yourself, you should opt to do that. Host your fonts on your front-end server and reference them directly with an `@font-face` block in your CSS. This has a number of benefits:

- Your users will not need to make any extra DNS lookups
- If your server uses HTTP/2, no extra connections are necessary
- If your server does not use HTTP/2, it's likely that the font can use a pooled connection (from HTTP/1.1's `keep-alive` support)

Note that web fonts require CORS to be set up to load fonts cross-origin. If you use a CDN, you may need to configure it to serve your web fonts with the appropriate headers. Alternatively, you can serve your web fonts from the same hostname as the pages themselves, which may provide satisfactory performance if the font files are small.


## Loading web fonts early

As mentioned above, a serious problem with web fonts is the time it takes for them to load. Slow-loading fonts can cause text not to render on the page.

The easiest way to begin loading fonts early is to use an inline style. This allows the font files to begin downloading as soon as the server responds with the page rather than waiting until the first CSS file has finished loading.


### Preload links

If you cannot use inline styles (e.g., you have a content security policy that disallows them), you can use a `<link>` tag to begin loading the files early.

```html
<link rel="preload" href="/path/to/my/font.woff2" as="font">
<link rel="preload" href="/path/to/my/font.woff" as="font">
```

Place these links immediately before your any CSS on your page. This will instruct the browser to begin downloading the files as soon as possible.

Alternatively, you can do the same thing with HTTP headers. Configure your server to respond with the following HTTP headers:

```
Link: </path/to/my/font.woff2>; rel=reload; as=font
Link: </path/to/my/font.woff>; rel=reload; as=font
```

Note that if you are loading fonts cross-origin, you'll need to add the `crossorigin` attribute to your `<link>` tags, and the `; crossorigin` suffix to the HTTP headers.


### HTTP/2 Server Push

If you're using a server that is compatible with HTTP/2, you can instruct it to preemptively serve the font files to the user using the "server push" feature. At the time of writing, this feature is not well-supported by most open source server options, so implementing server push for font files is left as an exercise to the reader.


### Service Workers

At the time of writing, service workers are not well-supported by any modern browser. However, they theoretically offer a good way of loading fonts in the browser. The service worker could be configured to preemptively download the font file or fetch it from the browser cache if needed.

If your application is already using service workers, you should consider how they could be leveraged to load fonts more efficiently.
