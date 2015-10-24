# Spriting

Spriting is a common practice that involves combining multiple small images into a single larger image. The image is then clipped and positioned using CSS to show only the individual components in the areas that they're needed.

On many websites, there are often tiny icons, graphics, and other visual components that simply require a large number of network requests. Each asset requires a separate request to the server, and any network delays can cause one or more of these assets to make the page take much longer to load. Spriting combines a large number of these small images into a single, larger image that can be downloaded more efficiently.

It's often the case that sprites are PNG files, since PNGs are generally quite small and benefit from being combined into larger files. Some pages that contain small photographs or textured elements that benefit from JPEG compression can in fact use sprites, though quality will be decreased. This can be mitigated by ensuring that each image within a JPEG sprite has a height and width that is a multiple of 8 pixels. This prevents the images from bleeding into one another.

Sprites should always be created from the highest quality source files. Never use previously-encoded images as the source for sprites, as the spriting process will re-encode the image (like taking a photocopy of a photocopy).


## Online tools

There are some spriting tools available directly in your browser. The first is Spritegen[^1], which contains many features. Spritegen allows you to specify encoding settings, dimensions, and provide formatting for the resulting CSS.

Another great, dead-simple tool is Spritificator by Matt Claypotch[^2]. Spritificator combines images into a sprite and provides the CSS necessary to access the images from your page. For most use cases, this is all that's needed.

[^1]: http://spritegen.website-performance.org/
[^2]: http://potch.me/projects/spritificator/