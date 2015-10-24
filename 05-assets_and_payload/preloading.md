# Preloading

Just as a `<link>` tag can be used to perform DNS lookups in advance, another kind of `<link>` tag can be used to instruct the browser to fetch an asset before it is needed.

```html
<link rel="prefetch" href="/images/will_be_used_later.jpg">
```

Optionally, an HTTP header can be specified (or its `<meta>` equivalent):

```raw
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


### Subresources

Chrome currently includes experimental support for a feature known as subresources. Subresources are `<link>` tags similar to prefetch and prerender requests, though they represent resources that will be used more immediately.

Subresources could be used, for example, to hint to the browser to request all of the CSS, JavaScript, and images at the very top of the page. A single HTTP/2 connection could be used to send all of the requests simultaneously, regardless of the position of the asset in the document or how much of the document has loaded. These are an incredibly powerful tool for fetching resources at the appropriate time.

Consider a CSS file that is loaded using an `@import` directive from a another CSS file in a document. Normally, the imported CSS file would only be able to start loading once the CSS file it is linked from has started to load. Using subresources, the request could be made well in advance without affecting how or when the CSS file is actually used.

Similarly, subresources could be used for scripts that use the `defer` and `async` tags, or are scattered between the `<head>` and `<body>`. Scripts referenced as subresources are not run until their corresponding `<script>` tag would otherwise have executed.

Unlike prefetches, subresources should be downloaded immediately with a high priority.

At the time of writing, a bug exists in the Chromium network stack that prevents subresource requests from being used if they haven't completed by the time the asset is used in the document[^chrome_subresources]. Until this issue is resolved, it is not advisable to use subresources. However, they remain a powerful tool and site owners should plan to be able to use them in future versions of Chrome and Opera.

[^chrome_subresources]: See http://crbug.com/312327
