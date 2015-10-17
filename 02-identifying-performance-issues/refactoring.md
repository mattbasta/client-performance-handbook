# Refactoring

Refactoring comes in many shapes and sizes. In some cases, it is the means by which tech debt is addressed. In other cases, it involves fixing the formatting of the code.

Refactoring can, however, have both positive and negative performance consequences. On one hand, a refactor could improve the efficiency of a component for common cases. On the other hand, a refactor could also introduce edge cases that perform much worse than the original code.

Before any refactoring work takes place anywhere in a codebase (front-end or back-end), you should ensure that there's an appropriate amount of instrumentation and reporting around those components so that issues can be identified and addressed as they arise.

Regular refactoring--regardless of how minor--is another technique to increase visibility of infrequently-looked-at pieces of code. It's easy to ignore pieces of an application that are known to work, but in doing so, you lose the ability to identify potential performance issues.

In my own work, I try to revisit every file I've worked on every four to six months or so. When I make my rounds, I look for the following:

- Has someone else changed my code? Does the code still work as I expect it to?
- Is the code I've written still effective?
- Has another part of the application changed in a way that would make my code be more effective if written differently?
- Have I learned anything since I last visited this code that could be applied to make it more effective?

