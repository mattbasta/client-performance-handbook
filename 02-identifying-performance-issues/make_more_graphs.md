# Make More Graphs

One of the most key aspects of identifying problems is being able to passively identify performance problems. Nobody has time to read through dozens--if not hundreds--of tables of numbers every day. Building software that can sniff out problems is equally hard. Instead, technology can be harnessed to help facilitate your understanding of your site's performance.

My personal recommendation is to set up monitors where you work that cycle through a series of charts. Graph everything: load times, ready times, times required to execute JavaScript...all of it. You don't need to be watching it constantly, but passively observing it and noting changes helps to bring awareness. If something looks interesting, it's worthwhile to run some queries and investigate.

Having charts and graphs of performance information also helps to outsource the work of identifying problems. If you work in an office with other engineers, it's not unlikely that you'll get lots of questions about what the graphs represent, or questions about why they changed in ways that they did. With more sets of eyes, it's easier to identify regressions. Not to mention, it helps draw attention to performance wins.

As always, thorough monitoring is always important for critical systems. All systems that require high uptime should already be monitoring performance anyway; client performance is only a single facet of that, but is often neglected. Piping all of your collected metrics into the same system that monitors back-end performance is a great idea, and often gets you charts and dashboards for free. Software like Nagios can be used to send you alerts when there are serious performance regressions.
