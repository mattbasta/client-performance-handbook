# Inlining and Bundling Assets

One of the most common questions that comes up with regard to assets is whether sites should have a single, large asset (like a master `include.js` file) or split the assets into smaller pieces.

There are a number of pros and cons for each approach:


## Single Large Asset

Pros:

- For small to medium-sized assets, this minimizes the number of connections that need to be made. High-latency users will benefit.
- Throughput tends to be higher for connections that send more data. Having one large asset helps to send more data at once.
- It is simpler to manage a single file than multiple files.
- For JavaScript, the `async` attribute can be used safely (albeit with few benefits).

Cons:

- For most sites, sharing content across pages means that assets will be downloaded that are not used or needed. Users that don't visit the other pages will have wasted that bandwidth.
- If one smaller part of the file changes, the whole file needs to be updated. For websites that are deployed frequently, this may mean that users rarely have a warm cache.
- For JavaScript files, the whole file must be loaded in order for its contents to start executing (versus executing smaller chunks of JavaScript as they arrive).


## Multiple Assets

Pros:

- Caching is improved, as small changes only invalidate the larger bundles that contain them. Individual assets can be grouped with other assets according to how often they change.
- For very large assets, downloading multiple files in parallel will almost always be faster than downloading one very very large file.

Cons:

- Multiple assets use multiple connections, which means unless the site uses domain sharding, it will reach the six connection limit fairly quickly.
- The issue of waste is not completely eliminated. If one page on a site uses all of two files, and another page on the site uses only part of each of those two files, it will either need to load both files (and waste the bandwidth for both parts that it doesn't use) or a third file will need to be created (preventing reuse and requiring the user to re-download information that's already in their cache).
- If multiple JavaScript files are used, it may not be possible to use the `async` attribute on the script tags without implementing a system for ensuring the files are initialized in the proper order.


### HTTP/2

When using HTTP/2 (or SPDY), some of the above points become invalid. Since a single connection is used and headers are compressed, all individual files can be requested with little overhead. Each file can be cached independently, saving significant amounts of bandwidth (all is related to caching are absent).

This may not be ideal, though, since older browsers (and Internet Explorer in most cases) cannot use HTTP/2. The application layer may also not know whether the client connected via HTTP/2 or not, making it difficult to decide whether to serve combined assets.

It is also worth noting that despite HTTP/2 eliminating the TCP and HTTPS overhead of establishing a new connection for each asset, requests for each asset do still need to be made. Requesting dozens, hundreds, or thousands of individual assets will add up regardless of whether HTTP/2 is used or not.


## Number of assets

Across a site, it may be smart to combine assets differently to promote cache reuse. For example, consider the following list of assets:

| Page      | Depends On    |
|-----------|---------------|
| Homepage  | jquery.js     |
| Homepage  | forms.js      |
| Homepage  | scroll.js     |
| Login     | jquery.js     |
| Login     | ajax.js       |
| Login     | forms.js      |
| Dashboard | jquery.js     |
| Dashboard | ajax.js       |
| Dashboard | charts.js     |
| Dashboard | fonts.js      |
| Dashboard | scroll.js     |
| Dashboard | urls.js       |

In this scenario, it might make more sense to create two bundles:

| Bundle 1  | Bundle 2  |
|-----------|-----------|
| jquery.js | ajax.js   |
| scroll.js | charts.js |
| forms.js  | fonts.js  |
|           | urls.js   |

used in the following way:

| Page      | Bundles    |
|-----------|------------|
| Homepage  | #1         |
| Login     | #1, #2     |
| Dashboard | #1, #2     |

In this case, the login page shares the same JS as the dashboard, even though most of the second bundle is unused. This doesn't matter, though, since virtually all users visiting the login page will eventually visit the dashboard (and their cache will be warm).

Figuring out how to perfectly distribute files and how many bundles of files to create isn't an exact science. Getting the split "close enough" will go a long way. After a certain amount of effort, the diminishing returns will yield little (if any) benefit.


## Combining Assets

One final technique that can potentially improve performance is combining multiple disparate assets together. For example:

- SVGs can be included as inline markup in HTML.
- Very small images can be converted to data URIs and included in CSS or in markup.
- Small CSS files can be inlined into a `<style>` tag.

In extreme cases (such as when dealing with users on poor network connections) it may be worthwhile to inline assets into the HTML. This may involve including CSS in `<style>` tags, JavaScript in `<script>` tags, etc. This is not recommended due to security, potential performance pitfalls (blocking the site's critical path), and a host of other reasons. However, for sites that target mainly high-latency, low-bandwidth connections, it may be necessary.
