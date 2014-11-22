# Recommendations

The previous chapters have given a lot of detailed information on many aspects of performance, and this chapter is intended to provide you with actionable tips that you can apply directly to your web application to help increase its performance.


## Turn on HTTP2 and/or SPDY

One of the simplest wins you can achieve without modifying your web application directly is to enable HTTP2 and SPDY on your front-end server. Depending on your stack, this can be trivial or complex. This section contains information for most common configurations, though other setups or servers (e.g., Apache Traffic Server or Zeus) should provide instructions in their respective manuals.

At the time of writing, HTTP2 is not well-supported enough to provide a proper setup tutorial, though SPDY is quite mature. Each section assumes you already have HTTPS set up for your site.


### Apache

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


### Nginx

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


## Minify Your Assets

If you aren't already, start using minifiers for your static assets. Many very good ones are available (as discussed in the chapter on minification). To set up a very basic minification system, you can follow the following instructions. As your project matures, it's often common to use a build system like Grunt, Gulp, or simple Makefiles.

These instructions assume that `./node_modules/bin` is on your path.

```bash
# Install the tools needed for a basic build system
npm install uglify-js clean-css

# Concatenate your JavaScript into a single file
# Change `static/` to the path to your assets
find static/ -name "*.js" -exec cat {} \; > static/main.js

# Minify your concatenated JavaScript
# -m performs name mangling, -c performs certain optimizations
uglifyjs -m -c static/main.js > static/main.min.js

# Concatenate your CSS into a single file
find static/ -name "*.css" -exec cat {} \; > static/styles.css

# Minify your concatenated CSS
cleancss static/styles.css > static/styles.min.css
```

After minifying, be sure to update your application to use the minified files instead of the unminified versions.


## Compress Your Images

Compressing your images can provide an astounding amount of performance benefit. For most websites, relatively small images outweigh even very large JavaScript files.

First, ensure you're using the right image format for each of your images. A guide for doing this is available in the Images section of the Assets and Payload chapter.


### OS X

If you're using OS X as your operating system, I highly recommend using ImageOptim:

http://imageoptim.com/

After installing, simply drag and drop your images into ImageOptim. They will automatically be losslessly compressed and saved for you. Most popular image types are supported.


### *NIX Operating Systems

If you're using any other Linux or UNIX operating system, you can use a number of tools to perform compression. The first tool is pngcrush. You can install it from all popular package managers.

```bash
# -ow overwrites the original file
pngcrush -ow static/img/my_png.png
```

JPEG images have a similar corresponding tool known as jpegoptim. It's used in a very similar way:

```bash
jpegoptim static/img/my_jpeg.jpg
```

Lastly, GIF files can be optimized with another such tool: gifsicle.

```bash
# gifsicle requires an output parameter, but it can be the same path
gifsicle static/img/my_gif.gif -O3 -o static/img/my_gif.gif
```
