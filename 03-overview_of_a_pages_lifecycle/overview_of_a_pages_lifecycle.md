# Overview of a Page's Lifecycle

When you load a web page in your browser, there's quite a lot of things that happen. Especially in rich internet applications, browsers are doing an extraordinary amount of work. Some of these operations, such as network IO, can be done concurrently. Other operations, like constructing the DOM or performing certain operations in JavaScript, block the entire page load process.
