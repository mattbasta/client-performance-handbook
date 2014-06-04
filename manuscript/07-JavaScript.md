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


## Head or Body: Where to put your code

## Defer, Async, Both, Neither


## Memory

## Improving CPU-Heavy Code

## API Performance

## Asm.js

## Frameworks and Performance

## Client-Side Templating
