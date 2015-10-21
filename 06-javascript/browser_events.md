# Browser Events

An event is an object that represents something that happened in the browser. JavaScript code can set up listeners for these objects using the `addEventListener` method. When an event occurs, the event loop begins executing the JavaScript code for each registered event listener in turn, passing the event object around.

More interestingly to the performance of the application is the purpose of the events that are fired. Of all of the different events, the following are the most critical to understand:

`load`
: The `load` event fires on any element that has just completed loading. It also fires on the `window`. When the window load event has completed loading, the page no longer has any further processing to do related to its lifecycle. At this point, the browser loading indicators will go away.

`DOMContentLoaded`
: This event fires after a page has started loading but before `load` has fired. It signals that the page is at a point where the user can begin interacting with its content. This means that the DOM has finished loading and all scripts have loaded and executed.

`visibilitychange`
: As discussed in Chapter 5, a very powerful technique for improving performance is page prerendering. In order to detect whether the page is being prerendered, you can use the page visibility API, which can be listened for using the `visibilitychange` event in combination with `document.visibilityState`. This is critical if animations or effects need to be shown to the user upon their visit (otherwise they will have started when the page is still being prerendered).

`readystatechange`
: Prior to Internet Explorer 9, there was no support for `DOMContentLoaded`. Instead, `readystatechange` could be used in conjunction with `document.readyState`. Similar results could be achieved on very basic pages, but various browser bugs ultimately made the results behave more like `load`. Unless you require old IE support, it is recommended that you do not use `readystatechange`.

Now you might ask yourself, "which event should my application listen for to start page execution?" That's a great question and unfortunately, the answer is not simple.

- Use `load` if your code relies on everything on the page being loaded, including images and `iframe`s.
- Use `DOMContentLoaded` if your code relies on all other scripts on the page having been executed.
- If your scripts are located at the end of the `<body>` and each script file does not depend on any resource on the page being loaded or any script that comes after it (and the scripts are not given an `async` attribute), you do not need to use *any* event. Simply execute your code immediately.
- If your scripts are marked with `defer` and they do not rely on any script that comes after themselves, you probably do not need to use `DOMContentLoaded`. `defer`'d scripts will always run after the DOM has completed loading. This allows your code to execute sooner.

Some interesting notes:

- jQuery's `ready` event (and closures passed to `$()`) are fired on `DOMContentLoaded` or on `readystatechange` when `document.readyState === 'complete'`.
- In the vast majority of circumstances, code can run on `DOMContentLoaded`. `load` is a poor option because it will wait for images and stylesheets to load, which will cause the page to appear and behave as if it is broken for a short time until absolutely everything has finished loading.
- When using the `readystatechange` event, you must check `document.readyState` to determine whether the page is ready yet. When `readyState` equals `'complete'`, the page is definitely ready. If your code does not do anything strange (particularly around `document.write`), you can check if `readyState` equals `'interactive'`, as well.

