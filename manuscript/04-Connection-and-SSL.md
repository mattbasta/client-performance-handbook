# Connection and SSL

## HTTP2 and SPDY

SPDY is a very new and quite powerful protocol developed by Google for improving web performance. It functions as a substitute for HTTP at the protocol level: requests are sent in a way that's very similar to traditional HTTP, and applications running on the web server are generally not even aware that the requests they are receiving were made over SPDY. In fact, neither the Chrome nor the Firefox developer tools distinguish requests made by SPDY from requests made over normal HTTP.

In general, there are virtually no downsides to using SPDY over HTTP.

HTTP2 is the latest version of the HTTP specification. HTTP2 is largely based on the work done as part of SPDY's development. In fact, the initial draft of HTTP2 was simply a direct copy of the SPDY specification. HTTP2 has since been codified into a full, stable specification. Further work on SPDY continues, using HTTP2 as a basis for SPDY4.

For the purposes of this section, HTTP2 will mean both HTTP2 and SPDY, though versions of SPDY newer than SPDY4 may behave differently.


### How does it work?

When a browser connects to a server and establishes a secure connection, the remote server uses a TLS extension known as *Next Protocol Negotiation* to signal to the client that it will use HTTP2 if the client is okay to use it. If the client can accept HTTP2, the HTTP2 connection is established.

Once the connection has been established, the client will make as many requests as it needs over the same connection. The server will then respond with each of the files multiplexed over the same connection. Each of the requests and responses compresses the headers using Gzip or DEFLATE. If the server can predict what the client is going to do next, it can preemptively send assets to the client before they have been requested.

- Single connection means one SSL handshake
- Compressed headers means far less information is sent over the wire (in both directions)
- Low overhead for requests means minified files don't need to be concatenated: they can be requested individually, improving caching and decreasing waste from downloading unnecessary files
- No need for domain sharding (in fact, domain sharding hurts HTTP2)
- HTTP2 connections stay open, preventing future requests from needing to open a new socket

Results posted in Google's SPDY whitepaper[^spdy_whitepaper] show performance improvements of 25% to 60%, though in practice improvements of between 5% to 15% can be expected[^spdy_improvement].

[^spdy_whitepaper]: http://www.chromium.org/spdy/spdy-whitepaper
[^spdy_improvement]: From *A comparison of SPDY and HTTP performance* by Microsoft Research


### Good god that's fast

You're not kidding.


### Which users benefit

All users in modern browsers benefit from HTTP2, but different users will benefit in different ways.

- **10th percentile users:** Users that already have very fast connections will see minor improvements in areas where large numbers of requests are being made. Improvements will also be seen when making subsequent requests, as a new connection and handshake don't need to be made.
- **50th percentile users:** These users will see a substantial benefit from the decreased need to make many connections to request many assets. As soon as the HTTP2 connection opens, the browser can make unlimited requests, not just one.
- **90th percentile users:** These users will see all of the benefits seen by the other users. Additionally, the 90th percentile will benefit from header compression. Headers are largely redundant between requests, and eliminating duplicate information shaves off a non-trivial number of TCP round trips.


### Downsides and Myths

SPDY is actually slower than plain-old HTTP
: False. There is virtually no real-world evidence suggesting that SPDY or HTTP2 is ineffective. A number of benchmarks have appeared that show SPDY to be slower across the board, though this can be attributed to a number of flaws in the benchmarks:

  - Testing in low-latency, high-bandwidth local connections rather than over the internet
  - Sharding SPDY connections across domains (as mentioned previously, domain sharding hurts SPDY performance)
  - Using a SPDY proxy rather than running a server that delivers content directly
  - Comparing benchmark data from a SPDY mirror of a website to the website itself rather than a HTTP(S) mirror of the website

  It *is* possible for non-SPDY requests to load faster in parallel than the same requests made over a SPDY connection, but only on very fast network connections with low latency. Even if such a situation were to occur, the difference between SPDY and non-SPDY load times would be trivial.

HTTP2 doesn't work on sites that pull in third-party content
: Partly false. HTTP2 doesn't work across domains unless the domains share the same IP address. Third party content doesn't (usually) live on the same address as first party content, meaning that HTTP2 connections are simply not used for third party content (unless of course the third party host uses HTTP2).

HTTP2 is hard to configure
: This is indeed something of a downside. For many developers, even setting up HTTPS properly and making sure it stays properly configured and up-to-date can be a real challenge. Implementing HTTP2 alongside that configuration doesn't make the challenge any easier.

HTTP2 isn't supported in IE
: This is somewhat true. Only version 3 of SPDY is supported by Internet Explorer 11, and even then only when running on Windows 8. This is because only Windows 8 supports the TLS extension (NPN) needed to signal that the client can use SPDY. At the time of writing, IE11 also supports HTTP2 when running on Windows 10 Technical Preview.


### QUIC

QUIC is another project by Google to eventually become the successor to SPDY. It is currently in a highly experimental state. Unlike SPDY, QUIC uses UDP rather than TCP, enabling content to be received in a truly asynchronous manner and eliminating blockages caused by a single slow packet. QUIC also seeks to decrease the number of round trips made to and from the server, and eliminate most--if not all--of the time required to establish a connection.

QUIC is currently implemented in Chrome and Opera and can be enabled by visiting `chrome://flags` and `opera://flags` respectively. Most Google properties support QUIC, and a reference implementation of the QUIC server has been published in the Chromium source code repository.


## HTTP Redirects

Redirects are common in most modern web applications. Here's a flow from a popular enterprise software application:

| Request | Response |
| -- | -- |
| GET `http://popularsoftware.biz` | `302 Found` redirect to `https://popularsoftware.biz` |
| GET `https://popularsoftware.biz` | `302 Found` redirect to `https://www.popularsoftware.biz` |
| GET `https://www.popularsoftware.biz` | `302 Found` redirect to `https://www.popularsoftware.biz/login` |

Many web applications frequently use redirects for the following:

- Redirecting the user to the HTTPS version of a website in lieu of HSTS.
- Redirecting the user to the `www` subdomain of the website.
- Redirecting the user from a homepage to a login page.
- Redirecting the user to the appropriate page when they arrive at the site.

In the example above, multiple redirects take place that ultimately take the user to a single place. This is very inefficient, however, for a few reasons:

- Switching from HTTP to HTTPS requires a new, secure connection to the server. HSTS could be used to help prevent the need for this redirect.
- Changing hostnames from `popularsoftware.biz` to `www.popularsoftware.biz` requires a new DNS lookup and also requires a new connection to the server.
- Changing pages from `/` to `/login` is simply wasteful, requiring multiple full round-trips from the client to the server.

If the server software that performed the original redirect had knowledge about the change in domain and the login page, two full redirects could be avoided. The ideal chain of redirects would look like this:

| GET `http://popularsoftware.biz` | `302 Found` redirect to `https://www.popularsoftware.biz/login` |

A single redirect could accomplish the work of three. In short, be mindful of redirections in your web application to prevent latency and minimize delays during page loads.
