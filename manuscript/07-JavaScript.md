# JavaScript

## The Event Loop

All browsers run on what is known as an event loop. The event loop is just that: a single loop that checks if there is any work to be done and if there is, it does it. Granted, there are parallel operations that take place (certain compositing operations and web workers, for instance) but in general, a surprisingly small amount of parallelism exists in your browser[^servo_parallel].

[^servo_parallel]: Servo, a project from Mozilla, seeks to build a browser that takes advantage of parallelism in all aspects of the page lifecycle. https://github.com/mozilla/servo

The event loop handles the execution of all JavaScript. When something in JavaScript needs to be done, the event loop notices the task and fires the appropriate code, waiting for it to complete. Since JavaScript is single-threaded, all other code will wait for it to finish, no matter what. Even `setTimeout` commands will wait until executing scripts have completed, regardless of how soon their timer is set for. This is the same reason that all browsers ask if you want to "stop unresponsive scripts" when a piece of code runs for a very long time without ever finishing. Perhaps it is stuck in an infinite loop or performing some sort of extremely intense processing; regardless, it is not finishing a run to completion and yielding control of the browser back to the event loop.

The process of the event loop triggering a JavaScript function and waiting for all subsequent processing within the JavaScript engine to complete is known as a "run to completion." Understanding how this works is very important to being able to understand the performance characteristics of an application.


## Browser Events

An event is simply an object that represents something that happened in the browser. JavaScript code can set up listeners for these objects using the `addEventListener` method. When an event occurs, the event loop begins executing the JavaScript code for each registered event listener in turn, passing the event object around.

More interestingly to the performance of the application is the purpose of the events that are fired. Of all of the different events, the following are the most critical to understand:

`load`
: The `load` event fires on any element that has just completed loading. It also fires on the `window`. When the window load event has completed loading, the page no longer has any further processing to do related to its lifecycle. At this point, the browser loading indicators will go away.

`DOMContentLoaded`
: This event fires after a page has started loading but before `load` has fired. It signals that the page is at a point where the user can begin interacting with its content. This means that the DOM has finished loading and all scripts have loaded and executed[^firefox_dcl_bug].

`visibilitychange`
: As discussed in Chapter 5, a very powerful technique for improving performance is page prerendering. In order to detect whether the page is being prerendered, you can use the page visibility API, which can be listened for using the `visibilitychange` event in combination with `document.visibilityState`. This is critical if animations or effects need to be shown to the user upon their visit (otherwise they will have started when the page is still being prerendered).

`readystatechange`
: Prior to Internet Explorer 9, there was no support for `DOMContentLoaded`. Instead, `readystatechange` could be used in conjunction with `document.readyState`. Similar results could be achieved on very basic pages, but various browser bugs ultimately made the results behave more like `load`. Unless you require old IE support, it is recommended that you do not use `readystatechange`.

[^firefox_dcl_bug]: Firefox has a very nasty bug which causes `DOMContentLoaded` to fire before certain scripts have finished executing. It has been fixed, but it will not become generally available until Firefox 31. A workaround does exist, which will be presented in upcoming sections. http://bugzil.la/688580

Now you might ask yourself, "which event should my application listen for to start page execution?" That's a great question and unfortunately, the answer is not simple.

- Use `load` if your code relies on everything on the page being loaded, including images and `iframe`s.
- Use `DOMContentLoaded` if your code relies on all other scripts on the page having been executed.
- If your scripts are located at the end of the `<body>` and each script file does not depend on any resource on the page being loaded or any script that comes after it (and the scripts are not given an `async` attribute), you do not need to use *any* event. Simply execute your code immediately.
- If your scripts are marked with `defer` and they do not rely on any script that comes after themselves, you probably do not need to use `DOMContentLoaded`. `defer`'d scripts will always run after the DOM has completed loading. This allows your code to execute sooner.

Some interesting notes:

- jQuery's `ready` event (and closures passed to `$()`) are fired on `DOMContentLoaded` or on `readystatechange` when `document.readyState === 'complete'`.
- In the vast majority of circumstances, code can run on `DOMContentLoaded`. `load` is a poor option because it will wait for images and stylesheets to load, which will cause the page to appear and behave as if it is broken for a short time until absolutely everything has finished loading.
- When using the `readystatechange` event, you must check `document.readyState` to determine whether the page is ready yet. When `readyState` equals `'complete'`, the page is definitely ready. If your code does not do anything strange (particularly around `document.write`), you can check if `readyState` equals `'interactive'`, as well.


## Defer, Async, Both, Neither

HTML5 adds two new attributes to the common `<script>` tag which can be used to improve the load times of pages containing one or more JavaScript files. These attributes are `defer` and `async`, and each offers subtle behavior changes that can help you to tune the performance of your page by minimizing the amount of time the browser spends idle waiting for network requests to complete.

