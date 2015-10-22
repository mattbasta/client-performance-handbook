# Redirects

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

| Request | Response |
| -- | -- |
| GET `http://popularsoftware.biz` | `302 Found` redirect to `https://www.popularsoftware.biz/login` |

A single redirect could accomplish the work of three. In short, be mindful of redirections in your web application to prevent latency and minimize delays during page loads.
