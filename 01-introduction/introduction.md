# Introduction

The performance of applications on the web historically was simple: servers received requests, performed some trivial processing, and returned a response. Servers are easy to measure and easy to fix. Engineers of the era didn't know how good they had it: performance problems could be generally traced down to an inefficient algorithm, a lack of resources, a hardware problem, or something that could be addressed with a phone call to a third party.

In the mid-2000s, the "web 2.0" movement made this more complicated. JavaScript--the language once known for enabling mouse trails and falling animated GIFs of snow--was replacing what was traditionally done with Flash. Pages became interactive, no longer requiring a full page reload to perform a single function. As the web exploded with new applications hosting new types of content that behaved in new and interesting ways, a whole new class of never-before-experienced performance issues arose.

Raising the baseline performance of the web wasn't easy, and there's a lot of very hard problems that are involved. Some solutions to those problems, like JIT-compiled JavaScript and hardware acceleration, were developed by browser vendors. Google, Mozilla, and Apple have spent enormous amounts of money building solutions that improve the performance of everyday browsing experiences.

Many of the problems, though, are still unsolved. Many have no universal best solution. For instance, websites that want to include high-resolution images for users with high-DPI screens don't want to obliterate the experience that users on low-throughput connections have. As a developer, finding the balance between a fast product and the product you want to deliver to your users is essential.

If you are reading this book with the intention of identifying solutions to problems that you are already experiencing with a product, I'd suggest flipping ahead to the chapter "Identifying Performance Issues".