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
    1. Why perf is important
    3. Categorizing users
    3. Prioritizing performance improvements
2. Identifying performance issues
    1. Browser timing API
    2. Timing data inside compiled files
    3. Storing timestamps at important page events
    4. Make more graphs
    5. Simplify
        - Tech debt
        - Refactoring
        - Premature optimization
3. Overview of a page’s lifecycle
4. Connection and SSL
    1. Overview of TCP
    2. DNS Lookups
    3. TCP Connections
    4. CDNs
    5. SPDY
5. Assets and Payload
    1. HTTP
    2. Gzip and Compression
    3. Images
    4. Prefetching and Prerendering
    5. Minification
        1. Markup
            - SVG
        2. CSS
        3. JS
            - Overview of JS magnification
            - Best minifiers and how to use them
            - Worst minifiers and how not to use them
            - Tips for writing optimizable JavaScript
    6. One asset to rule them all?
        - Number of assets
        - Combining assets
        - Weighing the benefits of multiple files
6. JavaScript
    1. Head or Body? Where the hell do I put my code
    2. defer, async, both, neither
    3. Browser events
    3. Memory
        - Memory management
        - Garbage collection
        - Dirty, shameful tricks
    3. CPU-heavy code
        - Optimizing for the JIT
        - Web workers
        - Typed arrays
    3. Performance of APIs
    4. Asm.js
        - Asm.js is a red herring
        - Why you should mostly never use it
        - ...and what are the exceptions
    5. Frameworks and Performance
        - Aren’t frameworks necessary?
        - Aren’t frameworks unnecessary?
        - jQuery
        - Underscore
        - Alternatives
    6. Client-side Templating
        - What it is
        - How it works
        - Is it better?
        - Costs
7. Conclusion
    1. Proverbs
    2. Thanks, credits


## Contributing

Please see the [contribution guidelines](CONTRIBUTING.md).
