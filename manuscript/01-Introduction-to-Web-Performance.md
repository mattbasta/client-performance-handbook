# Introduction to Web Performance

The performance of applications on the web historically was simple: servers received requests, performed some trivial processing, and returned a response. Servers are easy to measure and easy to fix. Engineers of the era didn't know how good they had it: performance problems could be generally traced down to an inefficient algorithm, a lack of resources, a hardware problem, or something that could be addressed with a phone call to a third party.

In the mid-2000s, the "web 2.0" movement made this more complicated. JavaScript--the language once known for enabling mouse trails and falling animated GIFs of snow--was replacing what was traditionally done with Flash. Pages became interactive, no longer requiring a full page reload to perform a single function. As the web exploded with new applications hosting new types of content that behaved in new and interesting ways, a whole new class of never-before-experienced performance issues arose.

Raising the baseline performance of the web wasn't easy, and there's a lot of very hard problems that are involved. Some solutions to those problems, like JIT-compiled JavaScript and hardware acceleration, were developed by browser vendors. Google, Mozilla, and Apple have spent enormous amounts of money building solutions that improve the performance of everyday browsing experiences.

Many of the problems, though, are still unsolved. Many have no universal best solution. For instance, websites that want to include high-resolution images for users with high-DPI screens don't want to obliterate the experience that users on low-throughput connections have. As a developer, finding the balance between a fast product and the product you want to deliver to your users is essential.

If you are reading this book with the intention of identifying solutions to problems that you are already experiencing with a product, I'd suggest flipping ahead to the chapter "Identifying Performance Issues".


> There is no place like home. Unless you don't like your home, in which case there is no place like your neighbor's home.


## Performance is about compromises

Performance is a subset of the larger concept of Quality. A product that is high-quality delights its users. A product that is fast also delights its users. Oftentimes, improving the quality of a product has a positive impact on performance, and vise versa. I find it interesting that many software companies have teams dedicated to improving performance of their product, but few have teams dedicated to improving overall quality of the product.

Quality comes at a cost, though. From a design perspective, adding a subtle radial gradient to an interface may be an enormous improvement to the product. The underlying code to add the gradient, though, may not be quite as friendly. For example, if the gradient is added with CSS, the effect may cause massive performance issues for users' computers, especially when scrolling. Instead of implementing the gradient in CSS, it may be necessary to implement it with an image. Of course, loading an extra image incurs its own overhead, and has a much more significant impact on page load times.

Most changes related to performance have similar compromises: parallelizing asset loading may decrease load times for users on high-bandwidth connections, but have the opposite effect for users on low-bandwidth connections. Adding more aggressive minification to JavaScript may significantly decrease the size of a page's assets, but make it virtually impossible to debug issues that arise in production.

Building performant applications is not hard, but making them work the way you'd love for them to work *is* hard. You want to deliver lots of features for your users, but you don't want to increase the payload of your assets. You want to support users that aren't on the absolute latest versions of their respective browsers, but you don't want to ignore new and powerful technologies provided by the latest browsers.

The key to making a successful effort in improving web performance is understanding the constraints of the problems that need to be solved and ensuring that the downsides of the changes that you choose to make are acceptable trade-offs.


> A man that expects no trade-offs will quickly misplace his car's stereo.


## The Value of Performance

As an engineer, it's often easy to forget that despite one's best efforts, humans can't work on multiple things simultaneously. Managers, however, find it quite easy to remember that on the engineer's behalf. When a performance issue is identified, it's simply too easy for many folks--especially in corporate culture--to write off performance as a secondary or low-priority goal.

> If it ain't broke, don't fix it.

Some performance issues are easy to justify fixing: as the amount of data in the system increases, the system gets slower. This is a scale issue: something that gets worse with no upper ceiling. The justification for issues like this usually sound like, "If we don't fix this slowdown, you won't be able to log in by this time next week."

Other performance issues are more nuanced, and usually more abundant. A page's load might only take 2s, but only 0.8s are spent loading the assets used on the page. What's happening for the remainder of the page load? The browser might be executing very slow JavaScript, layout thrashing, performing garbage collection, or more. Fixing any single one of those problems could--depending on the complexity of the problem and the code involved--take an expert a day or more to fix, and the performance gains would likely be measurable in hundreds of milliseconds.

For the average developer, divine intervention is in short supply and it would take nothing less to convince a manager that a couple hundred milliseconds are worth more than a few hours of time.

> It takes time to save time.

So what is the justification for improving performance?

