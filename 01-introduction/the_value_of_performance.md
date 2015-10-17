# The Value of Performance

As an engineer, it's often easy to forget that despite one's best efforts, humans can't work on multiple things simultaneously. Managers, however, find it quite easy to remember that on the engineer's behalf. When a performance issue is identified, it's simply too easy for many folks--especially in corporate culture--to write off performance as a secondary or low-priority goal.

Some performance issues are easy to justify fixing: as the amount of data in the system increases, the system gets slower. This is a scale issue: something that gets worse with no upper ceiling. The justification for issues like this usually sound like, "If we don't fix this slowdown, you won't be able to log in by this time next week."

Other performance issues are more nuanced, and usually more abundant. A page's load might only take 2s, but only 0.8s are spent loading the assets used on the page. What's happening for the remainder of the page load? The browser might be executing very slow JavaScript, layout thrashing, performing garbage collection, or more. Fixing any single one of those problems could--depending on the complexity of the problem and the code involved--take an expert a day or more to fix, and the performance gains would likely be measurable in hundreds of milliseconds.

For the average developer, divine intervention is in short supply and it would take nothing less to convince a manager that a couple hundred milliseconds are worth more than a few hours of time.

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

