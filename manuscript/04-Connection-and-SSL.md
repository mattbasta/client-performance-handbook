# Connection and SSL

## HTTPS

HTTPS is better known to most people as "SSL". SSL is a misused term: the actual SSL protocol was deprecated many years ago and was replaced with TLS, or Transport Layer Security. Few internet users, however, have ever heard the term TLS, and HTTPS certificates are still sold to site owners as "SSL certificates" in many cases. "HTTPS" is the friendly term used by browsers to describe HTTP connections over TLS, SSL, or any other secure connection.

One of the biggest web performance myths in existence is that HTTPS is slow because of the performance cost of encryption. This is false. The overhead of secure connections is incredibly low and most site owners will never notice it.

The primary bottleneck involving HTTPS is the number of round trips that are necessary to establish a connection. Two full round trips are necessary as part of a "handshake" to exchange cryptographic key information before the client can even begin to make a request.

One half of the solution to this problem is to simply decrease the number of requests (discussed in the previous section). Fewer connections to the server means that the handshake needs to be made fewer times.

Having a CDN can also help immensely. A CDN with a point of presence (PoP) near to the user decreases the amount of time it takes to actually perform the handshake. Additionally, some CDNs or network service providers will offer HTTPS termination services that allow you to terminate the HTTPS connection to the user in a local PoP rather than in a data center far away. Terminating the connection closer to the user means TLS handshakes take less time to be performed.

It is possible to create your own termination proxy using open source software like Nginx in combination with a geographic DNS service, though the effort and costs involved are often greater than the benefits. Constructing such a proxy is left as an exercise to the reader.

There are some secondary changes that you can make to improve HTTPS performance:

- **Use secure but fast ciphers.** The most secure ciphers that are employed are oftentimes overkill for most websites and are much slower than some of their good-enough-but-less-secure counterparts. The configurations provided in the sections on setting up HTTPS contain ideal cipher lists.
- **Cache credentials.** If the credentials for a client are cached, quite a bit of the connection process can be skipped. This is done with the `SSLSessionCache` and `SSLSessionCacheTimeout` directives in Apache and `ssl_session_cache` and `ssl_session_timeout` directives in Nginx.
- **Enable stapling.** OCSP stapling is a technique that is used to send information about the certificate chain for a site along with the site's key info during the connection process. By "stapling" this data, the browser may be able to skip a few extra steps to check to see whether the site's certificate has been revoked. This may or may not provide much benefit, and can even decrease performance.[^stapling_benefit] You should test whether OCSP stapling improves page load times for your users.

[^stapling_benefit]: OCSP stapling may not provide much benefit if the browser does not perform an OCSP request for the site. If an OCSP check would not have otherwise been performed, the overhead from stapling may overflow the TCP window and create additional packets.


### Setting up HTTPS on your site

Despite the downsides to running HTTPS on your website, it's usually always better to have HTTPS than to not have it. HTTP2, for instance, only works when HTTPS is enabled.

The first step to setting up HTTPS is obtaining a certificate. Depending on what you're looking to accomplish by setting up HTTPS, you may require a different type of certificate.

Here are some of the most common types:

Single Domain Certificate
: This type of certificate is good for a single hostname only.

Multi-Domain Certificate
: These certificates--also known as Subject Alternative Name certificates--cover up to 100 domain names. For instance, one of these certificates may cover `example.com`, `anotherexample.com`, etc. These certificates tend to be quite expensive.

Wildcard Certificate
: A wildcard certificate covers a domain (like `example.com`) and all of the subdomains (like `www.example.com` and `mail.example.com`). A wildcard certificate is valid for an unlimited number of subdomains. These certificates can be quite expensive and usually require the purchaser to agree to certain terms.


There are also a number of types of validation for certificates:

Domain Validated Certificate
: A domain validated certificate is a certificate that is issued for a single domain which is validated by the certificate issuer. These are easy to obtain and are usually inexpensive, and the process of obtaining one is usually done automatically.

Organization Validated Certificates
: These certificates display information about the certificate holder in modern browsers. Websites that are looking to provide HTTPS for identity rather than just encryption should consider this type of certificate. This process requires a more thorough validation process than the Domain Validated Certificates that involves a human verification step. Acquiring one of these certificates can take a few days.

Extended Validation Certificate
: This type of certificate is much more expensive and involves a very thorough validation process by the issuer, usually involving phone calls to the owner of the domain, requesting the domain owner add DNS records, adding an HTML tag to the site on the domain, perform certain kinds of background checks, or other verification techniques. Purchasers may be required that they have the rights to use the domain name that the certificate is being issued for. "EV" certificates turn the browser address bar green or show a green chiclet in modern browsers.

