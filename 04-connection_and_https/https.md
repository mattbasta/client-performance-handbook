# HTTPS

HTTPS is better known to most people as "SSL". SSL is a misused term: the actual SSL protocol was deprecated many years ago and was replaced with TLS, or Transport Layer Security. Few internet users, however, have ever heard the term TLS, and HTTPS certificates are still sold to site owners as "SSL certificates" in many cases. "HTTPS" is the friendly term used by browsers to describe HTTP connections over TLS, SSL, or any other secure connection.

One of the biggest web performance myths in existence is that HTTPS is slow because of the performance cost of encryption. This is false. The overhead of secure connections is incredibly low and most site owners will never notice it.

The primary bottleneck involving HTTPS is the number of round trips that are necessary to establish a connection. Two full round trips are necessary as part of a "handshake" to exchange cryptographic key information before the client can even begin to make a request.

One half of the solution is to decrease the number of requests (discussed in the previous section). Fewer connections mean the browser and the server will handshake fewer times.

Having a CDN can also help immensely. A CDN with a point of presence (PoP) near to the user decreases the amount of time it takes to actually perform the handshake. Additionally, some CDNs or network service providers will offer HTTPS termination services that allow you to terminate the HTTPS connection to the user in a local PoP rather than in a data center far away. Terminating the connection closer to the user means TLS handshakes take less time to be performed.

It is possible to create your own termination proxy using open source software like Nginx in combination with a geographic DNS service, though the effort and costs involved are often greater than the benefits. Constructing such a proxy is left as an exercise to the reader.

There are some secondary changes that you can make to improve HTTPS performance:

- **Use secure but fast ciphers.** The most secure ciphers that are employed are oftentimes overkill for most websites and are much slower than some of their good-enough-but-less-secure counterparts. The configurations provided in the sections on setting up HTTPS contain ideal cipher lists.
- **Cache credentials.** If the credentials for a client are cached, quite a bit of the connection process can be skipped. This is done with the `SSLSessionCache` and `SSLSessionCacheTimeout` directives in Apache and `ssl_session_cache` and `ssl_session_timeout` directives in Nginx.
- **Enable stapling.** OCSP stapling is a technique that is used to send information about the certificate chain for a site along with the site's key info during the connection process. By "stapling" this data, the browser may be able to skip a few extra steps to check to see whether the site's certificate has been revoked. This may or may not provide much benefit, and can even decrease performance.[^1] You should test whether OCSP stapling improves page load times for your users.

[^1]: OCSP stapling may not provide much benefit if the browser does not perform an OCSP request for the site. If the browser wouldn't have performed an OCSP request, the overhead from stapling may overflow the TCP window and create additional packets.
