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


### Preloading

The W3C has been working to formalize a specification for all of this. It is known as "preloading" rather than "prefetching". Ultimately, preloading works in a similar way to prefetching works. A `<link>` tag can be used, though it accepts an extra attribute: `as`.

```html
<link rel="preload" href="/my/image.png" as="image">
```

When the browser sees the `<link>` tag, it will begin downloading the image with the priority that an image would be given had an `<img>` tag existed. Using `as="font"` would create a network request with the priority of a web font.

The values that you can use for the `as=""` attribute are the same values used by the `fetch()` API. Common values include `font`, `script`, `style`, `audio`, and `video`.

A similar HTTP header can also be used:

```
Link: </my/image.png>; rel=preload; as=image
```
