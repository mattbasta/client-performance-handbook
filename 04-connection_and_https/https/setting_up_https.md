# Setting up HTTPS


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


## Apache

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


## Nginx

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
