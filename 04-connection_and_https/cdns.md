# CDNs

A CDN, or Content Delivery Networks, can do a lot to help speed up a website. Most CDNs offer a plethora of services:

<dl>
    <dt>Local Points of Presence (PoPs)</dt>
    <dd>Servers that are geographically close to users in a particular region. These servers allow users to connect with much lower latency than if they had been connecting over a much larger distance. Strategic positioning of PoPs allows a CDN to minimize the number of network "hops" a client needs to make to interact with the server.</dd>
    <dt>SSL Termination</dt>
    <dd>By terminating SSL (or performing the SSL handshake) closer to the user, connections can be established much more quickly, allowing the client to make requests faster than they otherwise could.</dd>
    <dt>DoS Protection</dt>
    <dd>Many CDNs market DoS (denial of service) attack prevention. By sitting between users and web servers, the CDN can be better prepared to accept a flood of requests from an attacker, or to block the flow of traffic from compromised users.</dd>
    <dt>Full-Blown Features</dt>
    <dd>It's usually the case that CDNs are quick to implement performance-improving features and cutting-edge technology for content delivery. This includes technology like HTTP/2, IPV6, and low-level connection tuning.</dd>
</dl>

Some CDNs offer value-added services, like country detection (eliminating the need for a GeoIP lookup) or Edge Includes. These can potentially have performance benefits if used properly.


## How does a CDN work?

Setting up a CDN is usually very simple to do. Virtually all CDNs operate as proxies that mirror the content from another site. For instance, you might set up `cdn1.example.com` to point at a CDN mirroring `www.example.com`. When `https://cdn1.example.com/asset/include.js` is requested, the CDN would fetch `https://www.example.com/asset/include.js` and cache it. All users requesting that URL from that point on would fetch the cached version of `include.js`.

Note that changing content once it has been cached by a CDN is not always possible. To modify the content, you must access it with a different name. For example, you might reference `https://cdn1.example.com/asset/include.js?20140101` to refer to a version of the file that was updated on January 1, 2014. Every time the file changes, you would update the query string to refer to a URL for the updated version. Using hashes instead of dates or times is also common.

Most CDNs accomplish this mirroring via DNS. Some CDNs, such as CloudFlare, will ask you to change the nameservers for your domain to point at their own nameservers. In doing this, they are able to automatically set the addresses for each of your hostnames to the addresses of their own PoP servers. Other CDNs, like EdgeCast, will give you hostnames to add as CNAME records in your own DNS manager.

Another type of CDN, like Amazon CloudFront, allows you to upload files to some sort of storage (in CloudFront's case, Amazon S3). You are then provided with a new hostname that you can point users at directly, or reference with a CNAME record.


## Scale

The biggest benefit to using a CDN for content hosting is to decrease load on your web servers. Serving static files is expensive in most cases, especially if your servers are serving both dynamic and static content. Users fetching static content tie up sockets, use lots of bandwidth, and decrease the server's ability to quickly respond to more important requests. Some "hacks" exist to make this easier, like Nginx's `XSendfile` plugin, but even these approaches can be problematic.

Large assets (JavaScript, images, web fonts, etc.) usually require a great deal of disk IO, increasing server load and decreasing the lifespan of a server's disk drives. Hosting large static assets in memory-based caches can be much faster, but also very expensive.

CDNs specialize in this type of static content. Their networks and the software that runs on them are designed to solve these very problems. Using a CDN eliminates the need to worry about scaling your infrastructure to handle large amounts of traffic around your assets.

Be wary, however, that designing your infrastructure with a CDN in mind makes your whole site reliant on the presence of a CDN. Changing CDNs can be extremely difficult down the road, especially if the new CDN does not offer the same features as your previous CDN. Weaning your site off of a third party CDN can also involve huge investments in new infrastructure later.


## Pitfalls

One of the most perplexing issues than many people encounter when evaluating a CDN is that they see *slower* responses than accessing their content directly. This is usually the case when only a small number of users are accessing a site with a CDN, especially if the users are distributed around the world.

The cause is that the CDN hasn't seen the files that the user is requesting yet. Instead of the user accessing content directly, the CDN servers are an extra "hop" involved in fetching the content. Until all of the CDN servers in all of the service provider's data centershave seen all of your files, slow response times may happen.

Another common pitfall is a CDN over-selling its service. When you use a CDN, your content is hosted on the same servers that host content for thousands of the CDN's other customers. If one of the CDN's other customers is experiencing a DoS attack, the speed of your website could be negatively impacted. Be sure to thoroughly research all of your options before you commit to any CDN.

Cost can be another pitfall with a CDN. Before choosing a CDN, model your expected costs based on your own data. Consider how much it will cost to be a customer with historical data, and also project figures accounting for future growth. Also be sure to compare this with the cost of bandwidth on your current host (if any) to account for potential savings. On some platforms, like Linode or Microsoft Azure, large amounts of bandwidth can be far more expensive than using a CDN to transfer the same content.
