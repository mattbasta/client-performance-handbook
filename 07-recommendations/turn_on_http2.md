# Turn on HTTP/2

One of the simplest wins you can achieve without modifying your web application directly is to enable HTTP/2 on your front-end server. Depending on your stack, this can be trivial or complex. This section contains information for most common configurations, though other setups or servers (e.g., Apache Traffic Server or Zeus) should provide instructions in their respective manuals. Each section assumes you already have HTTPS set up for your site.

Note that implementing HTTP/2 support at each level of your stack is unnecessary. That is, if you have Nginx sitting in front of Apache, turning HTTP/2 on for the connection between Nginx and Apache is probably unnecessary. HTTP/2 will only provide a measurable difference if it's implemented between your users and the first layer of your server stack.


## Apache

Google's `mod_spdy` plugin was donated to the Apache Foundation in 2012, meaning it is now a part of the Apache HTTPD core. To check whether it is already available, simply look in either of these two directories:

```
/etc/apache2/mods-available
/etc/httpd/conf.d
```

If `mod_spdy` is not available, you can install it rather easily with `dpkg`. First, download the appropriate `mod_spdy` binary from Google and follow the instructions for installing it:

https://developers.google.com/speed/spdy/mod_spdy/

At this point, you should see the `mod_spdy` configuration in the directories listed above.

To enable SPDY, open the configuration file. You'll see a conditional that looks like the following:

```
<IfModule spdy_module>

    ...

</IfModule>
```

Within this conditional, add the following directive:

```
SpdyEnabled on
```

If the same directive is set to "off" somewhere else in the file, remove it. Finally, restart Apache.


## Nginx

First, make sure you have OpenSSL 1.0.1 or higher installed:

```bash
apt-get update
apt-get install build-essential libssl-dev libpcre3 libpcre3-dev
```

Next, download and install Nginx. At the time of writing, the latest stable version of Nginx is 1.6.0. You may wish to visit http://nginx.org to find the most current stable version.

W> #### Warning!
W>
W> If you have Nginx already installed on your server, following these instructions will remove it and may not preserve your configuration. This is because---at the time of writing---no popular software repositories have versions of Nginx compiled to run SPDY. If you follow these instructions, be sure to back up your `nginx.conf` and any other custom configuration files you might have.


```bash
cd /tmp

# Download and unpack nginx
wget http://nginx.org/download/nginx-1.6.0.tar.gz
tar xvf nginx-1.6.0.tar.gz

cd nginx-1.6.0
# Configure nginx to use SPDY
./configure --with-http_spdy_module --with-http_ssl_module
# Note that you may need to pass additional config parameters
# if you require additional functionality.

# Remove existing nginx installations
dpkg -l | grep nginx
apt-get remove nginx-full nginx-common

make
make install
```

At this point, Nginx should be installed but not configured to your site. If you had a previous version of Nginx installed, restore your old configuration files. Make sure SSL is set up correctly using the setup guide in previous sections. Once you've completed that, setting up SPDY is simple. Just update your configuration to replace this:

```nginx
listen          443 ssl;
server_name     example.com;
```

with the following:

```nginx
listen          443 ssl spdy;  # Add SPDY
server_name     example.com;
spdy_headers_comp   5;  # Turn on SPDY header compression
```

Finally, restart Nginx with

```bash
/usr/local/nginx/sbin/nginx -s reload
```