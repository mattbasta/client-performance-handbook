# Introduction to Web Performance

The performance of applications on the web historically was simple: servers received requests, performed some trivial processing, and returned a response. Servers are easy to measure and easy to fix. Engineers of the era didn't know how good they had it: performance problems could be generally traced down to an inefficient algorithm, a lack of resources, a hardware problem, or something that could be addressed with a phone call to a third party.

In the mid-2000s, the "web 2.0" movement made this more complicated. JavaScript--the language once known for enabling mouse trails and falling animated GIFs of snow--was replacing what was traditionally done with Flash. As the web exploded with new applications hosting new types of content that behaved in new and interesting ways, a whole new class of never-before-experienced performance issues arose.

Making the web faster wasn't easy, and there's a lot of very hard problems that are involved. Some solutions to those problems, like JIT-compiled JavaScript and hardware acceleration, were developed by browser vendors. Google, Mozilla, and Apple have spent enormous amounts of money building solutions that improve the performance of everyday browsing experiences.

Many of the problems are still unsolved, or have no universal best solution. For instance, websites that want to include high-resolution images for users with high DPI screens don't want to obliterate the experience that users on low-throughput connections have. Finding the balance between a fast product and the product you want to deliver to your users is key.

If you are reading this book with the intention of identifying solutions to problems that you are already experiencing with a product, I'd suggest flipping ahead to the chapter "Identifying Performance Issues".


## Performance is about compromises

Performance is a subset of the larger concept of Quality. A product that is high-quality delights its users. A product that is fast also delights its users. Oftentimes, improving the quality of a product has a positive impact on performance, and vise versa. I find it interesting that many software companies have teams dedicated to improving performance of their product, but few have teams dedicated to improving overall quality of the product.

Quality comes at a cost, though. From a design perspective, adding a subtle radial gradient to an interface may be an enormous improvement to the product. The underlying code to add the gradient, though, may not be quite as friendly. For example, if the gradient is added with CSS, the effect may cause massive performance issues for users' computers, especially when scrolling. Instead of implementing the gradient in CSS, it may be necessary to implement it with an image. Of course, loading an extra image incurs its own overhead, and has a much more significant impact on page load times.

Most changes related to performance have similar compromises: parallelizing asset loading may decrease load times for users on high-bandwidth connections, but have the opposite effect for users on low-bandwidth connections. Adding more aggressive minification to JavaScript may significantly decrease the size of a page's assets, but make it virtually impossible to debug issues that arise in production.

Building performant applications is not hard, but making them work the way you'd love for them to work *is* hard. You want to deliver lots of features for your users, but you don't want to increase the payload of your assets. You want to support users that aren't on the absolute latest versions of their respective browsers, but you don't want to ignore new and powerful technologies provided by the latest browsers.

The key to making a successful effort in improving web performance is understanding the constraints of the problems that need to be solved and ensuring that the downsides of the changes that you choose to make are acceptable trade-offs.


## The Value of Performance