Self-Signed Certificates
: These certificates are free and can be created by anyone, however because the certificate is not issued by an authority, users accessing sites with self-signed certificates will receive a very scary warning. You don't want one of these for your site.


If you're using shared hosting or some other sort of managed host (i.e.: you don't manage your own server or the software running on it), the company that hosts your website should automatically start using the certificate if you purchased it from them, or should have some mechanism for adding the certificate to your site via a control panel.

If you manage your own host, you're probably using Nginx or Apache to serve your website. After you've purchased and downloaded your certificate, the first thing you're going to want to do is place your certificate and key files (I'll call them `example.com.pem` and `example.com.key` respectively) in a known location on your server. This is generally the `/etc/ssl` directory. If you received a `.cert` file instead of a `.pem` file, simply change the extension to `.pem`.

I> If you're using IIS, refer to the documentation on the IIS website (http://www.iis.net/learn/manage/configuring-security/how-to-set-up-ssl-on-iis) for instructions.

```bash
# Make the directories if they do not exist.
mkdir -p /etc/ssl/certs /etc/ssl/private

# Move the key and the cert to their proper locations.
mv example.com.pem /etc/ssl/certs
mv example.com.key /etc/ssl/private
```


#### Apache

If you're using Apache, first make sure that mod_ssl is installed. You can do that by running the following:

```bash
# Install the mod_ssl extension
sudo a2enmod ssl
```

Next, update your site's `VirtualHost` to look something like the following:

```apache
# This loads the mod_ssl module
LoadModule ssl_module modules/mod_ssl.so

# Use SSL stapling (available on Apache 2.4+)
SSLUseStapling on
SSLStaplingResponderTimeout 5
SSLStaplingReturnResponderErrors off
SSLStaplingCache shmcb:/var/run/ocsp(128000)

# Redirect all non-HTTPS users to HTTPS
Listen 80
<VirtualHost *:80>
    ServerName example.com
    Redirect permanent / https://example.com/
</VirtualHost>


Listen 443  # HTTPS runs on port 443
<VirtualHost *:443>
    ServerName example.com

    # Make sure SSL is enabled
    SSLEngine on

    # Point mod_ssl at the certificate files
    SSLCertificateFile /etc/ssl/certs/example.com.pem
    SSLCertificateKeyFile /etc/ssl/private/example.com.key

    # Tell mod_ssl to use fast but secure ciphers only
    SSLCipherSuite ECDH+AESGCM:ECDH+AES256:ECDH+AES128:DH+3DES:!ADH:!AECDH:!MD5
    # We want to use "good enough" ciphers instead of the strongest
    # ciphers where possible.
    SSLHonorCipherOrder on

    # Cache client credentials to improve performance
    SSLSessionCache shm:/usr/local/apache/logs/ssl_gcache_data(512000)
    SSLSessionCacheTimeout 600

    # Tell the client to always use HTTPS
    Header add Strict-Transport-Security "max-age=15768000"


    # Put your original Apache VirtualHost configuration here.

</VirtualHost>
```

Note that some authorities may require you to additionally add a `SSLCertificateChainFile` directive to your `VirtualHost`. If this is the case, the CA will make you aware of it and should provide the necessary file(s) and instructions on how to add this extra data.

After you've saved your configuration file, simply restart Apache.

```bash
service apache2 restart
```


#### Nginx

In your `nginx.conf` file (usually found in `/usr/local/nginx/conf/`), you'll find something like this:

```nginx

# ... snip ...

http {
    # ... snip ...

    server {
        listen       80;
        server_name  example.com;

        # ... snip ...
    }

    # ... snip ...

}
```

You should update the configuration to look like the following:

```nginx

# ... snip ...

http {
    # ... snip ...

    server {
        listen       80;
        server_name  example.com;

        # Redirect HTTP users to HTTPS
        return 301 https://example.com;
    }

    server {
        listen               443 ssl;
        server_name          example.com;

        # Point Nginx at the certificate files
        ssl_certificate      /etc/ssl/certs/example.com.pem;
        ssl_certificate_key  /etc/ssl/private/example.com.key;

        # Allow Nginx to use an SSL session cache
        ssl_session_cache    shared:SSL:20m;
        ssl_session_timeout  10m;

        # Some SSL configuration to only allow secure protocols and
        # ciphers.
        ssl_protocols        TLSv1 TLSv1.1 TLSv1.2;
        ssl_ciphers ECDH+AESGCM:ECDH+AES256:ECDH+AES128:DH+3DES:!ADH:!AECDH:!MD5;
        ssl_prefer_server_ciphers  on;
        keepalive_timeout    70;

        # Tell the browser that we always want the user to connect via
        # HTTPS to this site.
        add_header Strict-Transport-Security "max-age=31536000";

        # Enable OCSP stapling
        ssl_stapling on;
        resolver 8.8.8.8 8.8.4.4 valid=300s;  # DNS servers to reference
        resolver_timeout 5s;

        # At this point, include your previous Nginx configuration,
        # such as `location` directives.

        location / {  # Example location directive
            root    /var/www;
            index   index.html index.htm;
        }
    }

    # ... snip ...

}
```

