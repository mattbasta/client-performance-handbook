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

Every site should use HTTPS where possible. HTTPS comes at a cost, though: the handshake process used to exchange cryptographic keys at the beginning of the connection adds a number of extra roundtrips between the server and client.

## Request

Once the connection has been established (and secured, if over HTTPS), the client can make a request to the server. Depending on a number of factors, the information included in this request can be quite slow.

## Redirect

If the server receives a request and determines that the client should instead be requesting a different resource (or has completed its processing and is sending the client to another resource), it responds to the client with a redirect. A redirect is a combination of a `3XX` status code and a `Location` HTTP header.

## Response

After the server receives the client's request, it can begin processing and formulating a response. The first part of this response is the HTTP response headers, followed immediately by the content the user requested.

## Assets and Payload

Once the client has begun receiving the server's response, it will attempt to being parsing it. In the process, the client will make additional requests to the server as the response indicates more assets are required to build the page. This can include core components of the page, such as CSS, JavaScript, and images, and it can also include other content like fonts, multimedia and plugins, other pages and framed content, and more.

## Render and Composite

As the client builds the page that the user requested using the assets that it fetched, it begins calculating the layout of the page that was requested, rendering and compositing the elements in the layout, and painting those elements to the screen. The processes involved in this step are known as the "critical path."

## Execution

As the client downloads JavaScript, it must be executed (though the point at which the scripts execute may vary depending on how they were included in the page). Execution describes the initial "run-to-completion" for the scripts on the page.

## Initialization

There are a number of points at which the scripts on a page may execute. It's rarely the case, however, that any of these points are ideal times to begin running application logic; the DOM may not be available, other scripts may not have run yet, or other dependencies may not yet have been met. Initialization is the point at which applications choose to begin running application logic and setting up state for the page.
