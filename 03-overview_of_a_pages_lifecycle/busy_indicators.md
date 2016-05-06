# Busy Indicators

When making a web application, it is important to understand how users experience the loading process of the page. Different browsers show different types of busy indicators, and these indicators may appear or disappear at different phases of page load. Knowing how these affect your app's UX can help guide your design decisions.

## What is a busy indicator?

A busy indicator (or a "loading indicator") is a UI element in the browser that tells the user that something is loading. In some browsers, this may be a spinning icon on a browser tab. In other browsers, this may be a message in the status bar. Other browsers will show a progress bar along the top of the page or in the address bar. On desktops, the user's cursor may change to show that the page is loading.

Subtler indicators also exist. The refresh or reload button in the user's browser may turn into a "stop" or "X" button.

## What triggers a busy indicator?

First there is of course the basic loading of a website, either by clicking a link or by entering its url manually. But there is also a few others causes that can trigger them, depending on your browser and version:

- the user is redirected to another page
- an async script is being loaded
- the page creates a `<script>`, `<iframe>`, `<img>`, stylesheet or CSS background dynamically
