# Performance is About Compromises

Performance is a subset of the larger concept of Quality. A product that is high-quality delights its users. A product that is fast also delights its users. Oftentimes, improving the quality of a product has a positive impact on performance, and vise versa. I find it interesting that many software companies have teams dedicated to improving performance of their product, but few have teams dedicated to improving overall quality of the product's user experience.

Quality unfortunately comes at a cost. From a design perspective, adding a subtle radial gradient to an interface may be an aesthetically pleasing touch. The underlying code to add the gradient, though, may not be quite as friendly: a large radial gradient may cause laggy scrolling and dropped frames, more commonly known as "jank." Instead of implementing the gradient in CSS, it may be instead necessary to implement it with an image. Of course, loading an extra image incurs its own overhead, and has a much more significant impact on page load times.

Most changes related to performance have similar compromises: parallelizing asset loading may decrease load times for users on high-bandwidth connections, but have the opposite effect for users on low-bandwidth connections. Adding more aggressive minification to JavaScript may significantly decrease the size of a page's assets, but make it virtually impossible to debug issues that arise in a production environment.

Building performant applications is not hard, but doing so while also providing the best possible user experience is incredibly difficult to accomplish. You want to deliver lots of features for your users, but you don't want to increase the payload of your assets. You want to support users that aren't on the absolute latest versions of their respective browsers, but you don't want to ignore new and powerful technologies provided by the latest browsers.

The key to successfully improving web performance is understanding the constraints of the problems that need to be solved and ensuring that the downsides of the changes that you choose to make are acceptable trade-offs.
