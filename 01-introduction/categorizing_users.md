# Categorizing Users

As mentioned in the previous section, an often-overlooked aspect to web development is that not all users are created equal. Many variables play a role in performance, including bandwidth, latency, CPU power, browser type, payload, whether caches are warm, and much more. Users cannot simply be lumped into the same bucket and compared one-to-one.

Instead, the performance data that you gather and aggregate needs to be considered carefully to cluster data into manageable chunks. By identifying shortcomings (or potential shortcomings) in your site's performance, the data that you have can help you get a better picture of the scenarios that users are encountering.

Analytics tools like Google Analytics can give a detailed picture, but pulling in more than two facets of data (or grouping data together) can be a challenge. These tools are most useful for identifying when performance is impacting user experience. You can look at different facets of engagement, like the number of users that go through a checkout flow, and compare that against metrics like page load time.

Third party tools have an inherent flaw, though: they're not hosted on your own servers. For example, one common method for loading Google Analytics is with Google Tag Manager (GTM). GTM may take a couple of seconds to load, and then injects Google Analytics onto the page, which may take another second or two to load and send data back to Google. If your page already takes a few seconds to load as-is, your visitors may be leaving the page before Google Analytics ever has a chance to collect data! It's not uncommon for performance issues to come as a surprise in these cases, as the performance data in Google Analytics will be biased towards faster page loads.

These kinds of analytics tools tend to have little value for gathering hard performance-related data if you don't know what to look for or how to segment what you find. It can also be a challenge to automate these tools to provide scheduled reports or dashboards. This is understandable, as tools like Google Analytics are not designed to be used for performance analysis.

Some of the important information can be collected through tools like Google Analytics. Libraries like Boomerang by Yahoo!, custom reporting tools, or WebPageTest provide better, more actionable data.

Another ideal form for this data would look something like the following:

| Percentile | Ready Time (s) | Load Time (s) |
|------------|----------------|---------------|
| 10th       | 0.9            | 3.2           |
| 50th       | 2.2            | 6.4           |
| 90th       | 5.5            | 14.8          |


<dl>
    <dt>Percentile</dt>
    <dd>If you were to create a histogram of the load time for all of a page's visitors, each percentile represents a single point along that histogram. I.e.: the 50th percentile would represent the median value, the 10th percentile would represent the value 10% of the way across the histogram, etc.</dd>
    <dt>Ready Time</dt>
    <dd>The time it took from the moment the user's browser started navigating to the moment the user's browser declared that the page is interactive. In most browsers, this is when <code>DOMContentLoaded</code> finishes firing or <code>document.readyState</code> changes to <code>"interactive"</code></dd>
    <dt>Load Time</dt>
    <dd>The time it took from the start of navigation to the moment the user's browser has completely finished loading. In all browsers, this is the `load` event that fires on the `window` object.</dd>
<dl>

Some things to notice:

- **This information is for a single page.** Differences between pages can muddy the data. For sites with many pages, it is useful to group the data by pages that a user is likely to visit together. E.g.: pricing page and signup flow, login and dashboard, etc.
- **The information is broken down by percentile.** Grouping users by their total load time allows you to drill down on worthwhile performance improvements and prioritize the value of a change.
- **The tenth percentile is included.** Some would argue that including the 10th percentile is unnecessary: those users are already having a good experience. This information is included, though, because a negative change in the 10th percentile's numbers usually indicates a regression that affects all users.
- **Ready time is shown.** This number is important because a regression can mean that a piece of code is taking too long to execute. It's possible for the ready time to get worse without affecting the load time, making the user need to wait longer to use the page. This affects "perceived performance," which is discussed later.

Depending on where a site is hosted, it may also be useful to split these numbers by region. For example, you may wish to have a separate query for users in the United States, UK, and southeast Asia. The network properties--and in some cases, browser trends--in a particular region can have significant effects on the performance data that you collect.
