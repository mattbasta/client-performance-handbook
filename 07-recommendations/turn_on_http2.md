# Turn on HTTP/2

One of the simplest wins you can achieve without modifying your web application directly is to enable HTTP/2 on your front-end server. Depending on your stack, this can be trivial or complex. This section contains information for most common configurations, though other setups or servers (e.g., Apache Traffic Server or Zeus) should provide instructions in their respective manuals. Each section assumes you already have HTTPS set up for your site.

Note that implementing HTTP/2 support at each level of your stack is unnecessary. That is, if you have Nginx sitting in front of Apache, turning HTTP/2 on for the connection between Nginx and Apache is probably unnecessary. HTTP/2 will only provide a measurable difference if it's implemented between your users and the first layer of your server stack.


## Apache (httpd)

Enabling HTTP/2 in Apache's httpd is as simple as setting up the `mod_h2` (or `mod_http2`) plugin. This is available with Apache 2.4.17 and up. To check whether it is already available, simply look in either of these two directories:

```
/etc/apache2/mods-available
/etc/httpd/conf.d
```

https://httpd.apache.org/docs/2.4/en/mod/mod_http2.html

First, make sure that one of your configuration files loads the `http2_module` plugin:

```
LoadModule http2_module modules/mod_http2.so
<IfModule http2_module>
    LogLevel http2:info
</IfModule>
```

You're now ready to add the protocol to your `httpd.conf` file[^1]:

```
Protocols h2 http/1.1
```

[^1]: Configuring HTTP/2 to work without HTTPS enabled is left as an exercise to the reader.

Now, open your configuration file and find the virtual host that you want to enable HTTP/2 for. In that section, simply add the `H2Direct on` directive:

```
<VirtualHost *:80>
    H2Direct on
</VirtualHost>
```

More information is available at the developer's website:

https://icing.github.io/mod_h2/howto.html


## Nginx

As of Nginx 1.9.5, HTTP/2 support is included by default. It's very easy to enable the protocol. First, make sure all of your users are using HTTPS (with HSTS, redirects, etc.).

Next, simply update your conf files to look something like this:

```
server {
    listen 443 ssl http2 default_server;

    ssl_certificate    server.crt;
    ssl_certificate_key server.key;
    ...
}
```

Notice `http2` under the `listen` directive. Now, just restart your Nginx server with `nginx -s reload`.


## Apache Traffic Server

Apache Traffic Server (ATS) is a less-frequently used front-end server, but it's often the outermost layer in some stacks.

HTTP/2 is available in ATS 5.3.2 or above. First, configure your server to use HTTPS. After that, simply add the following config entry:

    CONFIG proxy.config.http2.enabled INT 1

That's it!