1. **At least some of your users are frustrated.** Most upper management would be appalled to see the 90th percentile performance numbers for their user base. An unoptimized site with international users could expect ten second load times and worse. If you wouldn't use your own product at the 90th percentile, neither will 10% or more of your users.
2. **The users that have the best performance make up only a small portion of your user base.** It's hard to identify performance problems when you're attached to a fiber optic internet connection connected to a CAT-6 wire in an office building. The users on Edge connections or accessing your content from the other side of the world are equally plentiful, in many cases, to the ones down the street from you. It's not hard for them to find a faster competitor.
3. **Everyone else did the research and the results are in: fast sites are more engaging.** Amazon reports "every 100ms delay costs 1% of sales." When at Google, Marissa Mayer reported that a 500ms delay in load times resulted in 20% fewer searches. Yahoo! engineer Stoyan Stefanov reported that a 400ms delay resulted in 5-9% less traffic. Bing engineers saw an average of 2.5% loss in revenue per visitor with a 1000ms delay.
4. **Poor performance is an indicator of tech debt.** Pages don't get slow on their own, and poor performance usually indicates that something is broken, poorly architected, or stretched beyond its original design. Ignoring tech debt is allowing your product to rot from the inside out.
5. **Each of your users has to load your site.** Your site's initial load is the first experience any user has with your website. If that experience is poor, your users already have a bad first impression before they even get to the rest of the content.

On a more tangible level, improved performance is often a sign of better resource utilization. The more efficiently an application can produce a response, the better the underlying resources are being used. For example:

- Decreased payload in a web application means smaller load times, but it also means decreased bandwidth usage. Depending on hosting costs, saving bytes can save quite a bit of money.
- Increased server efficiency means fewer servers are needed. When Facebook switched from Zend PHP to HipHop (a tool built to compile PHP to native code), they were able to decrease the number of web servers they used by a factor of five[^hiphopdecreasedservers].
- Decreasing the number of requests that a client is required to make in order to load a page proportionally decreases the number of requests that the back-end server needs to handle. Decreased load on the server results in greater capacity.
- Simplifying and refactoring code has many benefits, including decreased size and increased performance. Clean and well-structured code also requires less engineering attention than gnarly spaghetti code, and increases an engineer's ability to diagnose and fix problems more quickly in the future.

[^hiphopdecreasedservers]: See *The HipHop compiler for PHP*, presented at OOPSLA 2012


## Categorizing Users

An often-overlooked aspect to web development is that not all users are created equal. Many variables play a role in performance, including bandwidth, latency, processing power, browser, payload (if different content is served to different users), whether the user has visited a page before, and much more. The sheer number of variables that can affect a page's performance makes grouping users by each of them independently intractable.

Instead, the performance data that you gather and aggregate needs to be considered carefully to properly cluster data into manageable chunks. By identifying performance shortcomings in your site (or potential performance shortcomings), the data that you have can help you get a better picture of the different scenarios that users are encountering that's compromising performance.

Analytics tools like Google Analytics can give a detailed picture, but pulling in more than two facets of data (or grouping data together) can be a challenge. Here's an example of three visits to Poor Michael's website as seen by Google Analytics:

- A user on Safari 5 from Australia visits Michael's homepage and three other pages
- A user on Chrome for Android from Phoenix, AZ visits Michael's product list page and one other page
- A user on Internet Explorer 11 from Nashville, TN visits a product page on Michael's site and leaves immediately

Singling out individual visitors isn't something that's possible most of the time, and this data--while somewhat useful--doesn't give a great picture of the experience that each user had.

These kinds of analytics tools tend to have little value for gathering performance-related data if you don't know what to look for or how to segment what you find. It can also be a challenge to automate these tools to provide scheduled reports or dashboards. This is understandable, as tools like GA are not designed to be used for performance analysis.

> Information overload can be more expensive than ignorance if the information isn't free.

For a performance engineer, the following would have been more useful:

- The first user visits on Safari 5.0.1 visits from Australia, but leaves without interacting with anything after ten seconds. This user's experience is the worst: they're using an old browser to visit Poor Michael's site, using a poor internet connection with very high latency. This version of Safari lacks many web technologies, including SPDY, which Michael's site uses to battle latency. Connections to Australia are typically poor due to a limited number of under-sea cables running to the continent and a restrictive number of options for ISPs. This user has a fifteen second page load, but the page was interactive after five.
- The second user visits on Chrome for Android from Phoenix, AZ and stays on the site long enough to visit a few pages. This user has an "average" experience: they're running a modern browser and probably have Chrome's "Reduce data usage" feature turned on, which is further improving their load times. Geographically, the user is located in a part of the United States with decent internet access. The device rendering the site is one of the biggest bottlenecks here: a three-year-old Android tablet. The device's limited CPU and GPU prevent the site from parsing, initializing, and rendering as fast as possible. This user has a six second page load, but the page was interactive after two.
- The third user is visiting on Internet Explorer 11 from Nashville, TN and spends the most time on Poor Michael's website. This user's experience is the best: the site loads quickly in the user's modern browser, and the user is geographically very close to the server hosting the content in Virginia. This user has a three second page load, but the page was interactive in less than one.

