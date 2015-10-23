# Images on the Web

There are three viable image formats available on the web today:

JPEG
: The JPEG ("Joint Photographic Experts Group") image format is useful for storing photos and other graphics containing irregular or highly-textured content. JPEGs do not have an alpha channel.

GIF
: Previously encumbered by patents, the controversy surrounding licensing on the GIF ("Graphics Interchange Format") format has ended. GIFs are commonly used to encode small images, or images with few colors. Graphics with fewer than 256 colors can be losslessly represented by GIF. Using extensions, GIFs can represent far more colors or include animations, though this has a serious negative impact on file size. GIFs can indicate a transparent color, though they do not have an alpha channel.

PNG
: The PNG ("Portable Network Graphics") format was developed to circumvent the patent issues surrounding GIF. Unlike GIF, PNGs support true-color imaging, full alpha transparency, and gamma correction. PNGs can be compressed to eliminate some of those features. PNG does not support animation.

BMP
: One of the earliest formats, BMP ("Bitmap") encodes the color of each pixel individually. This results in very large file sizes, though images are encoded losslessly. BMP does not support an alpha channel. Despite its widespread support, the use of BMP files on the web is highly discouraged due to its extremely inefficient file format.

SVG
: Unlike other formats in this list, SVG ("Scalable Vector Graphics") stores image data in vector representation. Line art, regular shapes, and other smooth graphics can be represented by SVG with very small file sizes. Image data is encoded as shapes in an XML format. SVG images can be zoomed and scaled with no loss in quality.


Additionally, there are a number of formats that are only implemented by a subset of browsers:

JPEG XR
: Also known as "HD Photo," JPEG XR ("Extended Range") was developed by Microsoft to provide better compression, lossless compression support, separate encoding for different parts of an image, better color accuracy, alpha transparency, and more. JPEG XR is only supported in IE9 and up, however, and provides virtually no tangible benefits in comparison to WebP[^1].

TIFF
: TIFF ("Tagged Image File Format") is one of the first image formats supported on the web, though it currently has little to no support in any modern browsers. Many users can view TIFF files, though, through the use of third party plugins. While TIFF files are generally very large, they do offer a plethora of features that are not available in any other format, such as layers and sub-files. TIFF files are comparable in some ways to Photoshop's PSD format. Currently, TIFF is available to Internet Explorer and Safari users only; Mozilla has publicly stated that it has no plans to add support.

APNG
: PNG does not natively support animations (as GIF does), so APNG ("Animated Portable Network Graphics") was developed to fill this need. APNG is only supported in Firefox, though a Chrome extension is available.

WebP
: At the time of writing, this format is only supported in Chrome and Opera[^2]. WebP includes all features of PNG and GIF while providing exceptional compression of photographic images (like JPEG) with very little perceivable quality loss. Originally criticized for lacking animation support and other features, the latest versions of WebP has addressed all noted drawbacks.

[^1]: http://people.mozilla.org/~josh/lossy_compressed_image_study_october_2013/
[^2]: Though WebP is only officially supported in Chrome, Firefox does include a WebP decoder, as it is built into the libwebm library that Firefox uses to decode WebM files. A lack of support is purely political. See https://bugzil.la/856375
