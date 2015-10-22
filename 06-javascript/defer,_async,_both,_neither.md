# Defer, Async, Both, Neither

HTML5 adds two new attributes to the common `<script>` tag which can be used to improve the load times of pages containing one or more JavaScript files. These attributes are `defer` and `async`, and each offers subtle behavior changes that can help you to tune the performance of your page by minimizing the amount of time the browser spends idle waiting for network requests to complete.

The gains that this section purports may be greater or lesser depending on the browser. Modern browsers include what is known as a "preloader." The preloader will "look ahead" at the HTML that has been downloaded while `<script>` tags without `defer` or `async` are running. In doing so, the browser can get a head-start on downloading files that it thinks it will need in order to put the rest of the page together.

This is non-standard behavior, though. The HTML specification officially says that the browser should stop and wait while `<script>` tags are downloading and executing. The performance gains of using the preloader, however, are far too great to ignore: Mozilla reports a nearly 16% improvement[^1] and Google reports a roughly 20% improvement[^2].

[^1]: https://bugzilla.mozilla.org/show_bug.cgi?id=364315#c38
[^2]: https://plus.google.com/+IlyaGrigorik/posts/8AwRUE7wqAE

Even with the preloader in place, you can still provide significant benefit to your site by using `defer` and `async` to hint at how and when the browser should execute your code.


## `defer`

- Inline scripts (i.e., script without a `src=""` attribute) will ignore the `defer` attribute.
- All external `<script>` tags with the `defer` attribute will load in parallel (up to the per-domain connection limit).
- As each deferred script finishes downloading, it will execute.
- A deferred script will not execute until all previous deferred scripts have downloaded and executed.
- Deferred scripts should *never* call `document.write` or any similar functions.
- Deferred scripts will wait to execute until the full page has been downloaded and parsed. i.e., they will wait for the DOM to be fully loaded.
- Deferred scripts will block `DOMContentLoaded` from firing.

From a high level, a script with the `defer` attribute behaves very similarly to the default behavior of external script tags. Here are two scenarios:

```html
<script src="js/en-US.js"></script>
<script src="js/include.js"></script>
```

In the above example, the browser will download and execute the first script, then download and execute the second script.

```html
<script src="js/en-US.js" defer></script>
<script src="js/include.js" defer></script>
```

In the second example, the browser will begin downloading both files simultaneously. If `en-US.js` loads first, it will execute as soon as it can. If `include.js` loads first, it will wait for `en-US.js` to finish downloading and executing before it executes itself.

Most sites that do not have any inline scripts can safely add `defer` to all of their script tags without any other changes to their code. For browsers that support `defer` but do not have a preloader, this can provide immediate and significant performance improvements.

Because deferred scripts will wait until the DOM has completed before executing, it does not matter where the scripts are located on the page. This means that scripts can be placed earlier in the document (e.g., the `<head>`) in order to start downloading sooner without needing to worry about whether problems will arise because the DOM is not available. For example, consider the following code:

```js
alert(document.body.innerHTML);
```

If we save the above JS into `test.js` and run the following code, some potentially unexpected behavior will occur:

```html
<html>
  <head>
    <title>Demo</title>
    <script src="test.js"></script>
  </head>
  <body>hello</body>
</html>
```

If the above snippet is run, a JavaScript error will occur! `document.body` is undefined because when the script is running, the `<body>` hasn't been encountered yet and thus `document.body` hasn't been set. This problem can be completely resolved by putting the `defer` attribute on the `<script>` tag:

```html
<html>
  <head>
    <title>Demo</title>
    <script src="test.js" defer></script>
  </head>
  <body>hello</body>
</html>
```

With the `defer` attribute, you will receive an alert containing "hello", as you would expect.

You should use `defer` under the following circumstances:

- There are no inline scripts on your page, or none of the inline scripts depend on external scripts having been loaded.
- Your scripts are located at the end of the `<body>` to prevent issues related to the DOM being absent.
- Your scripts do not call `document.write` or `document.writeln`.
- Your code does not need to support Internet Explorer 9 or below. A bug in the way old IE supports deferred scripts will cause the scripts to occasionally execute out-of-order.


## `async`

