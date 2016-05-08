# The Event Loop

From a page's perspective, all browsers are single-threaded. All JavaScript execution (with few exceptions[^1]), rendering, compositing, and other user-facing operations happen sequentially. The event loop is the operation that makes this possible: it is a single loop that checks if there is any work to be done and executes it if there is.

First and foremost, the event loop handles the execution of all JavaScript. When something in JavaScript needs to take place, the event loop notices the task from a message queue and runs the appropriate code, waiting for it to complete. Since JavaScript is single-threaded, all other code will wait for it to finish, no matter what. Even `setTimeout` commands will wait until executing scripts have completed, regardless of how soon their timer is set for.

```js
// Record the current time
var now = Date.now();
console.log('Starting!');

// Set a timer for one second
setTimeout(function() {
    console.log('one second has passed');
}, 1000);

while (Date.now() - now < 2000) {
    // no-op
}

var delta = Date.now() - now;
console.log('Finished in ' + delta + ' milliseconds');
```

The above code shows this behavior off. The console will always output something that looks like this:

```
Starting!
Finished in 2000 milliseconds
one second has passed
```

Even though a timer was set for one second, it will wait until whatever code was running has fully finished executing.

This is the same reason that all browsers ask if you want to "stop unresponsive scripts" when a piece of code runs for a long time without finishing. Perhaps it is stuck in an infinite loop or performing some sort of CPU-intensive task. Regardless, it is not finishing a "run to completion" and yielding control of the browser back to the event loop.

A "run to completion" is the process of the event loop triggering a piece of JavaScript and waiting for it to fully complete. For example, launching two `setTimeouts` with the same delay will add two tasks in the message queue that will be processed by the event loop after the main script finishes, the second waiting for the first to complete. A run to completion will begin when any of the following things happen:

- A timer fires
- An event is triggered
- An `XMLHttpRequest` changes state
- A worker starts executing
- An `async` script begins to execute
- A promise object resolves or rejects

There are certainly more ways that a run to completion could begin. In any case, understanding how JavaScript runs in the browser is crucial to understanding its performance properties.


[^1]: Notable exceptions include web workers and service workers. There are also unusual edge cases where certain asynchronous operations can happen synchronously, breaking the usual event loop paradigm.
