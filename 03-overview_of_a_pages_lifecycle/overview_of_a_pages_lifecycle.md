# Overview of a Page's Lifecycle

When you load a web page in your browser, there's quite a lot of things that happen. In rich internet applications, browsers are doing an extraordinary amount of work. Some of these operations, such as network IO, can be done concurrently. Other operations, like constructing the DOM or performing certain operations in JavaScript, block the entire page load process.

This chapter discusses the different phases of a page's load. Some of the phases interact closely, like the HTTPS handshake and the initial request (where a HTTP/2 server might upgrade the connection). Other phases might not happen at all, like DNS lookups.

Use this chapter as a reference point for places to search for performance issues. Look at each phase of a page's load process individually and consider the impact of each one. Oftentimes you may find low-hanging fruit that can eliminate a problematic part of your page's lifecycle that affects the entire site.
