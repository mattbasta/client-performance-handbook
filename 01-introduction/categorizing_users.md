# Categorizing Users

An often-overlooked aspect to web development is that not all users are created equal. Many variables play a role in performance, including bandwidth, latency, processing power, browser, payload (if different content is served to different users), whether the user has visited a page before, and much more. The sheer number of variables that can affect a page's performance makes grouping users by each of them independently intractable.

Instead, the performance data that you gather and aggregate needs to be considered carefully to properly cluster data into manageable chunks. By identifying performance shortcomings in your site (or potential performance shortcomings), the data that you have can help you get a better picture of the different scenarios that users are encountering that's compromising performance.

Analytics tools like Google Analytics can give a detailed picture, but pulling in more than two facets of data (or grouping data together) can be a challenge. Here's an example of three visits to Poor Michael's website as seen by Google Analytics:

- A user on Safari 5 from Australia visits Michael's homepage and three other pages
- A user on Chrome for Android from Phoenix, AZ visits Michael's product list page and one other page
- A user on Internet Explorer 11 from Nashville, TN visits a product page on Michael's site and leaves immediately

Singling out individual visitors isn't something that's possible most of the time, and this data--while somewhat useful--doesn't give a great picture of the experience that each user had.

These kinds of analytics tools tend to have little value for gathering performance-related data if you don't know what to look for or how to segment what you find. It can also be a challenge to automate these tools to provide scheduled reports or dashboards. This is understandable, as tools like GA are not designed to be used for performance analysis.

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
