# Head or Body: Where the hell do I put my code?

It has long been a point of contention surrounding where to put script tags. Long ago, the de facto recommendation was to put the tags in the `<head>`. Today, the common recommendation is to put script tags at the end of the `<body>`. With the advent of HTML5 and modern JavaScript patterns, however, the "best" option is more nuanced.

The first thing to consider is how your pages are sent to the browser. If it takes quite a long time for your pages to load, it is imperative that you instruct the browser to start downloading the scripts as early in the page load as possible. If you are performing a header flush to output the `<head>` and start of the `<body>`, you should opt to place your scripts in the `<head>`. This allows the browser to download the files in parallel with the rest of your page. If you have a small amount of markup and cannot flush (e.g., when using Flask without streaming responses), placing the scripts at the end of the body will work just fine.

The next thing to consider is the requirements of your scripts. If your scripts require that the DOM is present and you are unable to use the `defer` attribute, the scripts must be placed at the end of the `<body>`. If you have a single script and the DOM does not need to be present when the script executes, `async` can be used and the script can be placed in `<head>`.

The last thing to consider is the relative ordering of elements in the destination that you choose for your scripts. What you place the scripts before or after can have a significant impact on the performance of the page as a whole.

```html
<html>
  <head>
    <title>Demo</title>
    <script src="external1.js"></script>  <!-- Script 1 -->
    <script src="deferred.js" defer></script>  <!-- Script 2 -->
    <link rel="stylesheet" href="styles.css">
    <script src="external2.js"></script>  <!-- Script 3 -->
  </head>
  <body></body>
</html>
```

Consider the example above. Because Script 1 does not have a `defer` or `async` attribute, it blocks the browser from beginning to process `styles.css`. Script 2, on the other hand, does not block `styles.css` because it is `defer`red. Script 3 is the ideal location for any JavaScript: after all `<style>` and `<link>` tags.

When placing scripts at the end of the `<body>`, it is best to ensure the script tags are the very last elements on the page. No other tags should be placed after them, unless there is a good reason. This helps prevent common coding errors (accessing an element in the DOM that does not exist yet). A script at the end of the `<body>` oftentimes does not benefit from `defer` or `async`, so those attributes are unnecessary.


## To inline or not to inline

A common question is whether scripts should be inline or not. In general, inline scripts should be avoided:

- Inline scripts decrease code quality by decentralizing JavaScript
- They decrease your ability to use `defer` and `async` effectively
- Confusing and troublesome issues related to execution order often appear, especially for deferred code
- Inline scripts must execute before the remainder of the page can be parsed and rendered, which could significantly increase perceived load time

There are a few use cases, however, that benefit from using inline scripts. In all of the below cases, it is important to test the effectiveness of inline scripts using the proper tools.

1. **Embedded pages:** Some pages will sometimes always have visitors that have cold caches. For instance, embeddable widgets or pages which will be `iframe`d on third party websites can have an overwhelming majority of visitors with cold caches. Many of these users will only request the page a single time. In this case, it may be more effective inline the scripts.
2. **Very small JavaScript files:** Some pages only require a very small amount of JavaScript. In this case, the overhead of making the request may be greater than the overhead of transferring extra data as part of the original markup. Be careful that the code being used is not complex, as it will block the remainder of the page from rendering, not to mention decrease overall code quality.
3. **Bootstrapping scripts:** Some JavaScript loaders may require scripts on the page in order to load the remainder of the JavaScript on the site. In this case, inline scripts may be necessary in order to avoid a large performance hit before the application can become even remotely interactive.

Especially when other performance best practices (like HTTP/2) are being used, external scripts are not the bottleneck for performance. Take great care to avoid premature optimization by attempting to inline JavaScript files on the page.
