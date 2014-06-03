# JavaScript

## The Event Loop

All browsers run on what is known as an event loop. The event loop is just that: a single loop that checks if there is any work to be done and if there is, it does it. Granted, there are parallel operations that take place (certain compositing operations and web workers, for instance) but in general, a surprisingly small amount of parallelism exists in your browser[^servo_parallel].

[^servo_parallel]: Servo, a project from Mozilla, seeks to build a browser that takes advantage of parallelism in all aspects of the page lifecycle. https://github.com/mozilla/servo

The event loop handles the execution of all JavaScript. When something in JavaScript needs to be done, the event loop notices the task and fires the appropriate code, waiting for it to complete. Since JavaScript is single-threaded, all other code will wait for it to finish, no matter what. Even `setTimeout` commands will wait until executing scripts have completed, regardless of how soon their timer is set for.

The process of the event loop triggering a JavaScript function and waiting for all subsequent processing within the JavaScript engine to complete is known as a "run to completion." Understanding how this works is very important to being able to understand the performance characteristics of an application.


## Browser Events




## Memory

## Improving CPU-Heavy Code

## API Performance

## Head or Body: Where to put your code

## Defer, Async, Both, Neither

## Asm.js

## Frameworks and Performance

## Client-Side Templating
