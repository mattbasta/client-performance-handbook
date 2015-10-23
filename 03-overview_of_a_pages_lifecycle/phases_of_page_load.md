# Phases of Page Load

## DNS Lookups

The first step in navigation to any page is the DNS lookup. Translating a domain name into an IP address is necessary, as without it, the client has no way of knowing what server to connect to. Fortunately, DNS lookups are cached and users will only experience the penalty of performing the lookup infrequently. When DNS lookups are neglected, however, they can have a drastic negative impact on page performance.

When developing, you should consider DNS lookups to take half of a second to complete. Every time you instruct the browser to connect to a new hostname (whether by redirection, loading an asset, performing an XHR, etc.), assume that the connection will take an extra half second. In the next chapter, techniques will be discussed to perform DNS queries in advance, helping to mitigate this problem.


## TCP Connection

Once a client knows what the address of the remote server is, the next step is to actually make the connection.

Describing the creation of the actual TCP connection is somewhat of a cop-out on my part, since DNS lookups involve a TCP connection. Unlike a DNS lookup, which is almost always made to the user's ISP, the connection to the remote web server involves communication with an unknown third party.

There is little you can do to improve the performance of the TCP connection itself. Rather, you can decrease the number of connections required by decreasing the number of assets or hostnames you connect to, or by turning on support for HTTP/2.

Another aspect of TCP connections is the inherent latency involved. The TCP protocol requires acknowledgment (an "ACK") of each packets sent over the network. Each packet requires an ACK before the next packet will be sent. This means that each packet requires at least a full round trip to the remote server.

Reducing this latency is often not possible, but sites can be optimized to avoid it in a number of ways:

- by using a CDN, which places the remote server physically closer to the user)
- moving data out of critical parts of the network request (e.g., removing cookies) to allow the page to begin loading sooner
- reducing the overall number of packets by decreasing the amount of data sent to the client


## HTTPS Handshake

Every site should use HTTPS where possible. HTTPS comes at a cost, though: the handshake process used to exchange cryptographic keys at the beginning of the connection adds a number of extra roundtrips.

The HTTPS handshake is unavoidable in order to use HTTPS. Reducing the number of connections between the client and the server is the only way to mitigate the cost of this step[^1]. Using HTTP/2 (discussed in the next chapter) can vastly reduce the number of connections.

[^1]: Of course, you could disable HTTPS in order to eliminate the HTTPS handshake. This is strongly discouraged, though, as the benefits of HTTPS far outweigh the handful of milliseconds that can be saved.


## Request

Once the connection has been established (and secured, if over HTTPS), the client can make a request to the server. Depending on a number of factors, the information included in this request can be quite slow.

The performance properties of the request vary with the use case. In general, the "best" requests are those that include the least amount of information. Requests can grow unweildy as the number of headers grows, or as the length of the URL increases.

Cookies are another factor that can have a negative impact on the performance of the request. Small requests can fit in a single TCP packet, requiring a single roundtrip. Large amounts of request headers or very large cookies can cause this to overflow into a second packet, or more.


## Redirect

If the server determines that the client should instead be requesting a different resource, it responds to the client with a redirect. A redirect is a combination of a `3XX` status code and a `Location` HTTP header.

Redirects should be minimized. In the next chapter, the costs of redirects are described in great detail, along with common ways of minimizing or eliminating redirects in your site.


## Response

After the server receives the client's request, it can begin processing and formulating a response. The first part of this response is the HTTP response headers, followed immediately by the content the user requested.

Because responses can be streamed, there is no straightforward advice for improving the performance of a site's response. Minimizing the time the server takes to respond to the client and reducing the size of the server's output are easy ways of achieving this, but that advice is not very actionable. The "Assets and Payload" chapter goes into great detail on this topic.


## Assets

Once the client has begun receiving the server's response, it will attempt to being parsing it. In the process, the client will make additional requests to the server as the response indicates more assets are required to build the page. This can include core components of the page such as CSS, JavaScript, and images. It can also include other content like fonts, multimedia and plugins, other pages and framed content, and more.


## Rendering and Compositing

As the client builds the page using the assets that it fetched, it calculates the layout and renders and composites the elements of the page. The processes involved in this step are known as the "critical path." Reducing the amount of work that happens during this step is crucial to improving the responsiveness of the browser. Achieving this is only possible through fine-tuning the CSS and JavaScript on the page.


## Execution

As the client downloads JavaScript, it must be executed (though the point at which the scripts execute may vary depending on how they were included in the page). Execution describes the initial "run-to-completion" for the scripts on the page.


## Initialization

There are a number of points at which the scripts on a page may execute. It's rarely the case, however, that any of these points are ideal times to begin running application logic; the DOM may not be available, other scripts may not have run yet, or other dependencies may not yet have been met. Initialization is the point at which applications choose to begin running application logic and setting up state for the page.
