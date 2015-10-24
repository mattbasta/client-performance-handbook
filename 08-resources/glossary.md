# Glossary

<dl>
    <dt>Compression</dt>
    <dd>The process of using an algorithm to reduce the size of a piece of data. Compression works by reducing redundant content within the data, or data that does not significantly affect quality.</dd>
    <dt>Encoding</dt>
    <dd>An encoding is a way of storing a piece of content. Image data, for example, is encoded into a format like JPEG, PNG, or GIF when stored in a file. The encoding may include features like compression, alpha transparency (for image), or metadata storage.</dd>
    <dt>Lossless</dt>
    <dd>A type of compression algorithm that does not reduce the quality of the data being compressed. A piece of data will remain unchanged if it is losslessly compressed and uncompressed.</dd>
    <dt>Lossy</dt>
    <dd>A type of compression algorithm that does not preserve the original data exactly. For example, a lossy image compression algorithm may remove certain fine details from the image.</dd>
    <dt>Minification</dt>
    <dd>The process of automatically rewriting code into a form that is smaller. This term may also be used to describe certain compiler-like optimizations to improve performance or remove unused code.</dd>
</dl>


## Images

<dl>
    <dt>Alpha Channel</dt>
    <dd>The alpha channel in an image is the color channel (see below) that represents the transparency of each pixel in the image. A pixel with an alpha channel value of zero is transparent. A pixel with an alpha channel value of 255 (or 100%) is opaque.</dd>
    <dt>Alpha Transparency</dt>
    <dd>The ability for a color or image to be slightly or fully transparent. An image with alpha transparency will blend with its background. The amount of transparency is defined by an image's alpha channel.</dd>
    <dt>Color Channel</dt>
    <dd>A color channel is a color value for each pixel in an image. When an image is encoded with RGB, it has a red, green, and blue color channel. Each channel has a single number for each pixel that represents the intensity of that color at the particular pixel. For instance, a "white" pixel would have its red, green, and blue color channels each set to 255 (or 100%). A firetruck red pixel would have its red channel set to 255, while the blue and green channels would be set to zero.</dd>
</dl>


## CSS

<dl>
    <dt>Declaration</dt>
    <dd>A declaration is an item within a rule set that applies a given property to the set of matched elements. E.g.: <code>color: red;</code></dd>
    <dt>Rule</dt>
    <dd>A rule is another name for a "declaration."</dd>
    <dt>Rule Set</dt>
    <dd>A rule set is a selector and zero or more declarations wrapped in curly braces. E.g., <code>.myElement {font-weight: bold;}</code></dd>
    <dt>Selector</dt>
    <dd>A selector is the string which describes which elements a rule set applies to. E.g., <code>#myElement .withAClass p</code></dd>
</dl>


## JavaScript

<dl>
    <dt>Closure</dt>
    <dd>A closure is a first-class function that references variables from another function after the second function has finished executing. These references are made with lexical scope.</dd>
    <dt>Event</dt>
    <dd>An object representing something that happened in the browser and the triggering of JavaScript code that has been put in place to accept it.</dd>
    <dt>JIT Compilation</dt>
    <dd>JIT compilers, or Just-In-Time compilers, are a cross between interpreters and static compilers. Code running in a JIT-enabled interpreter will run slowly as the interpreter finds out how the code works. Then, the JIT compiler will replace the slow code with fast, optimized code.</dd>
    <dt>Lexical Scope</dt>
    <dd>Lexical scoping is a feature of JavaScript that allows a function to access the variables of the function that it is running within. If one function is inside another function, the inner function can access and manipulate the outer function's variables.</dd>
    <dt>Run to Completion</dt>
    <dd>The process of triggering a JavaScript function, then waiting for all subsequent synchronous processing to complete.</dd>
</dl>