The gains that this section purports may be greater or lesser depending on the browser. Modern browsers include what is known as a "preloader." The preloader will "look ahead" at the HTML that has been downloaded while `<script>` tags without `defer` or `async` are running. In doing so, the browser can get a head-start on downloading files that it thinks it will need in order to put the rest of the page together.

This is non-standard behavior, though. The HTML specification officially says that the browser should stop and wait while `<script>` tags are downloading and executing. The performance gains of using the preloader, however, are far too great to ignore: Mozilla reports a nearly 16% improvement[^firefox_preloader_bench] and Google reports a roughly 20% improvement[^chrome_perloader_bench].

[^firefox_preloader_bench]: https://bugzilla.mozilla.org/show_bug.cgi?id=364315#c38
[^chrome_perloader_bench]: https://plus.google.com/+IlyaGrigorik/posts/8AwRUE7wqAE

Even with the preloader in place, you can still provide significant benefit to your site by using `defer` and `async` to hint at how and when the browser should execute your code.


### `defer`

- Inline scripts (i.e.: script without a `src=""` attribute) will ignore the `defer` attribute.
- All external `<script>` tags with the `defer` attribute will load in parallel (up to the per-domain connection limit).
- As each deferred script finishes downloading, it will execute.
- A deferred script will not execute until all previous deferred scripts have downloaded and executed.
- Deferred scripts should *never* call `document.write` or any similar functions.
- Deferred scripts will wait to execute until the full page has been downloaded and parsed. I.e.: they will wait for the DOM to be fully loaded.
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

Because deferred scripts will wait until the DOM has completed before executing, it does not matter where the scripts are located on the page. This means that scripts can be placed earlier in the document (e.g.: the `<head>`) in order to start downloading sooner without needing to worry about whether problems will arise because the DOM is not available. For example, consider the following code:

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


### `async`

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


### `defer` and `async` together

During the time where `defer` and `async` had not been widely adopted or standardized across browsers, it was recommended to specify both attributes for pages that contained only a single external script. The HTML5 specification has formally defined the behavior for this case, however: when `defer` and `async` are both specified, `async` overrides `defer`. If `async` is not supported by the browser, the browser can then fall back on `defer`.

Today, all modern browsers support both `defer` and `async`. There is no need to specify both.


### Inline scripts

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


### When to use neither `async` nor `defer`

There are few cases where neither of the attributes are appropriate for a piece of code. The most common use case is when inline scripts are needed. In this case, it may be more appropriate to lean more heavily on the browser's built-in preloader than it is to add hacks to support the inline scripts.

Additionally, `defer` should NOT be used when versions of Internet Explorer prior to IE10 must be supported. As mentioned previously, there is a bug that can cause old IE to execute deferred scripts in the wrong order.

Note that scripts without `defer` and `async` block the browser from parsing and rendering the remainder of the page. Even if the browser's preloader is able to begin to download download the page's CSS as referenced in a `<link>` tag, the "critical path" is blocked until the script has downloaded *and* executed. Consequently, it is a bad practice to place any sort of non-deferred script tag before a `<link>` or `<style>` tag on a page.


## Head or Body: Where the hell do I put my code?

It has long been a point of contention surrounding where to put script tags. Long ago, the de facto recommendation was to put the tags in the `<head>`. Today, the common recommendation is to put script tags at the end of the `<body>`. With the advent of HTML5 and modern JavaScript patterns, however, the "best" option is more complex.

The first thing to consider is how your pages are being sent to the browser. If it takes quite a long time for your pages to load, it is imperative that you instruct the browser to start downloading the scripts as early in the page load process as possible. If you are performing a header flush to output the `<head>` and start of the `<body>`, you should opt to place your scripts in the head so that the browser can download the files in parallel with the rest of your page. If you have a relatively small amount of HTML and cannot flush (e.g.: when using Flask without streaming responses), the benefit of having the scripts in the head is minimal and placing the scripts at the end of the body may simply be less work.

The next thing to consider is the requirements of your scripts. If your scripts require that the DOM is present and you cannot use the `defer` attribute, the scripts simply must be placed at the end of the `<body>`. If you only have a single script and do not rely on the DOM being ready, the script can be marked with `async` and placed in the `<head>`. If you have multiple scripts and can use the `defer` attribute, it is generally safe to place them in the `<head>`[^beware_firefox].

[^beware_firefox]: Beware the aforementioned Firefox bug that causes `DOMContentLoaded` to fire at an inappropriate time when multiple deferred scripts are used on a page. A workaround is discussed in the previous section.

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