This information can be collected partially though Google Analytics, but libraries like Boomerang by Yahoo!, custom reporting tools, or even WebPageTest provide much more complete and actionable data.

Another ideal form for this data would look something like the following:

| Percentile | Ready Time (s) | Load Time (s) |
|------------|----------------|---------------|
| 10th       | 0.9            | 3.2           |
| 50th       | 2.2            | 6.4           |
| 90th       | 5.5            | 14.8          |


Percentile
: If you were to create a histogram of the load time for all of a page's visitors, each percentile represents a single point along that histogram. I.e.: the 50th percentile would represent the median value, the 10th percentile would represent the value 10% of the way across the histogram, etc.

Ready Time
: The time it took from the moment the user's browser started navigating to the moment the user's browser declared that the page is interactive. In most browsers, this is when `DOMContentLoaded` starts to fire or `document.readyState` changes to `"interactive"`.

Load Time
: The time it took from the start of navigation to the moment the user's browser has completely finished loading. In all browsers, this is the `load` event that fires on the `window` object.


Some things to notice:

- This information is for a single page only. Differences between pages can muddy the data. For sites with many pages, it is useful to group the data by pages that a user is likely to visit together. E.g.: Login page, dashboard, and search might be grouped together.
- The information is broken down by percentile. Grouping users by their total load time allows you to drill down on worthwhile performance improvements and prioritize the value of a change.
- The tenth percentile is included. Some would argue that including the 10th percentile is unnecessary: those users are having a good experience. This information is included, though, because a negative change in the 10th percentile's numbers usually indicates a regression that affects all users.
- Ready time is shown. This number is important because a regression can mean that a piece of code is taking too long to execute. It's possible for the ready time to get worse without affecting the load time, making the user need to wait longer to use the page and decreasing perceived performance.

Depending on where a site is hosted, it may also be useful to split these numbers by region. For example, you may wish to have a separate query for users in the United States, UK, and southeast Asia. The network properties--and in some cases, browser trends--in a particular region can have significant effects on the performance data that you collect.


## Prioritizing Performance Improvements

Now that you know what to look at and why you should be looking, let's talk about what to do when you've found something interesting.

It's easy to want to fix every problem that comes along, or to want to skip over hard problems and instead take care of some low-hanging fruit. How should performance issues be addressed? There are a number of factors to consider:

- **How many users are affected by the issue?** If the benefits of the change are limited, it may be more valuable to push it off and work on something more impactful. On the other hand, if the benefit is substantial and the change is easy to make, it may make sense to go ahead with it.
- **How much benefit is there?** A change that impacts a very large number of users but doesn't provide much benefit may not be worth as much as a change that provides more benefit to a smaller number of users. Making targeted fixes can provide much needed benefits that help to make sizable differences in your performance numbers.
- **Which users are affected?** Sometimes it's hard to determine where attention should be focused. On one hand, it's important to benefit the largest number of users with the biggest wins possible. However, it may be more fiscally responsible to make changes that benefit paid users before focusing on fixes that impact free users.
- **How much does the change cost?** Not all performance improvements are free. Adding a CDN, for instance, may significantly increase hosting costs. If the costs involved outweigh the benefits that the improvement will provide, it may be more worthwhile to look elsewhere.
- **What other improvements does the change unlock?** If making the change (regardless of how small it may be) unlocks the ability to make other worthwhile performance improvements, it may be a valuable change to make.

In general, you can model how worthwhile a change is using the following formula:

{$$}
w = \frac{N W}{D C}
{/$$}

N
: The number of users that will see an improvement

W
: The expected wins

D
: The difficulty, or amount of time involved in making the change

C
: The (fiscal) cost of the change


Obviously, the units of each variable are subjective. Measuring difficulty, for example, might be better done in man hours for one team and sprints for another. Expected wins might be measured in dollars, engagement, or milliseconds. The dimensional analysis required to make this approach useful may not be simple and will vary greatly between organizations.

A good project manager should use a similar approach to determine what areas of performance deserve the most attention. It is very hard to say no to a change that provides a seemingly substantial benefit, but prioritizing the most valuable changes over less valuable changes will ultimately provide the most effective means of addressing a broad range of performance challenges.
