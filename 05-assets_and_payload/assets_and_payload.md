# Assets and Payload

Oftentimes, the slowest part of loading any single page is the time spent downloading the page's content and assets. There are many rules that browsers follow--or in some cases, don't follow--that make it difficult to understand when, how, and why requests are made and what order they're made in. There are also many little-known techniques that can be used to decrease the amount of time it takes to load these assets, whether it be by fetching content in advance, or by providing hints that clients can use to improve performance.

There are also techniques that can be used to minimize the amount of content sent over the wire. The fewer packets that need to be sent, the less time it takes to complete any given request.