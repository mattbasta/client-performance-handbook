# Optimize Web Fonts

Web fonts help a site achieve a look and feel that you can't achieve otherwise. The cost of using web fonts, though, can be high and degrade the performance of a page.

First, make sure the fonts that are loaded are actually being used. There are currently no good tools for automating this process, so a bit of manual effort will be required. Check for fonts that are loaded in the Network tab of your browser's developer tools. For each font, check your CSS to see that the font is referenced. Remove fonts that are altogether unused.

Second, remove glyphs from your fonts that you do not need. For example, if your website does not include localized text, you can probably remove non-Latin characters from your font. Most font services, like Google Web Fonts, will allow you to select which character sets to include in your font. If you host the font yourself, you can use a tool like Glyphs[^1] for OS X to remove glyphs from your font file.

[^1]: https://www.glyphsapp.com/

Next, make sure your server provides appropriate caching headers for font files. Fonts change infrequently, so you can set caching headers that prevent the file from being redownloaded. This can save a substantial amount of bandwidth, as well. For more information on caching headers, see the chapter "Caching Correctly."

Last, upgrade your WOFF files to WOFF2. WOFF2 is supported by most browsers, and your existing WOFF can be provided as a fallback. Many tools are available online for converting WOFF files to WOFF2 files, and almost all of them do a fine job.