# Tech Debt

When I talk about tech debt, I am referring to problems with a codebase that decrease its overall quality. These are things like temporary hacks that never got removed, `TODO` comments that are months or even years old, pieces of code that should have been replaced or refactored but still serve some minor purpose.

Being able to identify and track tech debt can help to track performance. In almost all cases, tech debt results in sub-optimal behavior by the application. When you find tech debt, log it. File a ticket in your bug tracking software, write it down in a wiki, mark it with an easily `grep`-able comment, etc.

Over time, as you identify and log tech debt, it will become clear which components in your application need attention. Rather than individual functions and lines, you may find entire files or systems that--while perfectly functional--could be doing a better job (or can be replaced by something far simpler).
