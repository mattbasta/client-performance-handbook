# Encoding

Choosing the appropriate image format can sometimes be tricky. The wrong format can cause exceptional performance problems with a website. For example, encoding a photograph as a PNG file could triple the file size of the image.

The flow chart below should make it easier to choose which format to use.

![Image formats and their properties](images/image_format_choices.png)

Note that the rules above are not hard and fast. Sometimes, it may be more efficient to encode an image as PNG instead of JPEG if the image is sufficiently small, or if the number of colors in the image is lower than 256. Experiment to see which format provides the best file size for your needs.

## Image encoding and quality

For many common formats--in particular, PNG and JPEG--there are a number of settings that you can use to improve the size, perceived performance, and quality of the images that you generate.


### PNG

For small PNG files with few colors, 8-bit PNGs are the best option. In Adobe products, this option is listed as "PNG-8". Using PNG-8 instead of "true color" PNGs can save a significant amount of bandwidth, especially if the number of colors the image uses is minimized.

A little known fact is that the 8-bit PNGs do indeed support full alpha transparency. At the time of writing, Adobe Photoshop does not support these files, which has led to many developers believing such an encoding is not possible. Adobe Fireworks supports this option, though, as do other PNG utilities (and of course, browsers). A good PNG compression tool will attempt to use a PNG-8 encoding with alpha transparency.

Additionally, if you have the option, enable PNG interlacing for any file larger than a couple kilobytes. Interlacing enables the full PNG to begin rendering earlier in the loading process and gradually increase in quality as more data loads.


### JPEG

When using Adobe products, saving a JPEG under the "Save for Web" or "Optimize" menus provides an option labeled "Optimized". This should always be checked: it reduces file size at the cost of dropping support for some very old software.

"Progressive" JPEGs are another option you may receive (in Adobe products and elsewhere), which behaves similarly to the interlaced PNG option. Any large JPEGs should be encoded as progressive.

As far as quality is concerned, a quality setting of "High" (or about 60% on a 0 to 100 scale) is perhaps the highest that any image needs to be encoded as. Above that, the file size increases tremendously with little improvement in quality.
