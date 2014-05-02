# Introduction to Web Performance

The performance of applications on the web historically was simple: servers received requests, performed some trivial processing, and returned a response. Servers are easy to measure and easy to fix. Engineers of the era didn't know how good they had it: performance problems could be generally traced down to an inefficient algorithm, a lack of resources, a hardware problem, or something that could be addressed with a phone call to a third party.

In the mid-2000s, the "web 2.0" movement made this more complicated. JavaScript--the language once known for enabling mouse trails and falling animated GIFs of snow--was replacing what was traditionally done with Flash. Pages became interactive, no longer requiring a full page reload to perform a single function. As the web exploded with new applications hosting new types of content that behaved in new and interesting ways, a whole new class of never-before-experienced performance issues arose.

Raising the baseline performance of the web wasn't easy, and there's a lot of very hard problems that are involved. Some solutions to those problems, like JIT-compiled JavaScript and hardware acceleration, were developed by browser vendors. Google, Mozilla, and Apple have spent enormous amounts of money building solutions that improve the performance of everyday browsing experiences.

Many of the problems, though, are still unsolved. Many have no universal best solution. For instance, websites that want to include high-resolution images for users with high-DPI screens don't want to obliterate the experience that users on low-throughput connections have. As a developer, finding the balance between a fast product and the product you want to deliver to your users is essential.

If you are reading this book with the intention of identifying solutions to problems that you are already experiencing with a product, I'd suggest flipping ahead to the chapter "Identifying Performance Issues".


    There is no place like home. Unless you don't like your home, in which case there is no place like your neighbor's home.


## Performance is about compromises

Performance is a subset of the larger concept of Quality. A product that is high-quality delights its users. A product that is fast also delights its users. Oftentimes, improving the quality of a product has a positive impact on performance, and vise versa. I find it interesting that many software companies have teams dedicated to improving performance of their product, but few have teams dedicated to improving overall quality of the product.

Quality comes at a cost, though. From a design perspective, adding a subtle radial gradient to an interface may be an enormous improvement to the product. The underlying code to add the gradient, though, may not be quite as friendly. For example, if the gradient is added with CSS, the effect may cause massive performance issues for users' computers, especially when scrolling. Instead of implementing the gradient in CSS, it may be necessary to implement it with an image. Of course, loading an extra image incurs its own overhead, and has a much more significant impact on page load times.

Most changes related to performance have similar compromises: parallelizing asset loading may decrease load times for users on high-bandwidth connections, but have the opposite effect for users on low-bandwidth connections. Adding more aggressive minification to JavaScript may significantly decrease the size of a page's assets, but make it virtually impossible to debug issues that arise in production.

Building performant applications is not hard, but making them work the way you'd love for them to work *is* hard. You want to deliver lots of features for your users, but you don't want to increase the payload of your assets. You want to support users that aren't on the absolute latest versions of their respective browsers, but you don't want to ignore new and powerful technologies provided by the latest browsers.

The key to making a successful effort in improving web performance is understanding the constraints of the problems that need to be solved and ensuring that the downsides of the changes that you choose to make are acceptable trade-offs.


    A man that expects no trade-offs will quickly misplace his car's stereo.


## The Value of Performance

As an engineer, it's often easy to forget that despite one's best efforts, humans can't work on multiple things simultaneously. Managers, however, find it quite easy to remember that on the engineer's behalf. When a performance issue is identified, it's simply too easy for many folks--especially in corporate culture--to write off performance as a secondary or low-priority goal.

    If it ain't broke, don't fix it.

Some performance issues are easy to justify fixing: as the amount of data in the system increases, the system gets slower. This is a scale issue: something that gets worse with no upper ceiling. The justification for issues like this usually sound like, "If we don't fix this slowdown, you won't be able to log in by this time next week."

Other performance issues are more nuanced, and usually more abundant. A page's load might only take 2s, but only 0.8s are spent loading the assets used on the page. What's happening for the remainder of the page load? The browser might be executing very slow JavaScript, layout thrashing, performing garbage collection, or more. Fixing any single one of those problems could--depending on the complexity of the problem and the code involved--take an expert a day or more to fix, and the performance gains would likely be measurable in hundreds of milliseconds.

For the average developer, divine intervention is in short supply and it would take nothing less to convince a manager that a couple hundred milliseconds are worth more than a few hours of time.

    It takes time to save time.

So what is the justification for improving performance?

1. **At least some of your users are frustrated.** Most upper management would be appalled to see the 90th percentile performance numbers for their user base. An unoptimized site with international users could expect ten second load times and worse. If you wouldn't use your own product at the 90th percentile, neither will 10% or more of your users.
2. **The users that have the best performance make up only a small portion of your user base.** It's hard to identify performance problems when you're attached to a fiber optic internet connection connected to a CAT-6 wire in an office building. The users on Edge connections or accessing your content from the other side of the world are equally plentiful, in many cases, to the ones down the street from you. It's not hard for them to find a faster competitor.
3. **Everyone else did the research and the results are in: fast sites are more engaging.** Amazon reports "every 100ms delay costs 1% of sales." When at Google, Marissa Mayer reported that a 500ms delay in load times resulted in 20% fewer searches. Yahoo! engineer Stoyan Stefanov reported that a 400ms delay resulted in 5-9% less traffic.
4. **Poor performance is an indicator of tech debt.** Pages don't get slow on their own, and poor performance usually indicates that something is broken, poorly architected, or stretched beyond its original design. Ignoring tech debt is allowing your product to rot from the inside out.
5. **Each of your users has to load your site.** Your site's initial load is the first experience any user has with your website. If that experience is poor, your users already have a bad first impression before they even get to the rest of the content.