Finally, restart Nginx:

```bash
service nginx restart

# If the above does not work, try
nginx -s reload
# or
/usr/local/nginx/sbin/nginx -s reload
```

Note that Nginx also supports verifying the OCSP responses against a local copy of the issuer, root, and intermediate certificates. This is beneficial to enable, but can be tricky to set up. See the Nginx documentation for more information[^nginx_ocsp_verify].

[^nginx_ocsp_verify]: http://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_stapling_verify


## CDNs

A CDN, or Content Delivery Networks, can do a lot to help speed up a website. Most CDNs offer a plethora of services:

Local Points of Presence (PoPs)
: Servers that are geographically close to users in a particular region. These servers allow users to connect with much lower latency than if they had been connecting over a much larger distance. Strategic positioning of PoPs allows a CDN to minimize the number of network "hops" a client needs to make to interact with the server.

SSL Termination
: By terminating SSL (or performing the SSL handshake) closer to the user, connections can be established much more quickly, allowing the client to make requests faster than they otherwise could.

DoS Protection
: Many CDNs market DoS (denial of service) attack prevention. By sitting between users and web servers, the CDN can be better prepared to accept a flood of requests from an attacker, or to block the flow of traffic from compromised users.

Full-Blown Features
: It's usually the case that CDNs are quick to implement performance-improving features and cutting-edge technology for content delivery. This includes technology like HTTP2, IPV6, and low-level connection tuning.


### How does a CDN work?

Setting up a CDN is usually very simple to do. Virtually all CDNs operate as proxies that mirror the content from another site. For instance, you might set up `cdn1.example.com` to point at a CDN mirroring `www.example.com`. When `https://cdn1.example.com/asset/include.js` is requested, the CDN would fetch `https://www.example.com/asset/include.js` and cache it. All users requesting that URL from that point on would fetch the cached version of `include.js`.

>A Note that changing content once it has been cached by a CDN is not always possible. To modify the content, you must access it with a different name.
>A
>A For example, you might reference `https://cdn1.example.com/asset/include.js?20140101` to refer to a version of the file that was updated on January 1, 2014. Every time the file changes, you would update the query string to refer to a URL for the updated version. Using hashes instead of dates or times is also common.

Most CDNs accomplish this mirroring via DNS. Some CDNs, such as CloudFlare, will ask you to change the nameservers for your domain to point at their own nameservers. In doing this, they are able to automatically set the addresses for each of your hostnames to the addresses of their own PoP servers. Other CDNs, like EdgeCast, will give you hostnames to add as CNAME records in your own DNS manager.

Another type of CDN, like Amazon CloudFront, allows you to upload files to some sort of storage. You are then provided with a hostname that you can point users at directly, or point a CNAME record at. Depending on your use case, this may be more or less convenient.


### Pitfalls

One of the most perplexing issues than many people encounter when evaluating a CDN is that they see *slower* responses than accessing their content directly. This is usually the case when only a small number of users are accessing a site with a CDN, especially if the users are distributed around the world.

The cause is that the CDN simply hasn't seen the files that the user is requesting yet. If you request a file from a CDN and the CDN doesn't have a copy of the file, it needs to pause the request, visit your server, wait for a response, then forward it on to the user. Until all of the servers in all of the applicable data centers at your CDN have seen all of your files, you'll notice some poor response times.

Another common pitfall is a CDN over-selling its service. When you use a CDN, your content is hosted on the same servers that host content for thousands of the CDN's other customers. If one of the CDN's other customers is experiencing a DoS attack, the speed of your website could be negatively impacted. Be sure to thoroughly research all of your options before you commit to any CDN.

Cost can be another pitfall with a CDN. Before choosing a CDN, model your expected costs based on your own data. Consider how much it will cost to be a customer with historical data, and also project figures accounting for future growth. Also be sure to compare this with the cost of bandwidth on your current host (if any) to account for potential savings. On some platforms, like Linode or Microsoft Azure, large amounts of bandwidth can be far more expensive than using a CDN to transfer the same content.


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
