# Images on the Web

There are three viable image formats available on the web today:

<dl>
    <dt>JPEG</dt>
    <dd>The JPEG ("Joint Photographic Experts Group") image format is useful for storing photos and other graphics containing irregular or highly-textured content. JPEGs do not have an alpha channel.</dd>
    <dt>GIF</dt>
    <dd>Previously encumbered by patents, the controversy surrounding licensing on the GIF ("Graphics Interchange Format") format has ended. GIFs are commonly used to encode small images, or images with few colors. Graphics with fewer than 256 colors can be losslessly represented by GIF. Using extensions, GIFs can represent far more colors or include animations, though this has a serious negative impact on file size. GIFs can indicate a transparent color, though they do not have an alpha channel.</dd>
    <dt>PNG</dt>
    <dd>The PNG ("Portable Network Graphics") format was developed to circumvent the patent issues surrounding GIF. Unlike GIF, PNGs support true-color imaging, full alpha transparency, and gamma correction. PNGs can be compressed to eliminate some of those features. PNG does not support animation.</dd>
    <dt>SVG</dt>
    <dd>Unlike other formats in this list, SVG ("Scalable Vector Graphics") stores image data in vector representation. Line art, regular shapes, and other smooth graphics can be represented by SVG with very small file sizes. Image data is encoded as shapes in an XML format. SVG images can be zoomed and scaled with no loss in quality.</dd>
</dl>

Some formats, like BMP, used to be supported on the web but have since been made obsolete and removed.

Additionally, there are a number of formats that are only implemented by a subset of browsers:

<dl>
    <dt>APNG</dt>
    <dd>
        PNG does not natively support animations (as GIF does), so APNG ("Animated Portable Network Graphics") was developed to fill this need. APNG is supported in Firefox and Safari, though a Chrome extension is available.
        <br><br>
        In recent years, developers have opted to use MP4 and WebM videos in lieu of animated graphics formats like GIF and APNG. Proper video file formats are not only better-supported, but offer much better compression and can optionally include audio. Because of this, APNG has limited usefulness.</dd>
    <dt>JPEG XR</dt>
    <dd>JPEG XR ("Extended Range", also known as "HD Photo") was developed by Microsoft. It aims to provide better quality, lossless compression, separate encoding for parts of an image, transparency, and more. IE9 and up include for support for JPEG XR, however the benefits are limited <a href="http://people.mozilla.org/~josh/lossy_compressed_image_study_october_2013/">when compared to better-supported encodings like WebP</a>.</dd>
    <dt>TIFF</dt>
    <dd>
        TIFF ("Tagged Image File Format") is one of the first image formats supported on the web, though it currently has little to no support in any modern browsers. Many users can view TIFF files, though, through the use of third party plugins.
        <br><br>
        TIFF is not compressed by default, meaning images encoded with it are very large. However, they do offer a plethora of features that are not available in any other format, such as layers and sub-files. TIFF files are comparable in some ways to Photoshop's PSD format. Currently, TIFF is available to some Internet Explorer and Safari users only. Mozilla has publicly stated that it has no plans to add support.</dd>
    <dt>WebP</dt>
    <dd>
        WebP is a very modern image format, designed to include the features of PNG and GIF. At the same time, it provides excellent compression of photographs (like JPEG) with little perceivable quality loss. Originally criticized for lacking certain features, the new versions of WebP have addressed all noted drawbacks.
        <br><br>
        Though WebP is only officially supported in Chrome, Firefox does include a WebP decoder, as it is built into the libwebm library that Firefox uses to decode WebM files. A lack of support is <a href="https://bugzil.la/856375">purely political</a>.
    </dd>
</dl>
