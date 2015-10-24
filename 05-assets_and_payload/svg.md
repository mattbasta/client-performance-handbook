# SVG

The only vector graphics format that works natively in all browsers is SVG. SVGs are generally small, and can be compressed well in some cases. Unfortunately, visually complex images that include irregular shapes can bloat the size of the SVG. For instance, consider the following path:

```xml
<path d="M5.9482467,13.4977018 L-2.59854728,6.47190027 C-3.38658022,5.83462046
-4.65285911,5.84398612 -5.42686033,6.49281906 C-6.20086155,7.141652
-6.1894866,8.18425239 -5.40145365,8.8215322 L4.55314211,16.8717831
L5.94824679,18 L7.34955712,16.8770089 L17.3949616,8.82675797
C18.1864999,8.19242996 18.2036227,7.14988283 17.4332066,6.49816377
C16.6627904,5.84644472 15.3965762,5.83234649 14.605038,6.4666745
L5.9482467,13.4977018 Z M5.9482467,13.4977018" id="Path 5" fill="#C7C7C7"
transform="translate(6.000000, 12.000000) rotate(-270.000000)
translate(-6.000000, -12.000000)"></path>
```

Though this is a simple shape, it can't be described using nice, round numbers. Shaving decimal places from the numbers in an SVG can help to decrease its size significantly:

```xml
<path d="M5.948,13.497 L-2.598,6.471 C-3.386,5.834 -4.652,5.843 -5.426,6.492
C-6.200,7.141 -6.189,8.184 -5.401,8.821 L4.553,16.871 L5.948,18 L7.349,16.877
L17.394,8.826 C18.186,8.192 18.203,7.149 17.433,6.498 C16.662,5.846
15.396,5.832 14.605,6.466 L5.948,13.497 Z M5.948,13.497" fill="#C7C7C7"
transform="translate(6, 12) rotate(-270) translate(-6, -12)"></path>
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

It is also worth noting that SVG has only been supported by Internet Explorer since version 9, and even then only minimally. Critical page elements should not be represented as SVGs if you require IE8 support. You can, however, add SVG support on the fly very simply:

```css
.ui-element {
  background-image: url(/images/raster-version.png);
}

@supports (clip-path: url(#1)) {
  .ui-element {
    background-image: url(/images/vector-version.svg);
  }
}
```

Only browsers that support the `clip-path`[^1] CSS declaration (and the CSS `@supports` block) will use the SVG version of the image. At the time of writing, this is Firefox and Chrome, but IE12 will almost surely support this. Unfortunately, Safari does not--perhaps ironically--support `@supports`, meaning desktop Safari and iOS users will not receive SVG images, either.

[^1]: `clip-path` is supported by all browsers that have basic SVG support.
