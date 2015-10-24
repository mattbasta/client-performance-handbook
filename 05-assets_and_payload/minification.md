# Minification

In the book *High Performance Web Sites* (Chapter 10), Steve Souders differentiates between minification and obfuscation. Today, the two concepts have largely merged into one: all proper minification tools tend to perform many complex optimizations, not just basic whitespace removal.

Ten years ago, tools like JSMin and YUI's CSS compressor were state-of-the-art. Today, the tools that have replaced them are more akin to compilers. UglifyJS and Google Closure Compiler have taken JavaScript optimization to a whole new level and made deep, complex minification techniques both safe and reliable as well as easy-to-use for the average web developer.

There are two techniques used in minification:

- Manipulations that reduce the size of the file being minified
- Manipulations that improve the file's ability to be compressed

When most people think of minification, they think of the first technique: reducing the overall size of the file. This may mean removing unused code, removing whitespace, replacing one string with a shorter version, etc. The second technique is equally important. Many optimizations that fall into that category do not decrease the size of the file at all (and in some cases, may even make it bigger). Instead, the optimizations improve the final compressed size of the file.

Gzip can consequently mask the gains that minification provides, and what is best is sometimes difficult to identify. For example, a file containing random noise will Gzip very poorly, but a significantly larger file containing human-readable text will Gzip very well (and can certainly become even smaller than the compressed noise). As such, some removal or replacement operations may be skipped in favor of operations intended to improve compression.

For example, UglifyJS2 has a `sort` option that uses smaller variable names for more commonly used variables. While this decreases overall file size, it tends to *increase* the compressed size.