Consider the example above. Because Script 1 does not have a `defer` or `async` attribute, it blocks the browser from beginning to process `styles.css`. Script 2, on the other hand, does not block `styles.css` because it is deferred. Script 3 is the ideal location for any JavaScript: after all `<style>` and `<link>` tags.

When placing scripts at the end of the `<body>`, it is best to ensure the script tags are the very last elements on the page. No other tags should be placed after them, unless there is a very specific purpose.


### To inline or not to inline

A common question is whether scripts should be placed inline or not. In general, inline scripts should be avoided

- Inline scripts decrease code quality by decentralizing JavaScript
- They decrease your ability to use `defer` and `async` effectively
- Confusing and troublesome issues related to execution order often appear, especially for deferred code
- Inline scripts must execute before the remainder of the page can be parsed and rendered, which could significantly increase perceived load time

There are a few use cases, however, that benefit significantly from using inline scripts. In all of the below cases, it is very important to test the effectiveness of inline scripts using the tools discussed earlier in the book to analyze whether they have a positive or negative performance impact.

1. **Embedded pages:** Some pages will sometimes always have visitors that have cold caches. For instance, embeddable widgets or pages which will be iframed on third party websites will usually have an overwhelming majoity of visitors with cold caches. Additionally, many of these users will only request the page a single time. In this circumstance, it may be more effective inline the scripts. The caching benefit of external scripts is eliminated, and minimizing the number of connections that the user makes to your servers is often beneficial.
2. **Very small JavaScript files:** Some pages only require a very small amount of JavaScript. In this case, the overhead of making the request may be greater than the overhead of transferring extra data as part of the original markup. Be careful that the code being used is not very complex, as it will block the remainder of the page from rendering and being displayed.
3. **Bootstrapping scripts:** Some JavaScript loaders may require scripts on the page in order to load the remainder of the JavaScript on the site. In this case, inline scripts may be necessary in order to avoid a very large performance hit before the application can become even remotely interactive.

In general--especially when other performance best practices (like SPDY) are being used--external scripts are not the bottleneck for page load performance. In fact, the ability to load multiple JavaScript files in parallel oftentimes significantly increases the page load performance that users with poor connection speeds will experience.


## Memory

With a built-in garbage collector, simple object literal syntax, and automatic "passing-by-reference"[^pass_by_reference] for objects, it's easy to forget the impact of memory allocation on application performance. For every object created, space must be allocated on the heap. When the object is dereferenced, it must be cleaned up to make room for other objects. Depending on the browser, this may have varying performance impacts.

[^pass_by_reference]: JavaScript isn't truly pass-by-reference. Rather, it's what you might call "pass-by-copy-of-a-reference." Unlike C++, you cannot point a reference to an entirely new object by reassigning the value of a passed argument.


### Garbage Collection

Garbage collection is the process of cleaning up objects on the heap that are no longer being used. If there are no variables, objects, or closures that contain references to an object, it becomes eligible for cleanup by the garbage collector.

At the time of writing, some browsers have better garbage collection algorithms than others. There are a few basic types of garbage collectors in use in JavaScript today:

Reference Counting GC
: A reference counting garbage collector keeps track of the number of references that exist to each object. When the reference count for an object drops to zero, it can safely be garbage collected. One of the biggest downsides to this approach is that cycles (an object referencing itself, or an object referencing another object which eventually references itself) can cause objects to improperly avoid being collected: the reference count will never reach zero. Techniques can be used to avoid cycles, though these can introduce a signficant amount of overhead.

Traditional Mark-and-Sweep GC
: A mark-and-sweep garbage collector stops all JavaScript execution when a set of conditions are met. During this "GC pause," the garbage collector inspects each object on the heap to see whether it has a reference. If an object has a reference, it is marked as "live." When all objects with references have been marked, the garbage collector revisits every allocated object. If an object is not marked as "live," it is freed. This approach has the downside of pausing JavaScript execution temporarily. In performance-critical applications, these pauses can cause frustrating delays. To help prevent issues around these pauses, some browsers perform this process incrementally: pauses are divided up and spread out over the course of an application's execution to make them less noticeable and take better advantage of browser idle time.

Generational Garbage Collection
: A generational garbage collection algorithm is a type of mark-and-sweep collection that keeps objects which live for only a short time separate from objects which live for quite a long time. If objects that become unreferenced quickly are kept separate, there is a smaller segment of memory that needs to be considered for garbage collection. Generational GC is the most efficient garbage collection algorithm used in browsers today.


Currently, the usage of generational garbage collection is limited. See the following table for information on each browser:



## Improving CPU-Heavy Code

## API Performance

## Asm.js

## Frameworks and Performance

## Client-Side Templating
