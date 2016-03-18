# Preloading

Just as a `<link>` tag can be used to perform DNS lookups in advance, another kind of `<link>` tag can be used to instruct the browser to fetch an asset before it is needed.

```html
<link rel="prefetch" href="/images/will_be_used_later.jpg">
```

Optionally, an HTTP header can be specified (or its `<meta>` equivalent):

```
Link: </images/will_be_used_later.jpg>; rel=prefetch
```

Lastly, web servers running HTTP/2 or SPDY can use their respective protocols' "push" functionality to preemptively send content to the browser, though usage of this and configuration is largely implementation-specific at the moment.

These assets are loaded with a lower priority than normal assets, and will only be loaded during "browser idle time". This is generally after the current page (or the outermost document, in the case of frames) has completed loading. Using prefetching is useful in many scenarios:

- On a login page to load assets for a logged-in dashboard: in this case, the client will already have cached the assets required to load the logged-in version of the site while the user is entering their credentials
- On a page that uses AJAX to display additional content that contains images: consider a form that displays a "Thank You" message with a graphic once submitted
- In an image gallery where only a subset of images are loaded at a time
- In an article that spans multiple pages


### Prerendering

Another technique used in some browsers is prerendering. This uses a similar `<link>` tag to prefetching:

```html
<link rel="prerender" href="http://example.com/login">
```

Unlike prefetching, however, prerendering instructs the browser to essentially begin navigating to the page in a background browsing context. This incurs the resource overhead of loading the prerendered page in the background. Prerendering should only be used for very specific cases:

- Pages that the user is almost always likely to visit next, like the Login page from a site's homepage
- Pages that involve true navigation (prerendering becomes complicated and messy for sites that use the HTML5 `pushState` and `popState` APIs to generate pages on the fly)
- Pages that are not very resource-heavy
- Pages that perform certain actions, such as creating popups, requesting HTTP authentication, playing audio or video, including plugins like Flash, or making non-GET AJAX requests.

Prerendering is supported in Chrome and Internet Explorer. At the time of writing, Firefox support is underway[^1].

[^1]: See http://bugzil.la/730101


### Preconnecting

There are cases where prerendering may not be possible. For example, if you have a login form that will redirect to a separate hostname (e.g., from `www.example.com` to `app.example.com`), a `prerender` tag will lead to an error. You may still wish to do some of the work of signing the user in beforehand, however. In this case, you can use `preconnect`.

```html
<link rel="preconnect" href="https://app.example.com" crossorigin>
```

When preconnecting, the browser will resolve the URL in the `href=""` attribute. It will then perform the following actions:

- Perform a DNS lookup of the hostname in `href=""`
- Create a new connection to the hostname at `href=""`
- Establish a new TLS session, if it is needed

If the request is to a different hostname than the one the user is currently on, the `crossorigin` attribute needs to be present.

The W3C says that this, "allows the user agent to mask the high latency costs of establishing a connection." [^2]

[^2]: https://www.w3.org/TR/resource-hints/#preconnect


### Preloading

The W3C has formalized a specification for this. It is called "preloading" rather than "prefetching". Ultimately, preloading works in a very similar way to prefetching. A `<link>` tag just as it is for `<link rel="prefetch>`, though it accepts an extra attribute: `as`.

```html
<link rel="preload" href="/my/image.png" as="image">
```

When the browser sees a `<link rel="preload">` tag, it will begin downloading the image. The priority of the request would be the same as if the image was loaded with an `<img>` tag. Using `as="font"` would make a request with the priority of a web font.

The values that you can use for the `as=""` attribute are the same values used by the `fetch()` API. Common values include `font`, `script`, `style`, `audio`, and `video`.

In addition to a `<link>` tag, you can also use a `Link` HTTP header:

```
Link: </my/image.png>; rel=preload; as=image
```

### Which should I choose?

Between prefetching, prerendering, preconnecting, and preloading, it can be difficult to decide which technique to use. Here are some tips for when to choose these options.

1. Choose prerendering if you expect your user to navigate to a particular page, like a "step two". The user must already have access to the page (i.e., their login state should not need to change).
2. Choose preloading if you want the user's browser to start downloading a specific file as soon as possible.
3. Choose prefetching if you want the user's browser to start downloading a specific file when the browser is idle.
4. Choose preconnecting if the user might not have access to the target URL just yet, or if you don't want the user to start using bandwidth on the file.

