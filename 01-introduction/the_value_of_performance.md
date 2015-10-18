# The Value of Performance

Some performance issues are easy to justify fixing: as the amount of data in the system increases, the system gets slower. This is a scale issue and gets worse with no upper ceiling. It's hard to argue with a statement like, "If we don't fix this slowdown, you won't be able to log in by this time next week."

Other performance issues are more nuanced, and generally more abundant. A page's load time might be a clean two seconds, but only 0.8s are spent loading the assets used on the page. What's happening for the remainder of the page load time? The browser might be executing very slow JavaScript, layout thrashing, performing garbage collection, or more. Fixing any single one of those problems could--depending on the complexity of the problem and the code involved--take an expert a day or more to fix, and the performance gains would likely be measurable in (a measly) hundreds of milliseconds.

For the average developer, divine intervention is in short supply and it would take nothing less to convince a manager that mere hundreds of milliseconds are worth more than a few hours of time to eliminate.

So what is the justification for improving performance?

1. **Not all users are created equal.** Most upper management would be appalled to see the 90th percentile performance numbers for their user base. An unoptimized site with international users could expect ten second load times and worse. If you wouldn't use your own product at the 90th percentile, neither will 10% or more of your users.
2. **The users that have the best performance make up only a small portion of your user base.** It's hard to identify performance problems when you're attached to a fiber optic connection via gigabit Ethernet. The users on Edge connections or accessing your content from the other side of the world are equally plentiful, in many cases, to the ones down the street from you. It's also not hard for them to find a faster competitor.
3. **Everyone else did the research and the results are in: fast sites are more engaging.** Amazon reports "every 100ms delay costs 1% of sales." When at Google, Marissa Mayer reported that a 500ms delay in load times resulted in 20% fewer searches. Yahoo! engineer Stoyan Stefanov reported that a 400ms delay resulted in 5-9% less traffic. Bing engineers saw an average of 2.5% loss in revenue per visitor with a 1000ms delay.
4. **Poor performance is an indicator of tech debt.** Pages don't get slow on their own, and poor performance usually indicates that something is broken, poorly architected, or stretched beyond its original design. Ignoring tech debt is allowing your product to rot from the inside out. Even if performance issues are not fixed, knowing why they exist and working to mitigate or monitor them is still a win.
5. **Each of your users has to load your site.** Your site's initial load is the first experience any user has with your website. When a user visits for the first time (or with a cold cache), they experience the worst performance properties of your website. If that experience is poor, your users already have a bad first impression before they even get to the rest of the content.

On a more tangible level, improved performance is often a sign of better resource utilization. The more efficiently an application can produce a response, the better the underlying resources are being used. For example:

- Decreased payload in a web application means smaller load times, but it also means decreased bandwidth usage. Depending on hosting costs, saving bytes can save a notable amount of money.
- Increased server efficiency means fewer servers are needed. When Facebook switched from Zend PHP to HipHop (a tool built to compile PHP to native code), they decreased the number of web servers they used by a factor of five[^1].
- The fewer requests that a client needs to make in order to load a page proportionally decreases the number of requests that the back-end server needs to handle. Decreased load on the server results in greater capacity.
- Simplifying and refactoring code has copious benefits, including decreased size and increased performance. Clean code requires less attention than gnarly spaghetti code, and improves an engineer's ability to fix problems faster.

[^1]: See *The HipHop compiler for PHP*, presented at OOPSLA 2012

