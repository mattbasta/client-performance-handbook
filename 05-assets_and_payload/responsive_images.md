# Responsive Images

A recent trend in web design is the use of high-resolution images to improve the appearance of pages on high-dpi (or "retina") displays. These images are usually saved at 300 dpi, though, and this can cause issues for users on slow connections (this is likely, as many users with high-dpi displays are using modern smartphones and tablets on mobile networks).

To begin with, realize that a 72dpi image that's enlarged to 300dpi is not going to look any sharper on a retina display than the 72 dpi original. An image will only be as high-quality as its original version.

Additionally, there's a fine balance between size and quality: rather than increasing the image's resolution, it may be more effective to simply increase the quality setting. A JPEG at 70% quality at 300dpi will look almost identical to a full-quality JPEG at 200DPI when scaled. Depending on the application encoding the image and the number of pixels, the smaller-but-higher-quality image may have a smaller file size.

Lastly, ensure you are using the appropriate encoding for the image format that you're using. Ensure that the image loads progressively (starts at a low quality and gets clearer) rather than sequentially (starts at the top and loads down). More information on encoding will be provided in a later section.


## Change the image via media queries in CSS

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


## The `<picture>` element

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


## Use vector graphics where possible

SVG is fairly well-supported across the board. Unlike raster graphics like PNG and JPEG, SVG images don't pixelate as they are scaled. On high-resolution displays, SVG images remain crisp and don't become fuzzy. Additionally, SVG files are usually much smaller than PNG or JPEG files, making them great candidates for interfaces.

Another vector options is web fonts. Fonts are encoded as vectors, and can be loaded using CSS's `@font-face` block. Most modern browsers support the WOFF format, but TTF or ODT fallbacks should be provided. As an added bonus, multiple small images can be encoded into a single font file (one image per glyph), reducing the number of connections needed when not running on HTTP2.

Beware, however, when using fonts: most pre-made icon fonts (like Font Awesome) include hundreds of icons. These can bloat the font file if they are unused, resulting in significant amounts of wasted bandwidth. Use a tool to remove unused glyphs from the font file before uploading it.