The `async` attribute is similar to the `defer` attribute, but the scripts may be executed at wildly different times. Unlike `defer`, scripts loaded with `async` are not executed in the order that they appear on the page. Instead, they are executed immediately after they are downloaded.

- Scripts with `async` will not wait until the DOM has finished parsing. Consequently, these scripts 
- Scripts with `async` will run out-of-order.
- `DOMContentLoaded` will not wait for `async` scripts to download or execute, though the `load` event will.

For example, consider the following script, which we'll call `whoami.js`:

```js
alert(document.currentScript.src);
```

The above JavaScript will alert the URL of the currently executing script file.

```html
<html>
  <head>
    <title>Demo</title>
    <script src="whoami.js?first" async></script>
    <script src="whoami.js?second" async></script>
    <script src="whoami.js?third" async></script>
  </head>
  <body></body>
</html>
```

In the above HTML, you might see any of the following alerts:

- "whoami.js?first", "whoami.js?third", then "whoami.js?second"
- "whoami.js?first", "whoami.js?second", then "whoami.js?third"
- "whoami.js?second", "whoami.js?first", then "whoami.js?third"
- "whoami.js?second", "whoami.js?third", then "whoami.js?first"
- "whoami.js?third", "whoami.js?first", then "whoami.js?second"
- "whoami.js?third", "whoami.js?second", then "whoami.js?first"

For most applications, simply adding `async` is not a viable option. `async` is better suited for new projects that can be built to support the limitations that adding the attribute imposes.

The following circumstances are cases where `async` can be very useful:

- Inline scripts have no dependencies on external scripts.
- Each external script has no dependencies on other external scripts.
- The order in which the scripts run is not important.
- The scripts do not rely on the DOM being fully loaded (the same as a script that is safe to include in the `<head>` without `defer` or `async`).
- Because `async` scripts do not block `DOMContentLoaded`, the code should not rely on the event firing.


## `defer` and `async` together

During the time where `defer` and `async` had not been widely adopted or standardized across browsers, it was recommended to specify both attributes for pages that contained only a single external script. The HTML5 specification has formally defined the behavior for this case, however: when `defer` and `async` are both specified, `async` overrides `defer`. If `async` is not supported by the browser, the browser can then fall back on `defer`.

Today, all modern browsers support both `defer` and `async`. There is no need to specify both.


## Inline scripts

Because inline scripts cannot be deferred, it is impossible to have an inline script execute after a deferred script. For instance:

```js
// external.js
console.log('external');
```

```html
<html>
  <head>
    <title>Demo</title>
    <script src="external.js" defer></script>
    <script>
    console.log('inline');
    </script>
  </head>
  <body></body>
</html>
```

The above code will always output the following:

```text
inline
external
```

How, then, do you execute an inline script after an deferred external script? The answer is to use an external script with the data inline. This can be achieved using a data URI:

```html
<html>
  <head>
    <title>Demo</title>
    <script src="external.js" defer></script>
    <script src="data:application/javascript;console.log('inline');"></script>
  </head>
  <body></body>
</html>
```

The above will always output the following:

```text
external
inline
```

This something of a dirty hack, but it can be exceptionally useful in some circumstances. Always make sure to consider all possible alternatives before choosing this approach, and make sure to thoroughly document what this code is and does so others are not confused by it.


## When to use neither `async` nor `defer`

There are few cases where neither of the attributes are appropriate for a piece of code. The most common use case is when inline scripts are needed. In this case, it may be more appropriate to lean more heavily on the browser's built-in preloader than it is to add hacks to support the inline scripts.

Additionally, `defer` should NOT be used when versions of Internet Explorer prior to IE10 must be supported. As mentioned previously, there is a bug that can cause old IE to execute deferred scripts in the wrong order.

Note that scripts without `defer` and `async` block the browser from parsing and rendering the remainder of the page. Even if the browser is able to begin to download the CSS referenced in a `<link>` tag, the "critical path" is blocked until the script has downloaded and executed. It is a bad practice to place any sort of non-deferred script tag before a `<link>` or `<style>` tag on a page. Also beware older browsers that do not respect `async` or `defer`, which can also cause them to block.
