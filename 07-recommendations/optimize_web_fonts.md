# Optimize Web Fonts

Web fonts help a site achieve a look and feel that you can't achieve otherwise. The cost of using web fonts, though, can be high and degrade the performance of a page.

First, make sure the fonts that are loaded are actually being used. There are currently no good tools for automating this process, so a bit of manual effort will be required. Check for fonts that are loaded in the Network tab of your browser's developer tools. For each font, check your CSS to see that the font is referenced. Remove fonts that are altogether unused.

Second, remove glyphs from your fonts that you do not need. For example, if your website does not include localized text, you can probably remove non-Latin characters from your font. Most font services, like Google Web Fonts, will allow you to select which character sets to include in your font. If you host the font yourself, you can use a tool like Glyphs for OS X to remove glyphs from your font file.