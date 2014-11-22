# Client-Side Performance Almanac

A web developer's resource for all things performance-related on the web, with
stats, stories, and proverbs.

by [Matt Basta](http://mattbasta.com)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/3.0/"><img alt="Creative Commons License" style="border-width:0" src="http://i.creativecommons.org/l/by-nc-sa/3.0/88x31.png"></a>

Performance is one of the most wide-reaching yet difficult-to-address topics in
web development. Because there are so many aspects to the discipline, it's
nearly impossible to consider each one in a serial way. In addition,
performance myths and misinformation often lead well-intentioned engineers to
make poor choices.

The Client-Side Performance Almanac is a guide to help web developers deliver
better quality products to their clients and employers.


## Chapter Outline

1. Introduction to web performance
    1. Performance is about compromises
    2. The Value of Performance
    3. Categorizing Users
    4. Prioritizing Performance Improvements
2. Identifying Performance Issues
    1. The Navigation Timing API
    2. Timing Data Inside JavaScript
    3. Storing Timestamps at Important Page Events
    4. Make More Graphs
    5. Simplify
3. Overview of a page’s lifecycle
    1. Page Lifecycle and Performance
4. Connection and SSL
    1. DNS Lookups
    2. TCP Connections
    3. SSL
    4. CDNs
    5. HTTP2 and SPDY
5. Assets and Payload
    1. HTTP
    2. Gzip and Compression
    3. Images
    4. Fetching Resources In Advance
    5. One asset to rule them all?
        - Number of assets
        - Combining assets
        - Weighing the benefits of multiple files
6. Minification
    1. Markup
        - SVG
    2. CSS
    3. JS
7. JavaScript
    3. Browser events
    2. defer, async, both, neither
    1. Head or Body? Where the hell do I put my code
    3. Memory
        - Memory management
        - Garbage collection
        - Dirty, shameful tricks
    3. CPU-heavy code
        - Optimizing for the JIT
        - Web workers
        - Typed arrays
    3. Asm.js
8. Recommendations
9. Conclusion
    1. Notable Tickets
    2. Mentioned Tools
    3. Glossary
    4. Thanks, credits
    5. Proverbs


## Contributing

Please see the [contribution guidelines](CONTRIBUTING.md).
