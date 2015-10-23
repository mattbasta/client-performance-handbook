# HTTP/2

HTTP/2 is a very new and quite powerful protocol developed by Google for improving web performance. It functions as a substitute for HTTP at the protocol level: requests are sent in a way that's very similar to traditional HTTP, and applications running on the web server are generally not even aware that the requests they are receiving were made over HTTP/2. In fact, neither the Chrome nor the Firefox developer tools distinguish requests made by HTTP/2 from requests made over normal HTTP.

In general, there are virtually no major downsides to using HTTP/2 over HTTP.

HTTP/2 is the latest version of the HTTP specification. HTTP/2 is largely based on the work done as part of SPDY's development. In fact, the initial draft of HTTP/2 was a direct copy of the SPDY specification. HTTP/2 has since been codified into a full, stable specification. Further work on SPDY continues, using HTTP/2 as a basis for SPDY4.

For the purposes of this section, HTTP/2 will mean both HTTP/2 and SPDY, though versions of SPDY newer than SPDY4 may behave differently.


## How does it work?

When a browser connects to a server and establishes a secure connection, the remote server uses a TLS extension known as *Next Protocol Negotiation* to signal to the client that it will use HTTP/2 if the client is okay to use it. If the client can accept HTTP/2, the HTTP/2 connection is established.

Once the connection has been established, the client will make as many requests as it needs over the same connection. The server will then respond with each of the files multiplexed over the same connection. Each of the requests and responses compresses the headers using a specially-designed algorithm. If the server can predict what the client is going to do next, it can preemptively send assets to the client.

- A single connection means one HTTPS handshake for the entire site
- Compressed headers means far less information gets sent over the wire (in both directions)
- Low overhead for requests means minified files don't need to be concatenated. Assets can be requested individually, improving caching and decreasing waste from downloading unnecessary files
- No need for domain sharding (in fact, domain sharding hurts HTTP/2)
- HTTP/2 connections stay open, preventing future requests from needing to open a new socket

Results posted in Google's SPDY whitepaper[^1] show performance improvements of 25% to 60%. In practice, improvements of 5% to 15% can be expected[^2].

[^1]: http://www.chromium.org/spdy/spdy-whitepaper
[^2]: From *A comparison of SPDY and HTTP performance* by Microsoft Research


## Which users benefit

All users in modern browsers benefit from HTTP/2, but different users will benefit in different ways.

- **10th percentile users:** Users that already have very fast connections will see minor improvements in areas where large numbers of requests are being made. Improvements will also be seen when making subsequent requests, as a new connection and handshake don't need to be made.
- **50th percentile users:** These users will see a substantial benefit from the decreased need to make many connections to request many assets. As soon as the HTTP/2 connection opens, the browser can make unlimited requests, not just one.
- **90th percentile users:** These users will see all of the benefits seen by the other users. Additionally, the 90th percentile will benefit from header compression. Headers are largely redundant between requests, and eliminating duplicate information shaves off a non-trivial number of TCP round trips.


## Downsides and Myths

HTTP/2 is actually slower than plain-old HTTP/1.1
: False. There is virtually no real-world evidence suggesting that  HTTP/2 is ineffective. A number of benchmarks have appeared that show SPDY to be slower across the board, though this can be attributed to a number of flaws in the benchmarks:

  - Testing in low-latency, high-bandwidth local connections rather than over the internet
  - Sharding SPDY connections across domains (as mentioned previously, domain sharding hurts SPDY performance)
  - Using a SPDY proxy rather than running a server that delivers content directly
  - Comparing benchmark data from a SPDY mirror of a website to the website itself rather than a HTTP(S) mirror of the website

It *is* possible for non-HTTP/2 requests to load faster in parallel than the same requests made over a HTTP/2 connection, but only on very fast network connections with low latency. Even if such a situation were to occur, the difference between HTTP/2 and non-HTTP/2 load times would be trivial.

HTTP/2 doesn't work on sites that pull in third-party content
: Partly false. HTTP/2 doesn't work across domains unless the domains share the same IP address. Third party content doesn't (usually) live on the same address as first party content, meaning that HTTP/2 connections are simply not used for third party content (unless of course the third party host uses HTTP/2).

HTTP/2 is hard to configure
: This is indeed something of a downside. For many developers, even setting up HTTPS properly and making sure it stays properly configured and up-to-date can be a real challenge. Implementing HTTP/2 alongside that configuration doesn't make the challenge any easier.

HTTP/2 isn't supported in IE
: This is somewhat true. Only SPDY3 is supported by Internet Explorer 11, and even then only when running on Windows 8. This is because only Windows 8 supports the TLS extension (NPN) needed to signal that the client can use SPDY. Microsoft Edge and IE11 also support HTTP/2 on Windows 10.


## QUIC

QUIC is another project by Google to eventually become the successor to SPDY. It is currently in a highly experimental state. Unlike HTTP/2, QUIC uses UDP rather than TCP, enabling content to be received in a truly asynchronous manner and eliminating blockages caused by a single slow packet. QUIC also seeks to decrease the number of round trips made to and from the server, and eliminate most--if not all--of the time required to establish a connection.

QUIC is currently implemented in Chrome and Opera and can be enabled by visiting `chrome://flags` and `opera://flags` respectively. Most Google properties support QUIC, and a reference implementation of the QUIC server has been published in the Chromium source code repository.
